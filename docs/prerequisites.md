# Prerequisites

## Foundation Infrastructure

This project requires [foundation-terraform-bootstrap](https://github.com/stephenabbot/foundation-terraform-bootstrap) to be deployed first. The foundation provides:

- S3 bucket for OpenTofu state storage with the name published to SSM Parameter Store
- DynamoDB table for state locking with the name published to SSM Parameter Store
- OIDC provider for GitHub Actions authentication with the ARN published to SSM Parameter Store
- Foundation deployment role that this project assumes for secure operations

Deploy the foundation project before attempting to use deployment roles. The deployment scripts will automatically discover and use foundation infrastructure through SSM parameter lookup.

**Foundation Deployment Role Permissions**: The foundation deployment role requires `IAMFullAccess` and `PowerUserAccess` managed policies to function as an IAM role factory. See the [Foundation IAM Permissions Strategy](https://github.com/stephenabbot/foundation-terraform-bootstrap/blob/main/docs/iam-permissions-strategy.md) for detailed rationale.

## Required Tools

The following tools must be installed and available in your PATH:

- OpenTofu version 1.10.7 or compatible for infrastructure deployment and state management
- AWS CLI version 2.x for AWS service interaction and credential management
- Bash version 3.2.57 or higher for script execution and automation (macOS default bash supported)
- GitHub CLI for repository variable management and GitHub API interaction
- jq for JSON processing in deployment scripts and parameter handling
- Git for repository information detection and version control

## Git Repository Setup

The repository must be:

- Initialized as a git repository with a remote origin configured pointing to GitHub
- Working directory must be clean with no uncommitted changes, untracked files, or unpushed commits
- Remote origin URL must match the target repository for the deployment role being created

These requirements ensure deployment metadata is accurate and prevent deploying roles with incorrect repository scoping in the OIDC trust policies.

## AWS Permissions

The deployment principal requires the following IAM permissions:

### IAM Permissions

- `iam:CreateRole`
- `iam:CreatePolicy`
- `iam:AttachRolePolicy`
- `iam:GetRole`
- `iam:GetPolicy`
- `iam:ListRoles`
- `iam:ListPolicies`
- `iam:DeleteRole`
- `iam:DeletePolicy`
- `iam:DetachRolePolicy`
- `iam:ListRoleTags`
- `iam:TagRole`

### SSM Permissions

- `ssm:PutParameter`
- `ssm:GetParameter`
- `ssm:DeleteParameter`
- `ssm:GetParameters`
- `ssm:GetParametersByPath`

### Backend Permissions

- `s3:GetObject`
- `s3:PutObject`
- `s3:DeleteObject`
- `dynamodb:GetItem`
- `dynamodb:PutItem`
- `dynamodb:DeleteItem`
- `dynamodb:Scan`

### Role Assumption (Optional)

- `sts:AssumeRole` for foundation deployment role when available

## Verification

Run the prerequisite verification script to validate your environment:

```bash
./scripts/verify-prerequisites.sh
```

This script checks all requirements and provides specific guidance for any missing components.
