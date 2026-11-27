# 📁 Project Structure

This document provides a comprehensive overview of the MyVote AWS project structure and organization.

## 📂 Directory Overview

```
My-Vote/
├── .github/                      # GitHub configuration
│   ├── ISSUE_TEMPLATE/          # Issue templates
│   │   ├── bug_report.md       # Bug report template
│   │   └── feature_request.md  # Feature request template
│   ├── workflows/               # GitHub Actions
│   │   └── ci.yml              # CI/CD pipeline
│   └── pull_request_template.md # PR template
│
├── dynamodb/                     # DynamoDB configuration
│   └── README.md                # DynamoDB setup guide
│
├── gateway/                      # API Gateway documentation
│   └── README.md                # API Gateway setup guide
│
├── iac/                          # Infrastructure as Code (Primary)
│   ├── cloudformation/          # CloudFormation templates
│   │   ├── myvote_cf.yaml      # Main CF template
│   │   └── readme.md           # CF deployment guide
│   │
│   ├── result/                  # Results frontend templates
│   │   ├── app.js.tpl          # App.js template
│   │   ├── index.html          # HTML template
│   │   ├── style.css           # Styling
│   │   └── myicon.png          # Favicon
│   │
│   ├── vote/                    # Voting frontend templates
│   │   ├── index.html.tpl      # HTML template
│   │   ├── style.css           # Styling
│   │   └── myicon.png          # Favicon
│   │
│   ├── apigateway.tf            # API Gateway resources
│   ├── cloudfront_result.tf     # Results CloudFront distribution
│   ├── cloudfront_vote.tf       # Voting CloudFront distribution
│   ├── dynamodb.tf              # DynamoDB table definition
│   ├── main.tf                  # Main Terraform configuration
│   ├── result.py                # Results Lambda function
│   ├── result.zip               # Results Lambda package
│   ├── result_lambda_backend.tf # Results Lambda infrastructure
│   ├── s3_bucket_result.tf      # Results S3 bucket
│   ├── s3_bucket_vote.tf        # Voting S3 bucket
│   ├── sns.tf                   # SNS topic configuration
│   ├── sqs.tf                   # SQS queue configuration
│   ├── variables.tf             # Terraform variables
│   ├── vote_processor.py        # Vote processor function
│   ├── vote_processor.zip       # Vote processor package
│   ├── vote_processor_lambda_backend.tf # Processor Lambda infra
│   ├── voting.py                # Voting Lambda function
│   ├── voting.zip               # Voting Lambda package
│   ├── vote_lambda_backend.tf   # Voting Lambda infrastructure
│   ├── terraform.tfstate        # Terraform state (gitignored)
│   ├── terraform.tfstate.backup # State backup (gitignored)
│   └── readme.md                # IaC documentation
│
├── iac_done_former2/             # Alternative IaC implementations
│   ├── myvote_aws_cloudformation.yml # CF template
│   ├── myvote_awscdk.py         # AWS CDK implementation
│   ├── myvote_troposphere.py    # Troposphere implementation
│   ├── myvoteaws_terraform.tf   # Alternative Terraform
│   └── readme.md                # Alternative IaC guide
│
├── result-backend/               # Results Backend Lambda
│   ├── results.py               # Lambda handler
│   └── README.md                # Setup and deployment guide
│
├── result-frontend/              # Results Frontend UI
│   ├── app.js                   # Application logic
│   ├── index.html               # Main HTML
│   ├── style.css                # Styling
│   ├── myicon.png               # Favicon
│   └── README.md                # Frontend documentation
│
├── vote-processor/               # Vote Processor Lambda
│   ├── processor.py             # Standalone processor
│   ├── processor_lambda.py      # Lambda handler
│   ├── lambda_can_access_dynamodb_policy.json # IAM policy
│   ├── lambda_can_access_sqs_policy.json      # IAM policy
│   └── README.md                # Processor documentation
│
├── voting-backend/               # Voting Backend Lambda
│   ├── voting.py                # Lambda handler
│   └── README.md                # Backend documentation
│
├── voting-frontend/              # Voting Frontend UI
│   ├── index.html               # Main HTML
│   ├── style.css                # Styling
│   ├── myicon.png               # Favicon
│   └── README.md                # Frontend documentation
│
├── .gitignore                    # Git ignore rules
├── CHANGELOG.md                  # Version history
├── CONTRIBUTING.md               # Contribution guidelines
├── DEPLOYMENT.md                 # Deployment guide
├── LICENSE                       # MIT License
├── PROJECT_STRUCTURE.md          # This file
├── README.md                     # Main documentation
└── SECURITY.md                   # Security policy

```

## 🗂️ Component Details

### Frontend Components

#### Voting Frontend (`/voting-frontend`)
**Purpose**: User interface for casting votes

**Files:**
- `index.html` - Main voting page with A/B buttons
- `style.css` - Custom styling and animations
- `myicon.png` - Application favicon

**Technology**: HTML5, CSS3, JavaScript (jQuery)

**Deployment**: Hosted on S3, served via CloudFront

#### Results Frontend (`/result-frontend`)
**Purpose**: Real-time voting results visualization

**Files:**
- `index.html` - Results display page
- `app.js` - Polling logic and chart updates
- `style.css` - Custom styling
- `myicon.png` - Application favicon

**Technology**: HTML5, CSS3, JavaScript (jQuery)

**Features:**
- Auto-refresh every 3 seconds
- Animated percentage bars
- Vote count display

### Backend Components

#### Voting Backend (`/voting-backend`)
**Purpose**: Accept and process vote submissions

**Files:**
- `voting.py` - Lambda handler function
- `README.md` - Setup documentation

**Function**: 
```python
def lambda_handler(event, context):
    # Receives vote from API Gateway
    # Publishes to SNS topic
    # Returns success/error response
```

**IAM Requirements**: SNS Publish permissions

#### Results Backend (`/result-backend`)
**Purpose**: Query and return voting statistics

**Files:**
- `results.py` - Lambda handler function
- `README.md` - Setup documentation

**Function**:
```python
def lambda_handler(event, context):
    # Queries DynamoDB
    # Aggregates vote counts
    # Returns JSON response
```

**IAM Requirements**: DynamoDB Read permissions

#### Vote Processor (`/vote-processor`)
**Purpose**: Process votes from queue and store in database

**Files:**
- `processor_lambda.py` - Lambda handler (serverless)
- `processor.py` - Standalone script (EC2)
- `lambda_can_access_dynamodb_policy.json` - IAM policy
- `lambda_can_access_sqs_policy.json` - IAM policy
- `README.md` - Processor documentation

**Function**:
```python
def lambda_handler(event, context):
    # Consumes from SQS queue
    # Processes vote data
    # Writes to DynamoDB
```

**IAM Requirements**: SQS Read, DynamoDB Write permissions

### Infrastructure as Code

#### Terraform (`/iac`)
**Primary IaC implementation**

**Key Files:**
- `main.tf` - Provider configuration, backend
- `variables.tf` - Input variables
- `apigateway.tf` - REST API definition
- `cloudfront_vote.tf` - Voting CDN
- `cloudfront_result.tf` - Results CDN
- `dynamodb.tf` - Database table
- `s3_bucket_vote.tf` - Voting frontend bucket
- `s3_bucket_result.tf` - Results frontend bucket
- `sns.tf` - Notification topic
- `sqs.tf` - Message queue
- `vote_lambda_backend.tf` - Voting Lambda
- `result_lambda_backend.tf` - Results Lambda
- `vote_processor_lambda_backend.tf` - Processor Lambda

**Usage:**
```bash
cd iac
terraform init
terraform plan
terraform apply
```

#### CloudFormation (`/iac/cloudformation`)
**Alternative IaC implementation**

**Files:**
- `myvote_cf.yaml` - Complete stack template
- `readme.md` - Deployment instructions

**Usage:**
```bash
aws cloudformation create-stack \
  --stack-name myvote \
  --template-body file://myvote_cf.yaml \
  --capabilities CAPABILITY_IAM
```

#### AWS CDK (`/iac_done_former2`)
**Additional IaC options**

**Files:**
- `myvote_awscdk.py` - CDK implementation
- `myvote_troposphere.py` - Troposphere implementation

### Documentation

#### Root Level Documentation

| File | Purpose |
|------|---------|
| `README.md` | Main project documentation, quick start |
| `DEPLOYMENT.md` | Detailed deployment instructions |
| `CONTRIBUTING.md` | Contribution guidelines |
| `SECURITY.md` | Security policy and best practices |
| `CHANGELOG.md` | Version history and changes |
| `PROJECT_STRUCTURE.md` | This file - project organization |
| `LICENSE` | MIT License terms |

#### Component Documentation
Each component directory includes:
- `README.md` - Component-specific setup and usage

### Configuration Files

#### Git Configuration
- `.gitignore` - Files/directories to exclude from git
  - Terraform state files
  - Python cache
  - AWS credentials
  - Build artifacts
  - IDE configurations

#### CI/CD Configuration
- `.github/workflows/ci.yml` - GitHub Actions pipeline
  - Python linting
  - Terraform validation
  - CloudFormation validation
  - Security scanning

#### Templates
- `.github/ISSUE_TEMPLATE/` - Issue templates
- `.github/pull_request_template.md` - PR template

## 🔄 Data Flow

```
User Browser (Voting)
    ↓
CloudFront (Voting)
    ↓
S3 Bucket (Static Site)
    ↓
API Gateway (/voting)
    ↓
Lambda (Voting Backend)
    ↓
SNS Topic
    ↓
SQS Queue
    ↓
Lambda (Vote Processor)
    ↓
DynamoDB (Votes Table)
    ↑
Lambda (Results Backend)
    ↑
API Gateway (/results)
    ↑
S3 Bucket (Static Site)
    ↑
CloudFront (Results)
    ↑
User Browser (Results)
```

## 📦 Deployment Artifacts

### Lambda Packages
- `iac/voting.zip` - Voting backend deployment package
- `iac/result.zip` - Results backend deployment package
- `iac/vote_processor.zip` - Vote processor deployment package

**Building Packages:**
```bash
# Example for voting backend
cd voting-backend
zip -r ../iac/voting.zip voting.py
```

### Frontend Builds
Frontends are static files deployed to S3:
- No build process required
- Direct upload to S3 buckets
- Served via CloudFront CDN

## 🔑 Key Dependencies

### Python (Backend)
```
boto3==1.26.137  # AWS SDK for Python
```

### JavaScript (Frontend)
```
jQuery 3.6.0  # DOM manipulation and AJAX
Font Awesome 4.4.0  # Icons
```

### Infrastructure
```
Terraform >= 1.0
AWS CLI >= 2.0
Python >= 3.8
```

## 🏗️ Architecture Patterns

### Microservices
- Decoupled components
- Independent deployment
- Service-specific scaling

### Event-Driven
- SNS/SQS for async processing
- Loose coupling
- Resilient to failures

### Serverless
- No server management
- Pay-per-use pricing
- Auto-scaling

## 📊 Resource Naming Convention

Format: `{project}-{component}-{environment}`

Examples:
- Lambda: `myvote-voting-backend-prod`
- S3: `myvote-voting-frontend-prod`
- DynamoDB: `myvote-votes-prod`
- SQS: `myvote-queue-prod`

## 🔍 Finding Things

### Need to modify...

**Vote submission logic**: 
→ `voting-backend/voting.py`

**Results display**: 
→ `result-frontend/app.js` and `result-frontend/index.html`

**Vote processing**: 
→ `vote-processor/processor_lambda.py`

**Infrastructure**: 
→ `iac/*.tf` (Terraform) or `iac/cloudformation/myvote_cf.yaml` (CloudFormation)

**UI styling**: 
→ `voting-frontend/style.css` or `result-frontend/style.css`

**API endpoints**: 
→ `iac/apigateway.tf`

**Database schema**: 
→ `iac/dynamodb.tf` or `dynamodb/README.md`

## 📝 Best Practices

### File Organization
✅ Keep related files together
✅ Use clear, descriptive names
✅ Include README in each major directory
✅ Separate IaC from application code

### Documentation
✅ Update README when adding features
✅ Document IAM policies
✅ Include deployment instructions
✅ Add inline code comments

### Version Control
✅ Commit infrastructure separately from code
✅ Use meaningful commit messages
✅ Tag releases
✅ Keep sensitive data out of git

---

## 🤝 Contributing

When adding new components:
1. Follow existing directory structure
2. Include README documentation
3. Add appropriate .gitignore entries
4. Update this PROJECT_STRUCTURE.md

---

**Last Updated**: 2024-01-XX
**Maintained By**: D1L5H4N
