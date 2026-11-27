# 🚀 Deployment Guide

Complete guide for deploying MyVote AWS application to production.

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Terraform Deployment](#terraform-deployment)
- [CloudFormation Deployment](#cloudformation-deployment)
- [Manual Deployment](#manual-deployment)
- [Post-Deployment Configuration](#post-deployment-configuration)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Production Checklist](#production-checklist)

---

## Prerequisites

### Required Tools

```bash
# AWS CLI
aws --version  # Should be 2.x or higher

# Terraform (for Terraform deployment)
terraform version  # Should be 1.0 or higher

# Python
python --version  # Should be 3.8 or higher

# Git
git --version
```

### AWS Account Setup

1. **AWS Account**: Active AWS account with billing enabled
2. **IAM User**: User with appropriate permissions
3. **Access Keys**: Configured AWS credentials

```bash
# Configure AWS CLI
aws configure
# Enter:
# - AWS Access Key ID
# - AWS Secret Access Key
# - Default region (e.g., us-east-1)
# - Default output format (json)

# Verify configuration
aws sts get-caller-identity
```

### Required IAM Permissions

Your IAM user/role needs permissions for:
- Lambda (create, update, invoke)
- API Gateway (create, deploy)
- S3 (create buckets, upload objects)
- CloudFront (create distributions)
- DynamoDB (create tables)
- SQS (create queues)
- SNS (create topics)
- IAM (create roles, attach policies)
- CloudWatch (create log groups)

---

## Quick Start

### 1-Command Deployment (Terraform)

```bash
# Clone repository
git clone https://github.com/D1L5H4N/My-Vote.git
cd My-Vote/iac

# Deploy everything
terraform init && terraform apply -auto-approve

# Get outputs
terraform output
```

---

## Terraform Deployment

### Step 1: Prepare Infrastructure Code

```bash
cd iac

# Review variables
cat variables.tf

# (Optional) Create terraform.tfvars for customization
cat > terraform.tfvars <<EOF
project_name = "myvote"
environment  = "production"
aws_region   = "us-east-1"
EOF
```

### Step 2: Initialize Terraform

```bash
# Initialize Terraform
terraform init

# Output:
# Initializing modules...
# Initializing the backend...
# Initializing provider plugins...
```

### Step 3: Plan Deployment

```bash
# Review what will be created
terraform plan

# Save plan to file (optional)
terraform plan -out=tfplan
```

Review the plan carefully. It should create:
- 2 S3 buckets (voting, results)
- 2 CloudFront distributions
- 1 API Gateway
- 3 Lambda functions
- 1 DynamoDB table
- 1 SQS queue
- 1 SNS topic
- Multiple IAM roles and policies

### Step 4: Apply Infrastructure

```bash
# Apply the plan
terraform apply

# Type 'yes' when prompted

# Or use saved plan
terraform apply tfplan
```

**Deployment Time**: ~10-15 minutes

### Step 5: Capture Outputs

```bash
# View all outputs
terraform output

# Save to file
terraform output > deployment-outputs.txt

# Individual outputs
terraform output voting_cloudfront_url
terraform output results_cloudfront_url
terraform output api_gateway_url
```

### Expected Outputs

```
voting_cloudfront_url = "https://d1234abc.cloudfront.net"
results_cloudfront_url = "https://d5678def.cloudfront.net"
api_gateway_url = "https://abc123.execute-api.us-east-1.amazonaws.com/prod"
dynamodb_table_name = "myvote-votes"
sqs_queue_url = "https://sqs.us-east-1.amazonaws.com/123456789/myvote-queue"
sns_topic_arn = "arn:aws:sns:us-east-1:123456789:myvote-topic"
```

---

## CloudFormation Deployment

### Step 1: Prepare Template

```bash
cd iac/cloudformation

# Validate template
aws cloudformation validate-template \
  --template-body file://myvote_cf.yaml
```

### Step 2: Create Stack

```bash
# Create the stack
aws cloudformation create-stack \
  --stack-name myvote-production \
  --template-body file://myvote_cf.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myvote \
    ParameterKey=Environment,ParameterValue=production \
  --capabilities CAPABILITY_IAM \
  --tags \
    Key=Project,Value=MyVote \
    Key=ManagedBy,Value=CloudFormation

# Monitor progress
aws cloudformation describe-stacks \
  --stack-name myvote-production \
  --query 'Stacks[0].StackStatus'
```

### Step 3: Wait for Completion

```bash
# Watch stack creation
aws cloudformation wait stack-create-complete \
  --stack-name myvote-production

# Check events
aws cloudformation describe-stack-events \
  --stack-name myvote-production \
  --max-items 10
```

### Step 4: Get Outputs

```bash
# Get all outputs
aws cloudformation describe-stacks \
  --stack-name myvote-production \
  --query 'Stacks[0].Outputs'
```

---

## Manual Deployment

For learning or troubleshooting, deploy manually:

### 1. Create DynamoDB Table

```bash
aws dynamodb create-table \
  --table-name Votes \
  --attribute-definitions \
    AttributeName=vote,AttributeType=S \
  --key-schema \
    AttributeName=vote,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region us-east-1
```

### 2. Create SNS Topic

```bash
# Create topic
aws sns create-topic --name myvote-topic

# Save topic ARN
SNS_TOPIC_ARN=$(aws sns list-topics --query 'Topics[?contains(TopicArn, `myvote-topic`)].TopicArn' --output text)
echo $SNS_TOPIC_ARN
```

### 3. Create SQS Queue

```bash
# Create queue
aws sqs create-queue --queue-name myvote-queue

# Save queue URL
SQS_QUEUE_URL=$(aws sqs get-queue-url --queue-name myvote-queue --query 'QueueUrl' --output text)
echo $SQS_QUEUE_URL

# Subscribe queue to SNS topic
aws sns subscribe \
  --topic-arn $SNS_TOPIC_ARN \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:us-east-1:ACCOUNT_ID:myvote-queue
```

### 4. Create Lambda Functions

```bash
# Package voting backend
cd voting-backend
zip voting.zip voting.py

# Create Lambda function
aws lambda create-function \
  --function-name myvote-voting-backend \
  --runtime python3.9 \
  --role arn:aws:iam::ACCOUNT_ID:role/lambda-execution-role \
  --handler voting.lambda_handler \
  --zip-file fileb://voting.zip \
  --environment Variables="{SNS_TOPIC_ARN=$SNS_TOPIC_ARN}"
```

Repeat for other Lambda functions (results, vote-processor).

### 5. Create API Gateway

```bash
# Create REST API
aws apigateway create-rest-api \
  --name myvote-api \
  --description "MyVote API"

# Configure resources and methods
# (This is complex - recommend using Terraform/CloudFormation)
```

### 6. Create S3 Buckets

```bash
# Create voting frontend bucket
aws s3 mb s3://myvote-voting-frontend-$RANDOM

# Upload files
aws s3 sync voting-frontend/ s3://myvote-voting-frontend-$RANDOM/ \
  --acl public-read

# Configure website hosting
aws s3 website s3://myvote-voting-frontend-$RANDOM/ \
  --index-document index.html
```

### 7. Create CloudFront Distribution

```bash
# Create distribution (complex - use AWS Console or IaC)
aws cloudfront create-distribution \
  --origin-domain-name myvote-voting-frontend-$RANDOM.s3.amazonaws.com \
  --default-root-object index.html
```

---

## Post-Deployment Configuration

### Update Frontend API Endpoints

After deployment, update the API Gateway URLs in frontend files:

#### 1. Voting Frontend

```bash
# Get API Gateway URL from outputs
API_URL=$(terraform output -raw api_gateway_url)

# Update voting frontend
cd voting-frontend
sed -i "s|\${backend_api_gateway}|${API_URL}|g" index.html

# Re-upload to S3
aws s3 cp index.html s3://YOUR_VOTING_BUCKET/
```

#### 2. Results Frontend

```bash
# Update results frontend
cd result-frontend
sed -i "s|\${backend_api_gateway}|${API_URL}|g" app.js

# Re-upload to S3
aws s3 cp app.js s3://YOUR_RESULTS_BUCKET/
```

### Invalidate CloudFront Cache

```bash
# Get distribution IDs
VOTING_DIST_ID=$(aws cloudfront list-distributions --query "DistributionList.Items[?Comment=='Voting Frontend'].Id" --output text)
RESULTS_DIST_ID=$(aws cloudfront list-distributions --query "DistributionList.Items[?Comment=='Results Frontend'].Id" --output text)

# Create invalidation
aws cloudfront create-invalidation \
  --distribution-id $VOTING_DIST_ID \
  --paths "/*"

aws cloudfront create-invalidation \
  --distribution-id $RESULTS_DIST_ID \
  --paths "/*"
```

---

## Verification

### 1. Test Backend APIs

```bash
# Get API Gateway URL
API_URL=$(terraform output -raw api_gateway_url)

# Test voting endpoint
curl -X POST ${API_URL}/voting \
  -H "Content-Type: application/json" \
  -d '{"MessageAttributes":{"vote":{"Type":"String","StringValue":"a"},"voter":{"Type":"String","StringValue":"test"}}}'

# Expected response: {"statusCode": 200, ...}

# Test results endpoint
curl ${API_URL}/results

# Expected response: {"a": 1, "b": 0}
```

### 2. Test Lambda Functions

```bash
# Invoke voting Lambda directly
aws lambda invoke \
  --function-name myvote-voting-backend \
  --payload '{"body":"{\"vote\":\"a\"}"}' \
  response.json

# Check response
cat response.json

# Invoke results Lambda
aws lambda invoke \
  --function-name myvote-results-backend \
  --payload '{}' \
  response.json

cat response.json
```

### 3. Check CloudWatch Logs

```bash
# View Lambda logs
aws logs tail /aws/lambda/myvote-voting-backend --follow

# View recent errors
aws logs filter-log-events \
  --log-group-name /aws/lambda/myvote-voting-backend \
  --filter-pattern "ERROR"
```

### 4. Verify DynamoDB

```bash
# Scan votes table
aws dynamodb scan --table-name Votes

# Check item count
aws dynamodb describe-table --table-name Votes \
  --query 'Table.ItemCount'
```

### 5. Test Frontend

```bash
# Get CloudFront URLs
VOTING_URL=$(terraform output -raw voting_cloudfront_url)
RESULTS_URL=$(terraform output -raw results_cloudfront_url)

echo "Voting UI: $VOTING_URL"
echo "Results UI: $RESULTS_URL"

# Open in browser
open $VOTING_URL  # macOS
start $VOTING_URL  # Windows
xdg-open $VOTING_URL  # Linux
```

---

## Troubleshooting

### Common Issues

#### 1. CORS Errors

**Problem**: Browser console shows CORS errors

**Solution**:
```bash
# Update API Gateway CORS settings
aws apigateway update-rest-api \
  --rest-api-id YOUR_API_ID \
  --patch-operations \
    op=add,path=/*/OPTIONS/*,value=200
```

#### 2. Lambda Function Errors

**Problem**: 500 errors from API

**Solution**:
```bash
# Check Lambda logs
aws logs tail /aws/lambda/FUNCTION_NAME --follow

# Update Lambda environment variables
aws lambda update-function-configuration \
  --function-name FUNCTION_NAME \
  --environment Variables="{TABLE_NAME=Votes}"
```

#### 3. CloudFront 403 Errors

**Problem**: 403 Forbidden on CloudFront

**Solution**:
```bash
# Check S3 bucket policy allows CloudFront OAI
aws s3api get-bucket-policy --bucket YOUR_BUCKET

# Update bucket policy if needed
```

#### 4. SQS Messages Not Processing

**Problem**: Votes not appearing in DynamoDB

**Solution**:
```bash
# Check SQS queue
aws sqs get-queue-attributes \
  --queue-url $SQS_QUEUE_URL \
  --attribute-names All

# Check Lambda is triggered
aws lambda list-event-source-mappings \
  --function-name myvote-vote-processor
```

---

## Production Checklist

Before going live:

### Security
- [ ] All IAM roles use least privilege
- [ ] CloudFront uses HTTPS only
- [ ] API Gateway has throttling enabled
- [ ] DynamoDB encryption enabled
- [ ] CloudTrail logging enabled
- [ ] Secrets stored in AWS Secrets Manager

### Performance
- [ ] Lambda memory optimized
- [ ] DynamoDB capacity appropriate
- [ ] CloudFront caching configured
- [ ] API Gateway caching enabled (optional)

### Monitoring
- [ ] CloudWatch alarms created
- [ ] SNS notifications for alarms
- [ ] Log aggregation set up
- [ ] Cost alerts configured

### Backup & Recovery
- [ ] DynamoDB backups enabled
- [ ] S3 versioning enabled
- [ ] Disaster recovery plan documented

### Documentation
- [ ] Architecture diagram updated
- [ ] API documentation complete
- [ ] Runbook for operations team
- [ ] Contact information updated

---

## Cleanup

To remove all resources:

### Terraform

```bash
cd iac
terraform destroy -auto-approve
```

### CloudFormation

```bash
aws cloudformation delete-stack --stack-name myvote-production
```

### Manual

```bash
# Delete in reverse order of creation
aws cloudfront delete-distribution --id $DIST_ID --if-match $ETAG
aws s3 rb s3://BUCKET_NAME --force
aws lambda delete-function --function-name FUNCTION_NAME
aws apigateway delete-rest-api --rest-api-id API_ID
aws dynamodb delete-table --table-name Votes
aws sqs delete-queue --queue-url $QUEUE_URL
aws sns delete-topic --topic-arn $TOPIC_ARN
```

---

## Support

Need help? 
- Check [Troubleshooting Guide](README.md#troubleshooting)
- Review [AWS Documentation](https://docs.aws.amazon.com/)
- Open an [Issue](https://github.com/D1L5H4N/My-Vote/issues)

---

**Happy Deploying! 🚀**
