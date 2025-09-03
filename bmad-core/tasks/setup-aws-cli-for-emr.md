# Setup AWS CLI for EMR Operations

**Purpose**: Configure AWS CLI and verify permissions for EMR cluster operations, logging, and cost analysis.

**Elicit**: true

## Prerequisites Check

Let me verify your AWS CLI setup and permissions:

1. **Do you have AWS CLI installed?**
   - Run: `aws --version`
   - If not installed, we'll guide you through installation

2. **Are you authenticated with AWS?**
   - Run: `aws sts get-caller-identity`
   - Do you see your account details and user/role information?

3. **What AWS region are your EMR clusters in?**
   - Example: `us-west-2`, `us-east-1`, `eu-west-1`
   - We'll set this as your default region

## AWS CLI Installation

### Step 1: Install AWS CLI (if needed)

**macOS:**

```bash
# Using Homebrew (recommended)
brew install awscli

# Or using pip
pip3 install awscli
```

**Linux:**

```bash
# Using pip
pip3 install awscli

# Or using package manager (Ubuntu/Debian)
sudo apt update
sudo apt install awscli
```

**Windows:**

```powershell
# Download and run AWS CLI MSI installer from:
# https://aws.amazon.com/cli/
```

**Verify Installation:**

```bash
aws --version
# Should show: aws-cli/2.x.x Python/3.x.x
```

### Step 2: Configure AWS CLI

**Option A: Using AWS Configure (Interactive)**

```bash
aws configure
# Enter:
# AWS Access Key ID: [Your access key]
# AWS Secret Access Key: [Your secret key]
# Default region name: [e.g., us-west-2]
# Default output format: json
```

**Option B: Using Environment Variables**

```bash
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-west-2"
```

**Option C: Using IAM Role (EC2/ECS/Lambda)**

```bash
# If running on AWS infrastructure, ensure IAM role has required permissions
aws sts get-caller-identity
```

### Step 3: Verify Authentication

```bash
# Test basic connectivity
aws sts get-caller-identity

# Expected output:
{
    "UserId": "AIDACKCEVSQ6C2EXAMPLE",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/YourUsername"
}
```

## Required IAM Permissions

### Step 4: Verify EMR Permissions

**Minimum Required Permissions:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "elasticmapreduce:ListClusters",
        "elasticmapreduce:DescribeCluster",
        "elasticmapreduce:ListSteps",
        "elasticmapreduce:DescribeStep",
        "elasticmapreduce:ListInstances",
        "elasticmapreduce:ListInstanceGroups",
        "elasticmapreduce:GetManagedScalingPolicy",
        "elasticmapreduce:CreatePersistentAppUI",
        "elasticmapreduce:DescribePersistentAppUI",
        "elasticmapreduce:GetPersistentAppUIPresignedURL",
        "elasticmapreduce:ListPersistentAppUIs"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket", "s3:GetBucketLocation"],
      "Resource": ["arn:aws:s3:::your-emr-logs-bucket", "arn:aws:s3:::your-emr-logs-bucket/*"]
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeInstanceTypes",
        "ec2:DescribeSpotPriceHistory"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": ["pricing:GetProducts", "pricing:DescribeServices"],
      "Resource": "*"
    }
  ]
}
```

**Test Permissions:**

```bash
# Test EMR access
aws emr list-clusters --region us-west-2

# Test S3 access (replace with your bucket)
aws s3 ls s3://your-emr-logs-bucket/ --region us-west-2

# Test EC2 access
aws ec2 describe-instances --region us-west-2 --max-items 1
```

## Advanced Configuration

### Step 5: Configure AWS CLI Profiles (Optional)

For multiple AWS accounts or regions:

```bash
# Configure additional profiles
aws configure --profile emr-production
aws configure --profile emr-staging

# Use specific profile
aws emr list-clusters --profile emr-production --region us-west-2

# Set environment variable for profile
export AWS_PROFILE=emr-production
```

### Step 6: Set Up Region-Specific Configurations

```bash
# Set default region for EMR operations
aws configure set region us-west-2

# Or use environment variable
export AWS_DEFAULT_REGION=us-west-2

# Verify configuration
aws configure list
```

### Step 7: Configure Output Formatting

```bash
# Set JSON output (recommended for parsing)
aws configure set output json

# Alternative formats: table, text, yaml
aws configure set output table
```

## Validation Tests

### Step 8: Comprehensive Permission Testing

**EMR Operations:**

```bash
# List all clusters
aws emr list-clusters --region us-west-2

# Test cluster description (replace with actual cluster ID)
aws emr describe-cluster --cluster-id j-XXXXXXXXXXXXX --region us-west-2

# Test step listing
aws emr list-steps --cluster-id j-XXXXXXXXXXXXX --region us-west-2

# Test instance listing
aws emr list-instances --cluster-id j-XXXXXXXXXXXXX --region us-west-2
```

**S3 Log Access:**

```bash
# Test log bucket access
aws s3 ls s3://your-emr-logs-bucket/logs/ --region us-west-2

# Test log file access
aws s3 ls s3://your-emr-logs-bucket/logs/j-XXXXXXXXXXXXX/ --region us-west-2
```

**Cost Analysis:**

```bash
# Test pricing API access
aws pricing describe-services --service-code AmazonEMR --region us-east-1

# Test EC2 pricing for instance analysis
aws ec2 describe-spot-price-history --instance-types m5.xlarge --max-items 1 --region us-west-2
```

## Optimization Settings

### Step 9: Performance Optimizations

**CLI Configuration:**

```bash
# Increase CLI timeout for large operations
aws configure set cli_read_timeout 0
aws configure set cli_connect_timeout 60

# Enable CLI paging for large results
aws configure set cli_pager ''

# Set maximum items for list operations
aws configure set max_items 1000
```

**Environment Variables:**

```bash
# Disable SSL verification if behind corporate firewall (not recommended for production)
# export AWS_CA_BUNDLE=/path/to/ca-bundle.pem

# Enable debug logging
export AWS_CLI_DEBUG=1

# Set custom endpoint (for VPC endpoints)
# export AWS_ENDPOINT_URL_EMR=https://elasticmapreduce.us-west-2.amazonaws.com
```

## Troubleshooting Common Issues

### Authentication Problems

**Issue: "Unable to locate credentials"**

```bash
# Solution 1: Check configuration
aws configure list

# Solution 2: Set environment variables
export AWS_ACCESS_KEY_ID="your-key"
export AWS_SECRET_ACCESS_KEY="your-secret"

# Solution 3: Use IAM role (if on EC2)
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

**Issue: "Access Denied"**

```bash
# Check current identity
aws sts get-caller-identity

# Check specific permission
aws iam simulate-principal-policy \
    --policy-source-arn arn:aws:iam::123456789012:user/YourUser \
    --action-names elasticmapreduce:ListClusters \
    --resource-arns "*"
```

### Region Issues

**Issue: "Cluster not found"**

```bash
# List all regions
aws ec2 describe-regions --query 'Regions[].RegionName'

# Search for clusters in all regions
for region in $(aws ec2 describe-regions --query 'Regions[].RegionName' --output text); do
    echo "Checking region: $region"
    aws emr list-clusters --region $region --max-items 5
done
```

### Network Issues

**Issue: "Connection timeout"**

```bash
# Test connectivity
curl -I https://elasticmapreduce.us-west-2.amazonaws.com

# Use VPC endpoint if available
aws configure set emr.endpoint_url https://vpce-xxxxx-emr.us-west-2.vpce.amazonaws.com
```

## Success Validation

### Step 10: Final Verification

Run these commands to ensure everything is working:

```bash
# 1. Authentication check
aws sts get-caller-identity

# 2. EMR access check
aws emr list-clusters --region us-west-2 --max-items 5

# 3. S3 access check (if you have EMR log bucket)
aws s3 ls s3://your-emr-logs-bucket/ --region us-west-2 | head -5

# 4. EC2 access check
aws ec2 describe-instances --region us-west-2 --max-items 1

# 5. Pricing API check
aws pricing describe-services --service-code AmazonEMR --region us-east-1
```

**Success Indicators:**

- ✅ All commands execute without permission errors
- ✅ JSON responses are properly formatted
- ✅ Cluster and instance data is retrieved
- ✅ S3 log bucket is accessible
- ✅ Pricing API responds correctly

## Integration with Spark EMR Expert

Once AWS CLI is configured, you can use these commands:

- `*list-emr-clusters` - Discover available clusters
- `*describe-emr-cluster {cluster-id}` - Get detailed cluster info
- `*query-emr-steps {cluster-id}` - Analyze step execution
- `*check-emr-instances {cluster-id}` - Check instance health
- `*fetch-emr-logs {cluster-id}` - Retrieve logs via S3
- `*aws-cost-analysis {cluster-id}` - Analyze costs and optimization

## Security Best Practices

1. **Use IAM Roles** instead of access keys when possible
2. **Rotate Access Keys** regularly
3. **Use Least Privilege** - only grant necessary permissions
4. **Enable CloudTrail** to log API calls
5. **Use MFA** for sensitive operations
6. **Store Credentials Securely** - never commit to version control
7. **Use AWS SSO** for enterprise environments

Your AWS CLI is now ready for comprehensive EMR analysis and debugging!
