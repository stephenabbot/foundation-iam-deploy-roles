# Resource Tagging

## Tagging Strategy

All resources created by this project follow a consistent tagging strategy aligned with foundation standards for cost allocation, ownership tracking, and resource management.

## Standard Tags

### Base Tags

Applied to all resources:

- **Project** - Git repository name extracted from remote origin
- **Repository** - Full repository URL for traceability
- **Environment** - Environment name from project directory structure
- **Owner** - Resource owner (default: StephenAbbot)
- **ManagedBy** - Management tool (OpenTofu)
- **DeploymentId** - Deployment identifier (default: Default)

### Conditional Tags

Applied when available:

- **DeployedBy** - IAM principal ARN that performed the deployment

## Tag Sources

### Git Repository Detection

Project and repository tags are automatically extracted from git remote origin:

```bash
git remote get-url origin
```

Supports both HTTPS and SSH GitHub URLs:
- `https://github.com/org/repo.git`
- `git@github.com:org/repo.git`

### Environment Detection

Environment tag comes from project directory structure:
- Path: `projects/{project-name}/{environment}/policies/deployment-policy.json`
- Example: `projects/my-project/prd/policies/deployment-policy.json` → Environment: `prd`

### Deployment Principal

DeployedBy tag captures the IAM principal performing the deployment:
- Uses `aws sts get-caller-identity` to get current principal ARN
- Includes assumed roles when using foundation deployment role
- Provides audit trail for deployment actions

## Resource-Specific Tagging

### IAM Roles

Tags applied to deployment roles:

```json
{
  "Project": "foundation-iam-deploy-roles",
  "Repository": "https://github.com/stephenabbot/foundation-iam-deploy-roles.git",
  "Environment": "prd",
  "Owner": "StephenAbbot",
  "ManagedBy": "OpenTofu",
  "DeploymentId": "Default",
  "DeployedBy": "arn:aws:iam::123456789012:user/admin"
}
```

### IAM Policies

Same tag set as roles for consistency and cost allocation.

### SSM Parameters

Tagged for parameter management and cost tracking:
- Enables filtering parameters by project or environment
- Supports automated cleanup based on tags
- Provides ownership and management context

## Cost Allocation

### Project-Based Allocation

Primary cost allocation dimension:
- **Project** tag enables cost reports by repository
- Supports chargeback to project teams
- Enables cost optimization analysis per project

### Environment-Based Allocation

Secondary cost allocation dimension:
- **Environment** tag separates production from development costs
- Supports environment-specific cost controls
- Enables cost comparison across environments

### Owner-Based Allocation

Tertiary cost allocation dimension:
- **Owner** tag enables cost reports by team or individual
- Supports organizational cost allocation
- Enables accountability for resource costs

## Tag Validation

### Automatic Validation

OpenTofu validates tag consistency:
- All resources receive same base tag set
- Tag values are consistent across related resources
- Missing tags cause deployment failures

### Manual Validation

Verify tagging with AWS CLI:

```bash
# Check role tags
aws iam list-role-tags --role-name gharole-my-project-prd

# Check policy tags
aws iam list-policy-tags --policy-arn arn:aws:iam::123456789012:policy/ghpolicy-my-project-prd

# Check SSM parameter tags
aws ssm list-tags-for-resource --resource-type Parameter --resource-id "/deployment-roles/my-project/role-arn"
```

## Tag Governance

### Required Tags

All resources must have:
- Project
- Repository
- Environment
- Owner
- ManagedBy

### Optional Tags

Additional tags can be added via module variables:
- Custom project-specific tags
- Compliance or security tags
- Business unit or cost center tags

### Tag Policies

Consider implementing AWS Organizations tag policies:
- Enforce required tag presence
- Validate tag value formats
- Prevent tag modification by unauthorized principals

## Cost Reporting

### AWS Cost Explorer

Use tags for cost analysis:
- Group by Project tag for repository-level costs
- Group by Environment tag for environment comparison
- Group by Owner tag for team-level allocation

### Third-Party Tools

Tags support integration with:
- CloudHealth for cost optimization
- Cloudability for cost management
- Custom cost reporting tools

## Cleanup and Lifecycle

### Tag-Based Cleanup

Use tags for automated resource cleanup:
- Identify resources by project for bulk operations
- Filter by environment for development resource cleanup
- Use ManagedBy tag to identify OpenTofu-managed resources

### Lifecycle Management

Tags support lifecycle policies:
- Archive development environments based on age
- Implement cost controls based on project tags
- Automate compliance reporting using tag data
