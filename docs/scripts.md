# Scripts and Workflows

## Overview

This project provides five core scripts for managing IAM deployment roles and corresponding GitHub Actions workflows for automation. Each script serves a specific purpose in the deployment lifecycle.

## Core Scripts

### 1. `deploy.sh`
**Purpose**: Deploy all IAM roles and policies for configured projects

**Usage**:
```bash
./scripts/deploy.sh
```

**Functionality**:
- Verifies prerequisites before deployment
- Manages GitHub repository variables (local execution only)
- Assumes foundation deployment role when available
- Configures Terraform backend dynamically
- Deploys all project roles in a single operation
- Provides comprehensive deployment status

**GitHub Workflow**: `deploy.yml` - Triggers on push to main, PRs, and manual dispatch

### 2. `verify-prerequisites.sh`
**Purpose**: Validate environment and prerequisites before deployment

**Usage**:
```bash
./scripts/verify-prerequisites.sh
```

**Functionality**:
- Checks git repository status and cleanliness
- Validates required tools (OpenTofu, AWS CLI, GitHub CLI, jq)
- Verifies AWS authentication and permissions
- Ensures foundation infrastructure is available
- Prevents self-deployment scenarios

**GitHub Workflow**: `verify-prerequisites.yml` - Runs on all pushes and PRs

### 3. `create-project.sh`
**Purpose**: Create new project directory structure and policy templates

**Usage**:
```bash
./scripts/create-project.sh PROJECT_NAME
```

**Functionality**:
- Creates project directory structure under `projects/`
- Generates default deployment policy template
- Validates project name format and availability
- Prevents creation of self-referential projects

**GitHub Workflow**: `create-project.yml` - Manual dispatch with project name input, creates PR

### 4. `destroy.sh`
**Purpose**: Remove all deployed IAM roles and policies

**Usage**:
```bash
./scripts/destroy.sh
```

**Functionality**:
- Performs prerequisite validation
- Assumes foundation deployment role when available
- Destroys all Terraform-managed resources
- Handles resource dependencies correctly
- Provides destruction confirmation

**GitHub Workflow**: `destroy.yml` - Manual dispatch with "DESTROY" confirmation required

### 5. `list-deployed-resources.sh`
**Purpose**: List and verify all deployed resources

**Usage**:
```bash
./scripts/list-deployed-resources.sh
```

**Functionality**:
- Displays Terraform state information
- Lists all IAM roles and policies with details
- Shows SSM parameters and their values
- Provides resource tagging information
- Detects potential orphaned resources

**GitHub Workflow**: `list-deployed-resources.yml` - Runs on pushes, PRs, and manual dispatch

## GitHub Actions Workflows

### Workflow Triggers

| Workflow | Push to Main | Pull Request | Manual Dispatch | Special Requirements |
|----------|--------------|--------------|-----------------|---------------------|
| `deploy` | ✅ (deploy only) | ✅ (validate only) | ✅ | - |
| `verify-prerequisites` | ✅ | ✅ | ✅ | - |
| `create-project` | ❌ | ❌ | ✅ | Project name input |
| `destroy` | ❌ | ❌ | ✅ | "DESTROY" confirmation |
| `list-deployed-resources` | ✅ | ✅ | ✅ | - |

### Authentication

All workflows use OIDC authentication to assume the foundation deployment role:
- **Role ARN**: `arn:aws:iam::{AWS_ACCOUNT_ID}:role/terraform-deployment-roles-{AWS_ACCOUNT_ID}-us-east-1`
- **Region**: `us-east-1`
- **Permissions**: `id-token: write`, `contents: read`

### Workflow Features

- **Automatic Setup**: OpenTofu 1.10.7 installed automatically
- **Credential Management**: No long-lived credentials stored
- **Environment Detection**: Workflows adapt behavior based on trigger type
- **Safety Controls**: Destructive operations require explicit confirmation
- **PR Integration**: Create-project workflow automatically creates pull requests

## Local vs. CI/CD Execution

### Local Execution
- Uses current AWS credentials or assumes foundation role
- Manages GitHub repository variables automatically
- Provides interactive feedback and error messages
- Supports development workflow with git validation

### GitHub Actions Execution
- Uses OIDC to assume foundation deployment role
- Reads pre-configured GitHub repository variables
- Provides structured logging and status reporting
- Integrates with GitHub's security and audit features

## Prerequisites

Before using scripts or workflows:

1. **Foundation Infrastructure**: Deploy [foundation-terraform-bootstrap](https://github.com/stephenabbot/foundation-terraform-bootstrap)
2. **Local Tools**: Install OpenTofu 1.10.7, AWS CLI, GitHub CLI, jq, git
3. **AWS Permissions**: Ensure foundation deployment role has required permissions
4. **GitHub Variables**: `AWS_ACCOUNT_ID` variable (managed automatically by deploy script)

## Error Handling

All scripts include comprehensive error handling:
- **Prerequisite Failures**: Clear guidance on missing requirements
- **Permission Issues**: Detailed error messages with resolution steps
- **State Conflicts**: Automatic lock clearing and state reconciliation
- **Partial Failures**: Safe recovery and rollback procedures

## Best Practices

- **Run Prerequisites First**: Always verify environment before deployment
- **Local Testing**: Test changes locally before relying on GitHub Actions
- **Clean Git State**: Ensure repository is clean before deployment
- **Review Policies**: Validate deployment policies before creating projects
- **Monitor Resources**: Regularly run list-deployed-resources for visibility
