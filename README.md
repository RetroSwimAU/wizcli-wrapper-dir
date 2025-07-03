# Wizcli Wrapper Mod

This is a repository for the **Wizcli Wrapper - directory scan mod** GitHub action. 

You can use this action for the following:
- scanning Infrastructure as Code (IaC) files in your repository for vulnerabilities and compliance issues:
  - Terraform	-> HCL (.tf) files
  - Terraform	-> JSON output of Terraform plan. Generate this file in your plan stage using this command: `terraform plan -out plan.tfplan && terraform show -json plan.tfplan > plan.tfplanjson`
  - AWS CloudFormation	-> JSON or YAML files
  - Azure Resource Manager	-> JSON files
  - Kubernetes	-> YAML manifest files
  - Helm -> YAML chart files
  - Docker -> Files named Dockerfile or with a .dockerfile extension
- scanning local Docker images for vulnerabilities
  - The wiz-cli scans local Docker images, analyzing binaries and packages in the container image, performing full vulnerability and secrets scans, and then outputting the results to the terminal
- scanning for secrets in your repository and Docker images
  - the wizcli will search for secrets during the IaC and the Docker scan by default
- scan code in directories ***(new)***
  - wizcli scans code in directories for vulnerable dependencies, probably build your project first?

Not supported:

- scanning of VM Images
- scanning of Virtual Machines

# Prerequisites

- Wiz [service account](https://docs.wiz.io/wiz-docs/docs/set-up-wiz-cli#generate-a-wiz-service-account-key)
- The following secrets in GitHub repo or organization. (`WP only`) We already completed this step for your Github EMU Organization and added the secrets
   - `WIZ_CLIENT_ID`
   - `WIZ_CLIENT_SECRET`
- Linux runner (Ubuntu, Debian, CentOS, etc.)

# Usage

```yaml
  uses: RetroSwimAU/wizcli-wrapper-mod@v0.2-alpha
  with: 
    wiz_client_id: ${{ secrets.WIZ_CLIENT_ID }}
    wiz_client_secret: ${{ secrets.WIZ_CLIENT_SECRET }}

```

# Table of Contents

- [Wizcli Wrapper Mod](#wizcli-wrapper-mod)
- [Prerequisites](#prerequisites)
- [Usage](#usage)
- [Table of Contents](#table-of-contents)
  - [All default values expanded](#all-default-values-expanded)
- [Scenarios](#scenarios)
  - [Scan only IaC with custom policy](#scan-only-iac-with-custom-policy)
  - [Scan only Docker images with custom policy and relative Dockerfile path](#scan-only-docker-images-with-custom-policy-and-relative-dockerfile-path)
  - [Scan code in specified path](#scan-code-in-specified-path)
  - [Scanning for secrets and breaking the build](#scanning-for-secrets-and-breaking-the-build)
- [Development](#development)
  - [Contributing](#contributing)
  - [Release instructions](#release-instructions)
- [License](#license)

## All default values expanded

This section contains all inputs of the action and their default values. You can use this section as a template for your workflow file to modify some of the values. You can find detailed descriptions of the inputs in the [action.yml](./action.yml) file.

```yaml
name: Wiz Full Scan
on:
  pull_request:
    paths:
      - "**.tf**"
      - "**.tfvars"
      - "**.tfplanjson"
      - "**.json"
      - "**.yaml"
      - "**.yml"
      - "**Dockerfile"
      - "**.dockerfile"
  push:
    branches:
      - main

jobs:
  wiz-full-scan:
    name: wiz-full-scan
    runs-on: [REPLACE WITH YOUR RUNNER] # e.g. [ubuntu-latest] or [self-hosted,sg-kubernetes,ap-northeast-1]
    steps:
      
      # Checkout the repository to the GitHub Actions runner
      - name: Check out repository
        uses: actions/checkout@v3

      # Run both wiz iac and docker scans, and upload the results to Wiz
      - name: Wiz Full Scan With Default Values
        uses: RetroSwimAU/wizcli-wrapper-mod@v0.2-alpha
        with:

          # Directory vulnerability scan defaults
          enable_dir_scan: "false"
          dir_scan_path: "."
          wiz_dir_policy: "Default vulnerabilities policy"
          wiz_dir_report_name: "${{ github.repository }}-directory-${{ github.run_number }}"
          wiz_dir_tags: "repo=${{ github.repository }},commit_sha=${{ github.sha }},pr_title=${{ github.event.pull_request.title }},pr_number=${{ github.event.number}},event_name=${{ github.event_name }},github_workflow=${{ github.workflow }}"

          # Docker images vulnerability scan defaults
          enable_docker_scan: "false"
          docker_scan_path: "."
          docker_image_tag: "${{ github.repository }}-${{ github.run_number }}"
          wiz_docker_policy: "Default vulnerabilities policy"
          wiz_docker_tags: "repo=${{ github.repository }},commit_sha=${{ github.sha }},pr_title=${{ github.event.pull_request.title }},pr_number=${{ github.event.number}},event_name=${{ github.event_name }},github_workflow=${{ github.workflow }}"

          # IaC scan defaults
          enable_iac_scan: "false"
          iac_scan_path: "."
          wiz_iac_policy: "Default IaC policy"
          wiz_iac_report_name: "${{ github.repository }}-${{ github.run_number }}"
          wiz_iac_tags: "repo=${{ github.repository }},commit_sha=${{ github.sha }},pr_title=${{ github.event.pull_request.title }},pr_number=${{ github.event.number}},event_name=${{ github.event_name }},github_workflow=${{ github.workflow }}"
          

          # Docker images vulnerability scan defaults
          enable_docker_scan: "false"
          docker_image_tag: "${{ github.repository }}-${{ github.run_number }}"
          docker_scan_path: "."
          wiz_docker_vulnerabilities_policy: "Default vulnerabilities policy"
          wiz_docker_report_name: "${{ github.repository }}-${{ github.run_number }}"
          wiz_docker_tags: "repo=${{ github.repository }},commit_sha=${{ github.sha }},pr_title=${{ github.event.pull_request.title }},pr_number=${{ github.event.number}},event_name=${{ github.event_name }},github_workflow=${{ github.workflow }}"

          # Common inputs
          wiz_client_id: ${{ secrets.WIZ_CLIENT_ID }}
          wiz_client_secret: ${{ secrets.WIZ_CLIENT_SECRET }}
```

# Scenarios
## Scan only IaC with custom policy

```yaml
- name: Wiz IaC Scan
  uses: RetroSwimAU/wizcli-wrapper-mod@v0.2-alpha
  with: 
    enable_iac_scan: "true"
    wiz_iac_policy: "YOUR_CUSTOM_IAC_POLICY"
    wiz_client_id: ${{ secrets.WIZ_CLIENT_ID }}
    wiz_client_secret: ${{ secrets.WIZ_CLIENT_SECRET }}
```

## Scan only Docker images with custom policy and relative Dockerfile path

```yaml
- name: Wiz Docker Image Vulnerability Scan
  uses: RetroSwimAU/wizcli-wrapper-mod@v0.2-alpha
  with: 
    enable_docker_scan: "true"
    docker_scan_path: "./YOUR_RELATIVE_PATH_TO_DOCKERFILE_FOLDER"
    wiz_docker_policy: "YOUR_CUSTOM_VULN_POLICY"
    wiz_client_id: ${{ secrets.WIZ_CLIENT_ID }}
    wiz_client_secret: ${{ secrets.WIZ_CLIENT_SECRET }}
```

## Scan code in specified path

```yaml
- name: Wiz Directory Vulnerability Scan
  uses: RetroSwimAU/wizcli-wrapper-mod@v0.2-alpha
  with: 
    enable_dir_scan: "true"
    dir_scan_path: "./YOUR_RELATIVE_PATH_TO_CODE_TOP_LEVEL"
    wiz_dir_policy: "YOUR_CUSTOM_VULN_POLICY"
    wiz_client_id: ${{ secrets.WIZ_CLIENT_ID }}
    wiz_client_secret: ${{ secrets.WIZ_CLIENT_SECRET }}
```


## Scanning for secrets and breaking the build

```yaml
- name: Wiz Full Scan With Secrets Check
  uses: RetroSwimAU/wizcli-wrapper-mod@v0.2-alpha
  with: 
    enable_dir_scan: "true"
    enable_iac_scan: "true"
    enable_docker_scan: "true"
    wiz_iac_policy: "Default IaC policy,YOUR_CUSTOM_SECRETS_POLICY"
    wiz_docker_vulnerabilities_policy: "Default vulnerabilities policy,YOUR_CUSTOM_SECRETS_POLICY"
    wiz_client_id: ${{ secrets.WIZ_CLIENT_ID }}
    wiz_client_secret: ${{ secrets.WIZ_CLIENT_SECRET }}
```

# Development
## Contributing

1. Fork the repository 
2. Create a feature branch `feature/your-feature-name`
3. Test locally and create a pull request to the `main` branch
## Release instructions

1. Locate the semantic version of the [upcoming release][release-list] (a draft is maintained by the [`draft-release` workflow][draft-release])
2. Publish the draft release from the `main` branch with semantic version as the tag name, _without_ the checkbox to publish to the GitHub Marketplace checked
3. After publishing the release, the [`release` workflow][release] will automatically run to create/update the corresponding the major version tag such as `v0`


# License

The scripts and documentation in this project are released under the [MIT License](./LICENSE)

<!-- references -->
[release-list]: /releases
[draft-release]: .github/workflows/draft-release.yml
[release]: .github/workflows/release.yml
