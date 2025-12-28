# Architecture

## Overview

This project implements a centralized IAM role management system for GitHub Actions deployments using OpenTofu infrastructure as code. The architecture enables secure, scalable deployment workflows without long-lived credentials.

## Core Components

### OIDC Authentication Flow

```
GitHub Actions → OIDC Provider → IAM Role → AWS Resources
```

1. **GitHub Actions Workflow** requests temporary credentials
2. **OIDC Provider** validates GitHub repository and workflow context
3. **IAM Role** assumes identity based on trust policy conditions
4. **AWS Resources** are accessed using temporary credentials

### Project Structure

```
projects/
├── project-name/
│   └── environment/
│       └── policies/
│           └── deployment-policy.json
```

Dynamic discovery scans this structure to create roles for each project-environment combination.

### Resource Naming

- **IAM Roles:** `gharole-{project}-{environment}`
- **IAM Policies:** `ghpolicy-{project}-{environment}`
- **SSM Parameters:** `/deployment-roles/{project}/role-arn`

## Trust Policy Architecture

### OIDC Conditions

Trust policies use StringLike conditions for repository scoping:

```json
{
  "StringEquals": {
    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
  },
  "StringLike": {
    "token.actions.githubusercontent.com:sub": "repo:org/project:*"
  }
}
```

### Repository Scoping

Each role is scoped to a specific GitHub repository:
- Prevents cross-repository role assumption
- Enables fine-grained access control
- Supports multiple environments per repository

## Backend Architecture

### State Management

- **S3 Bucket** stores OpenTofu state files
- **DynamoDB Table** provides state locking
- **State Key Pattern:** `deployment-roles/{org-repo}/terraform.tfstate`

### Foundation Dependencies

Retrieves backend configuration from foundation project:
- `/terraform/foundation/s3-state-bucket`
- `/terraform/foundation/dynamodb-lock-table`
- `/terraform/foundation/oidc-provider`

## Service Discovery

### SSM Parameter Store

Role ARNs are published to predictable SSM parameter paths:
- **Path Pattern:** `/deployment-roles/{project}/role-arn`
- **Region:** us-east-1 (centralized)
- **Type:** String parameter

### Consuming Project Integration

Projects discover their deployment roles:

```bash
ROLE_ARN=$(aws ssm get-parameter --name "/deployment-roles/my-project/role-arn" --query Parameter.Value --output text)
```

## Security Architecture

### Least Privilege Progression

1. **Initial Deployment** - Broad permissions for discovery
2. **CloudTrail Analysis** - Identify actual permissions used
3. **Policy Refinement** - Reduce to minimum required permissions
4. **Ongoing Monitoring** - Continuous permission optimization

### Trust Boundary

- **Repository Isolation** - Each role scoped to single repository
- **Environment Separation** - Separate roles per environment
- **Temporal Credentials** - No long-lived access keys

### Audit Trail

- **CloudTrail Logging** - All API calls logged
- **Resource Tagging** - Deployment principal tracking
- **SSM Parameter History** - Role ARN change tracking

## Operational Architecture

### Deployment Workflow

1. **Prerequisite Validation** - Environment and permissions check
2. **Foundation Role Assumption** - Secure deployment context
3. **Backend Configuration** - Dynamic state backend setup
4. **Resource Deployment** - OpenTofu apply operation
5. **Verification** - Resource listing and validation

### Self-Deployment Prevention

- **Git Repository Detection** - Prevents circular dependencies
- **Project Name Validation** - Blocks self-referential configurations
- **Deployment Safety** - Ensures operational integrity

### Idempotent Operations

- **Multiple Executions** - Safe to run repeatedly
- **State Reconciliation** - Handles configuration drift
- **Error Recovery** - Graceful handling of partial failures

## Scalability Considerations

### Multi-Project Support

- **Dynamic Discovery** - Automatic project detection
- **Parallel Deployment** - Single OpenTofu configuration
- **Resource Isolation** - Independent project lifecycles

### Regional Architecture

- **Control Plane** - us-east-1 for IAM and SSM
- **Data Plane** - Regional resource deployment
- **Cross-Region** - Consistent role discovery

### Performance Optimization

- **Batch Operations** - Single deployment for all projects
- **State Efficiency** - Optimized state file structure
- **Parameter Caching** - SSM parameter optimization

## Integration Patterns

### GitHub Actions Integration

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - name: Configure AWS Credentials
    uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: ${{ env.ROLE_ARN }}
      aws-region: us-east-1
```

### Multi-Repository Pattern

- **Centralized Role Management** - Single repository for all roles
- **Distributed Consumption** - Multiple repositories use roles
- **Decoupled Lifecycle** - Independent deployment and consumption

### Foundation Integration

- **Shared Infrastructure** - Common OIDC provider and backend
- **Consistent Configuration** - Aligned tagging and naming
- **Operational Harmony** - Coordinated deployment workflows

## Monitoring and Observability

### CloudTrail Integration

- **API Call Tracking** - All IAM operations logged
- **Role Usage Monitoring** - AssumeRole event tracking
- **Permission Analysis** - Actual vs granted permissions

### Resource Tracking

- **Tag-Based Monitoring** - Cost and usage tracking
- **State File Analysis** - Resource drift detection
- **Parameter Monitoring** - SSM parameter access patterns

### Operational Metrics

- **Deployment Success Rate** - Workflow reliability tracking
- **Role Assumption Patterns** - Usage analysis
- **Permission Optimization** - Security posture improvement
