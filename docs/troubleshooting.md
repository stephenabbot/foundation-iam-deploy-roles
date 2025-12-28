# Troubleshooting

## Common Issues

### Role Assumption Failures

**Symptom:** GitHub Actions cannot assume the deployment role

**Causes and Solutions:**

1. **OIDC Provider Missing**
   - Verify foundation infrastructure is deployed
   - Check SSM parameter `/terraform/foundation/oidc-provider` exists
   - Ensure OIDC provider has correct GitHub thumbprints

2. **Repository Name Mismatch**
   - Verify role trust policy includes correct repository name
   - Check GitHub repository name matches exactly what was configured
   - Repository names are case-sensitive

3. **GitHub Actions Configuration**
   - Ensure workflow has `id-token: write` permissions
   - Verify `aws-actions/configure-aws-credentials` action configuration
   - Check role ARN is correctly retrieved from SSM Parameter Store

### Policy Attachment Issues

**Symptom:** Deployment role lacks necessary permissions

**Causes and Solutions:**

1. **Invalid Policy JSON**
   - Validate JSON syntax in deployment-policy.json files
   - Check policy document does not exceed AWS size limits
   - Ensure policy actions and resources are correctly formatted

2. **Policy Not Attached**
   - Verify policy attachment exists in AWS console
   - Check OpenTofu state shows policy attachment resource
   - Run `./scripts/list-deployed-resources.sh` to verify

3. **Insufficient Permissions**
   - Start with broad permissions and refine based on CloudTrail analysis
   - Test role permissions by assuming role manually with AWS CLI
   - Review AWS CloudTrail logs for permission denied errors

### SSM Parameter Access

**Symptom:** Consuming projects cannot find role ARN

**Causes and Solutions:**

1. **Parameter Path Incorrect**
   - Verify parameter exists at `/deployment-roles/{project-name}/role-arn`
   - Check parameter is in us-east-1 region
   - Ensure project name matches directory name exactly

2. **Parameter Permissions**
   - Verify consuming project has `ssm:GetParameter` permissions
   - Check parameter is not encrypted with inaccessible KMS key
   - Ensure parameter value contains valid role ARN

3. **Parameter Construction**
   - Parameter path uses project directory name, not GitHub repository name
   - Path format: `/deployment-roles/{project-name}/role-arn`
   - Use AWS CLI to verify parameter exists: `aws ssm get-parameter --name "/deployment-roles/my-project/role-arn"`

### Deployment Script Failures

**Symptom:** Deploy script fails with errors

**Causes and Solutions:**

1. **Prerequisites Not Met**
   - Run `./scripts/verify-prerequisites.sh` first
   - Resolve all prerequisite failures before deployment
   - Ensure git repository is clean and pushed

2. **Backend Configuration Issues**
   - Verify foundation infrastructure is deployed
   - Check SSM parameters for S3 bucket and DynamoDB table exist
   - Ensure current credentials can access backend resources

3. **State Lock Issues**
   - Clear stale DynamoDB locks if deployment was interrupted
   - Check DynamoDB table for lock entries
   - Deploy script automatically clears stale locks

4. **Role Assumption Failures**
   - Foundation deployment role assumption is optional
   - Script continues with current credentials if assumption fails
   - Ensure current credentials have required permissions

### Project Creation Issues

**Symptom:** Cannot create new project with create-project.sh

**Causes and Solutions:**

1. **Self-Deployment Prevention**
   - Cannot create deployment role for the current project itself
   - This prevents circular dependencies
   - Use different project name or deploy from different repository

2. **Role Name Too Long**
   - Role names have 64 character limit
   - Format: `gharole-{project-name}-{environment}`
   - Use shorter project names if limit exceeded

3. **Project Already Exists**
   - Check if project directory already exists
   - Remove existing project directory or use different name
   - Verify project was not previously created

### Resource Cleanup Issues

**Symptom:** Destroy script fails to remove all resources

**Causes and Solutions:**

1. **Policy Attachments**
   - Policies must be detached before deletion
   - Destroy script handles this automatically
   - Manual cleanup may be needed if script fails

2. **Resource Dependencies**
   - IAM resources have dependency order requirements
   - Destroy script processes in correct order
   - Check AWS console for remaining resources if script fails

3. **Permissions Issues**
   - Ensure current credentials can delete all resource types
   - Foundation deployment role provides necessary permissions
   - Manual cleanup may require elevated permissions

## Diagnostic Commands

### Check Role Status

```bash
# List all deployment roles
aws iam list-roles --query 'Roles[?starts_with(RoleName, `gharole-`)]'

# Get specific role details
aws iam get-role --role-name gharole-my-project-prd

# Check role trust policy
aws iam get-role --role-name gharole-my-project-prd --query 'Role.AssumeRolePolicyDocument'
```

### Check SSM Parameters

```bash
# List all deployment role parameters
aws ssm get-parameters-by-path --path "/deployment-roles" --recursive

# Get specific parameter
aws ssm get-parameter --name "/deployment-roles/my-project/role-arn"
```

### Check Backend State

```bash
# List state files
aws s3 ls s3://your-state-bucket/deployment-roles/

# Check DynamoDB locks
aws dynamodb scan --table-name your-lock-table --filter-expression "contains(LockID, :key)" --expression-attribute-values '{"key":{"S":"deployment-roles"}}'
```

### Test Role Assumption

```bash
# Assume role manually
aws sts assume-role --role-arn "arn:aws:iam::123456789012:role/gharole-my-project-prd" --role-session-name "test-session"

# Test with GitHub Actions format
aws sts assume-role-with-web-identity --role-arn "arn:aws:iam::123456789012:role/gharole-my-project-prd" --role-session-name "github-actions" --web-identity-token "token"
```

## Getting Help

1. Run `./scripts/verify-prerequisites.sh` to check environment
2. Run `./scripts/list-deployed-resources.sh` to verify current state
3. Check AWS CloudTrail logs for detailed error information
4. Review OpenTofu plan output for resource changes
5. Consult AWS documentation for service-specific issues
