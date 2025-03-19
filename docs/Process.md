# Shared CI/CD Step Library for Terraform Module Repositories

This guide outlines how to create, share, and integrate reusable continuous integration (CI) steps for Terraform module repositories. The objective is to provide a common, modular framework that different teams can adopt with minimal effort while meeting the best practices of CI/CD.

## Overview

Our setup leverages:
- **Terraform** for Infrastructure as Code (IaC)
- Two repositories:
  - A **Terraform Module Repo** (contains reusable modules)
  - A **Terraform Environment Repo** (manages environment-specific configurations)
- **GitHub** as our version control system (VCS)
- **GitHub Actions** for CI steps in module repos
- **CircleCI** for additional environment testing and orchestration

The goal is to enable teams to easily adopt new CI steps developed centrally while ensuring all coding and design principles are met.

---

## Step 1: Define and Develop Modular CI Steps

### 1.1. Identify Common CI Tasks
- **Examples:**  
  - Code formatting & linting (e.g., using `terraform fmt` and `terraform validate`)
  - Terraform initialization and plan
  - Automated testing of module changes (e.g., dry-run or plan output validation)
  - Artifact generation and version tagging
  - Deployment approvals (for environments like pre-dev, dev, uat, prod)

### 1.2. Develop CI Steps as Self-contained Modules
- **For GitHub Actions:**  
  Create reusable workflows that encapsulate each CI step.  
  _Example: A reusable workflow for Terraform validation (see below)._

- **For CircleCI:**  
  Define reusable commands or executor configurations in the `.circleci/config.yml` file that teams can reference.

### 1.3. Parameterize Each CI Step
- Make each module configurable via input parameters (e.g., environment name, branch, Terraform variable files).
- Use environment variables or YAML parameters so that the same step can run with different configurations without code changes.

---

## Step 2: Document & Version Your CI Step Library

### 2.1. Documentation
- **For Each Module:**  
  - Write clear usage instructions, including input parameters, expected outputs, and any dependencies.
  - Maintain a README with examples and guidelines on how to integrate the CI steps.
  
### 2.2. Version Control
- Publish the shared CI step library in a dedicated repository.
- Use semantic versioning for the library so teams can safely upgrade (e.g., v1.0.0, v1.1.0).

---

## Step 3: Create a Repository for the Shared CI Steps

### 3.1. Repository Structure Example
```
shared-ci-steps/
├── .github/
│   └── workflows/
│       ├── terraform-validate.yml     # Reusable workflow for validation
│       ├── terraform-plan.yml         # Reusable workflow for planning changes
│       └── terraform-deploy.yml       # Reusable workflow for deployment steps
├── circleci/
│   └── config.yml                     # Contains reusable commands/executors
├── docs/
│   ├── README.md                      # Overview and integration guidelines
│   └── examples.md                    # Example configurations
└── VERSION                          # File storing current version
```

---

## Step 4: Example – A Reusable GitHub Actions Workflow for Terraform Validation

Create a file `.github/workflows/terraform-validate.yml` in the shared CI steps repo:

```yaml
name: Reusable Terraform Validate

on:
  workflow_call:
    inputs:
      working_directory:
        required: true
        type: string
      extra_arguments:
        required: false
        type: string

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2

      - name: Terraform Init & Validate
        run: |
          cd ${{ inputs.working_directory }}
          terraform init ${{ inputs.extra_arguments }}
          terraform validate
```

*Usage:* Any team can call this reusable workflow from their module repo by referencing it in their own workflow file.

---

## Step 5: Integrate Shared CI Steps into a Team's Repository

### 5.1. GitHub Actions Integration

In a team’s Terraform module repository, create a workflow (e.g., `.github/workflows/ci.yml`) that calls the shared workflow:

```yaml
name: CI for Terraform Module

on:
  pull_request:
    branches: [ main ]

jobs:
  terraform_validate:
    uses: organization/shared-ci-steps/.github/workflows/terraform-validate.yml@v1.0.0
    with:
      working_directory: './modules/my-module'
      extra_arguments: '-backend=false'
```

### 5.2. CircleCI Integration

Teams using CircleCI can import the reusable commands from the shared library. For example, in their `.circleci/config.yml`:

```yaml
version: 2.1

# Import reusable commands defined in the shared repo (via git submodules or by referencing via configuration)
commands:
  terraform-validate:
    steps:
      - checkout
      - run:
          name: Terraform Init & Validate
          command: |
            cd modules/my-module
            terraform init
            terraform validate

jobs:
  validate:
    docker:
      - image: hashicorp/terraform:latest
    steps:
      - terraform-validate

workflows:
  version: 2
  test:
    jobs:
      - validate
```

---

## Step 6: Testing, Debugging & Continuous Improvement

### 6.1. Local Testing & Pre-Commit Hooks
- Encourage teams to run `terraform validate` and `terraform plan` locally.
- Set up pre-commit hooks (using tools like `pre-commit`) to run formatting and lint checks automatically.

### 6.2. Logging and Error Reporting
- Ensure each CI step logs detailed output.
- Use built-in logging features of GitHub Actions and CircleCI for debugging.
- Include optional debugging flags in reusable workflows.

### 6.3. Feedback and Version Updates
- Collect feedback from teams using the shared library.
- Update documentation and version the changes accordingly.

---

## Step 7: Governance & Collaboration

### 7.1. Code Reviews & Access Control
- Use GitHub branch protection rules to enforce code reviews before merging CI steps.
- Establish a dedicated team to manage the shared CI library.

### 7.2. Shared Ownership & Communication
- Hold periodic reviews to gather input and update CI steps as needed.
- Document changes and release notes with every new version.

---

## Summary

This guide establishes a robust, modular, and shareable CI/CD step library that:
- **Ensures Modularity:** Each CI step is self-contained and parameterized.
- **Promotes Reusability:** Reusable workflows (GitHub Actions) and commands (CircleCI) allow teams to integrate the same steps without duplication.
- **Supports Easy Testing & Debugging:** Local testing, logging, and pre-commit hooks reduce integration issues.
- **Encourages Collaboration:** Clear documentation, version control, and governance ensure teams can share, review, and continuously improve the CI pipeline.

Any team managing a Terraform module repository can now easily add or update CI steps from this library—enabling faster, more reliable, and standardized deployments across the organization.

