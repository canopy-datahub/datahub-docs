# DataHub AWS Deployment Guide

## 📋 Quick Navigation

- [Prerequisites](#prerequisites)
- [Deployment Checklist](#deployment-checklist)
- [Step-by-Step Deployment](#step-by-step-deployment)
  - [Pre-Deployment Setup](#pre-deployment-setup)
  - [Core AWS Infrastructure](#core-aws-infrastructure)
  - [Post-Deployment Verification](#post-deployment-verification)
- [Troubleshooting](#troubleshooting)
- [Cleanup](#cleanup)

---

## Prerequisites

Before starting, ensure you have:

### Required Tools
- [ ] **AWS Account** with appropriate permissions (AdministratorAccess or specific service permissions)
- [ ] **AWS CLI** installed and configured
- [ ] **Docker** installed and running (for building images and Lambda layers)
- [ ] **Git** installed
- [ ] **Python 3.11+** (for Lambda deployment scripts)
- [ ] **Java 17 JDK** (for building Spring Boot services)
- [ ] **Maven** (for Java builds)
- [ ] **Node.js 18+** (for building Next.js frontend)
- [ ] **PostgreSQL client (psql)** installed

### Installation Commands

<details>
<summary>📦 Click to expand installation instructions for your OS</summary>

#### macOS
```bash
# AWS CLI
brew install awscli

# Docker Desktop
brew install --cask docker

# Other tools
brew install git python@3.11 openjdk@17 maven node postgresql

# Verify installations
aws --version
docker --version
git --version
python3 --version
java --version
mvn --version
node --version
psql --version
```

#### Linux (Ubuntu/Debian)
```bash
# AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Other tools
sudo apt-get update
sudo apt-get install -y git python3.11 openjdk-17-jdk maven nodejs postgresql-client
```

#### Windows
```powershell
# Use Chocolatey (https://chocolatey.org/)
choco install awscli docker-desktop git python311 openjdk17 maven nodejs postgresql

# Or download installers from official websites
```

</details>

---

## Deployment Checklist

Use this checklist to track your progress. Each step links to detailed instructions below.

### Pre-Deployment Setup (15 min)
- [ ] **Step 0:** [Clone repositories](#step-0-clone-repositories) (5 min)
- [ ] **Step 1:** [Configure AWS CLI](#step-1-configure-aws-cli) (5 min)
- [ ] **Step 2:** [Set environment variables](#step-2-set-environment-variables) (5 min)

### Core AWS Infrastructure (~3 hours)
- [ ] **Step 3:** [Deploy Networking Stack](#step-3-deploy-networking-stack) (5 min)
- [ ] **Step 4:** [Deploy S3 Stack](#step-4-deploy-s3-stack) (5 min)
- [ ] **Step 5:** [Deploy LoadBalancer Stack](#step-5-deploy-loadbalancer-stack) (5 min)
- [ ] **Step 6:** [Deploy RDS Stack](#step-6-deploy-rds-stack) (15 min)
  - [ ] **Step 6a:** [Configure RDS Security Group](#step-6a-configure-rds-security-group)
  - [ ] **Step 6b:** [Deploy RDS Stack](#step-6b-deploy-rds-stack)
- [ ] **Step 7:** [Setup Database Schema](#step-7-setup-database-schema) (30 min)
- [ ] **Step 8:** [Deploy Route53 Stack](#step-8-deploy-route53-stack) (Optional) (5 min)
- [ ] **Step 9:** [Deploy CloudWatch Stack](#step-9-deploy-cloudwatch-stack) (5 min)
- [ ] **Step 10:** [Deploy SQS Stack](#step-10-deploy-sqs-stack) (5 min)
- [ ] **Step 11:** [Deploy ECR Stack](#step-11-deploy-ecr-stack) (5 min)
- [ ] **Step 12:** [Deploy OpenSearch Stack](#step-12-deploy-opensearch-stack) (20 min)
  - [ ] **Step 12a:** [Configure OpenSearch Credentials](#step-12a-configure-opensearch-credentials)
  - [ ] **Step 12b:** [Deploy OpenSearch Stack](#step-12b-deploy-opensearch-stack)
- [ ] **Step 13:** [Deploy SecretsManager Stack](#step-13-deploy-secretsmanager-stack) (5 min)
- [ ] **Step 14:** [OpenSearch Reindex Lambda](#step-14-opensearch-reindex-lambda) 
  - [ ] **Step 14a:** [Create Lambda Layer](#step-14a-create-lambda-layer) (5 min)
  - [ ] **Step 14b:** [Upload OpenSearch Reindex Lambda Code](#step-14b-upload-opensearch-reindex-lambda-code) (5 min)
- [ ] **Step 15:** [Upload Email Service Lambda Code](#step-15-upload-email-service-lambda-code) (5 min)
- [ ] **Step 16:** [Deploy Lambda Stack](#step-16-deploy-lambda-stack) (10 min)
- [ ] **Step 17:** [Deploy ECS Stack](#step-17-deploy-ecs-stack) (20 min)
- [ ] **Step 18:** [Deploy SES Stack](#step-18-deploy-ses-stack) (20 min)
  - [ ] **Step 18a:** [Deploy SES Stack](#step-18a-deploy-ses-stack) (10 min)
  - [ ] **Step 18b:** [Configure SES Email Verification](#step-18b-configure-ses-email-verification)  (10 min)
- [ ] **Step 19:** [Deploy EventBridge Stack](#step-19-deploy-eventbridge-stack) (5 min)
- [ ] **Step 20:** [Build and Deploy Services](#step-20-build-and-deploy-services)  (50 min)

### Post-Deployment (10 min)
- [ ] **Step 21:** [Verify deployment](#step-21-verify-deployment)
- [ ] **Step 22:** [Test application](#step-22-test-application)

---

## Step-by-Step Deployment

## Pre-Deployment Setup

### Step 0: Clone Repositories
**Time:** 5 minutes

Clone all necessary repositories from the BMIR DataHub GitHub organization:

```bash
# Create workspace directory
mkdir -p ~/dataHub
cd ~/dataHub

# Clone all repositories
git clone https://github.com/bmir-datahub/datahub-cloud-replication.git
git clone https://github.com/bmir-datahub/datahub-development.git
git clone https://github.com/bmir-datahub/datahub-service-entity.git
git clone https://github.com/bmir-datahub/datahub-service-search.git
git clone https://github.com/bmir-datahub/datahub-service-user.git
git clone https://github.com/bmir-datahub/datahub-service-submission.git
git clone https://github.com/bmir-datahub/datahub-service-report.git
git clone https://github.com/bmir-datahub/datahub-service-download.git
git clone https://github.com/bmir-datahub/datahub-service-email.git
git clone https://github.com/bmir-datahub/datahub-lib-keycloak-auth.git
git clone https://github.com/bmir-datahub/datahub-ui-main.git

# Verify all repositories are cloned
ls -la
```

✅ **Verify:** You should see 11 directories in `~/dataHub/`

---

### Step 1: Configure AWS CLI
**Time:** 5 minutes

Set up AWS credentials for deployment:

**Get AWS credentials:**
1. Log into AWS Console
2. Go to IAM → Users → Select your user
3. Security Credentials tab → Create Access Key
4. Download and save the credentials securely
5. Replace `YOUR_ACCESS_KEY_ID` and `YOUR_SECRET_ACCESS_KEY` below

```bash
# Create AWS credentials file
mkdir -p ~/.aws

# Add your credentials
cat <<EOL >> ~/.aws/credentials
[datahub-rep]
aws_access_key_id=YOUR_ACCESS_KEY_ID
aws_secret_access_key=YOUR_SECRET_ACCESS_KEY
EOL

# Set default region
cat <<EOL >> ~/.aws/config
[profile datahub-rep]
region=us-east-1
output=json
EOL
```



✅ **Verify:** Test AWS connectivity
```bash
aws sts get-caller-identity --profile datahub-rep
```

**Expected output:** Should show your AWS account ID, user ARN, and user ID.

---

### Step 2: Set Environment Variables
**Time:** 5 minutes

Set environment variables that will be used throughout the deployment:

```bash
# Set environment (dev, test, or prod)
export ENV=dev

# Set AWS profile and region
export AWS_PROFILE=datahub-rep
export AWS_DEFAULT_REGION=us-east-1

# Navigate to CloudFormation repository
cd ~/dataHub/datahub-cloud-replication

# Verify settings
echo "Environment: $ENV"
echo "AWS Profile: $AWS_PROFILE"
echo "AWS Region: $AWS_DEFAULT_REGION"
```
##### 💡 **Tip:** Add these to your `~/.bashrc` or `~/.zshrc` to persist across sessions.
---

## Core AWS Infrastructure

### Step 3: Deploy Networking Stack
**Time:** 5 minutes

Creates VPC, subnets, security groups, and networking infrastructure.

```bash
aws cloudformation deploy \
  --stack-name DataHub-Networking-${ENV} \
  --template-file modules/Networking.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --tags projectname=datahub environment=${ENV}
```

✅ **Verify:** 
```bash
aws cloudformation describe-stacks \
  --stack-name DataHub-Networking-${ENV} \
  --query 'Stacks[0].StackStatus' \
  --output text
```
**Expected:** `CREATE_COMPLETE`

---

### Step 4: Deploy S3 Stack
**Time:** 5 minutes

Creates S3 buckets for data files, metadata files, data dictionary files and lambda functions code.

#### 1. Configure DataHubUniqueId for S3 Buckets 

S3 bucket names must be globally unique. Update the `DataHubUniqueId` parameter:

```bash
cd ~/dataHub/datahub-cloud-replication

# Open parameters file for your environment
nano parameters-${ENV}.json  

# edit your `DataHubUniqueId` parameter

# save it in the environment variable
export DataHubUniqueId=$(cat parameters-${ENV}.json | grep -o '"DataHubUniqueId": "[^"]*"' | cut -d'"' -f4)
echo "DataHubUniqueId: $DataHubUniqueId"
```

**Update the `DataHubUniqueId` value:**
- Use lowercase letters, numbers, and hyphens only
- Keep it short (5-20 characters)
- Make it unique (e.g., your institution name + random string)
- Example: `stanford-abc123`, `myorg-dev`, `acme-prod`

**Bucket names that will be created:**
- `datahub-upload-portal-{DataHubUniqueId}-{ENV}`
- `sftp-datahub-{DataHubUniqueId}-{ENV}`
- `datahub-review-{DataHubUniqueId}-{ENV}`
- `datahub-default-{DataHubUniqueId}-{ENV}`
- `datahub-lambda-artifacts-{DataHubUniqueId}-{ENV}`

#### 2. Deploy into AWS
```bash
aws cloudformation deploy \
  --stack-name DataHub-S3-${ENV} \
  --template-file modules/S3.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --tags projectname=datahub environment=${ENV}
```

⚠️ **Troubleshooting:** If deployment fails with "BucketAlreadyExists", go back to Step 4.1 and choose a different `DataHubUniqueId`.

✅ **Verify:** 
```bash
aws s3 ls | grep datahub-${DataHubUniqueId}-${ENV}
```
**Expected:** Should list 5 buckets

---

### Step 5: Deploy LoadBalancer Stack
**Time:** 5 minutes

Creates Application Load Balancer, target groups, and routing rules.

```bash
aws cloudformation deploy \
  --stack-name DataHub-LoadBalancer-${ENV} \
  --template-file modules/LoadBalancer.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --tags projectname=datahub environment=${ENV}
```

✅ **Verify:** Get ALB DNS name
```bash
aws elbv2 describe-load-balancers \
  --query 'LoadBalancers[?contains(LoadBalancerName, `datahub`)].DNSName' \
  --output text
```
**Expected:** DNS name like `datahub-alb-dev-123456789.us-east-1.elb.amazonaws.com`

---

### Step 6: Deploy RDS Stack
**Time:** 15 minutes

Creates RDS PostgreSQL database instance.

#### 6a. Configure RDS Security Group

Before deploying, add your local IP to allow database connection for schema setup:

```bash
# Get your public IP
curl -4 ifconfig.me
```

Edit [RDS.yaml](https://github.com/bmir-datahub/datahub-cloud-replication/blob/feature/aws/modules/RDS.yaml) at line 153:
```yaml
- CidrIp: "YOUR_PUBLIC_IP/32"  # Replace with your IP from above
  Description: "Your workstation"
```

#### 6b. Deploy RDS Stack

```bash
aws cloudformation deploy \
  --stack-name DataHub-RDS-${ENV} \
  --template-file modules/RDS.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --tags projectname=datahub environment=${ENV}
```

⏳ **Wait time:** 10-15 minutes for RDS instance to be created

✅ **Verify:** Get RDS endpoint
```bash
aws rds describe-db-instances \
  --query 'DBInstances[?DBInstanceIdentifier==`datahub-postgres-'${ENV}'`].Endpoint.Address' \
  --output text
```
#### **Expected:** Endpoint like `datahub-postgres-dev.abc123.us-east-1.rds.amazonaws.com`
---

### Step 7: Setup Database Schema
**Time:** 30 minutes

This step creates the database schema, tables, views, and initial data in your RDS PostgreSQL instance.

#### Quick Setup (Automated)

Use the automated Python deployment script:

```bash
cd ~/dataHub/datahub-development/db/postgres/db-create-scripts

# Deploy database schema to dev environment
python deploy_to_rds.py --env dev --region us-east-1 --profile datahub-rep
```

**What the script does:**
1. ✅ Verifies AWS credentials and RDS connectivity
2. ✅ Gets RDS endpoint automatically
3. ✅ Tests database connection
4. ✅ Runs all SQL scripts in order:
   - `01_create_user_roles.sql` - Creates database users and roles
   - `02_create_base_db.sql` - Creates tables, views, functions
   - `03_populate_base_tables.sql` - Populates lookup tables
   - `04_populate_variable_tables.sql` - Populates variable data
   - `05_populate_test_data.sql` - Adds test data (dev/test only)
5. ✅ Provides next steps for Secrets Manager update

⏳ **Wait time:** ~5-10 minutes for all scripts to complete

✅ **Verify:** The script will confirm successful deployment and show table counts.

#### Post-Deployment: Update Secrets Manager

After database deployment, update the [secret](https://github.com/bmir-datahub/datahub-cloud-replication/blob/feature/aws/modules/SecretsManager.yaml) with RDS credentials.

⚠️ **Note:** Step 6 will display the RDS endpoint. Use that endpoint to update `host` value in Secrets Manager (see detailed guide below).


#### 📘 Detailed Documentation

**See:** [README_RDS_DEPLOYMENT.md](https://github.com/bmir-datahub/datahub-development/blob/feature/aws/db/postgres/db-create-scripts/README_RDS_DEPLOYMENT.md)

---

### Step 8: Deploy Route53 Stack (Optional)
**Time:** 5 minutes

Creates DNS records and routing configurations.

⚠️ **Note:** This stack is organization-dependent. Consult with your DNS administrator before deploying.

```bash
cd ~/dataHub/datahub-cloud-replication

aws cloudformation deploy \
  --stack-name DataHub-Route53-${ENV} \
  --template-file modules/Route53.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --tags projectname=datahub environment=${ENV}
```

💡 **Skip this step** if you don't have Route53 hosted zones configured.

---

### Step 9: Deploy CloudWatch Stack
**Time:** 5 minutes

Creates monitoring, logging, and alerting infrastructure.

```bash
cd ~/dataHub/datahub-cloud-replication 

aws cloudformation deploy \
  --stack-name DataHub-CloudWatch-${ENV} \
  --template-file modules/CloudWatch.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --tags projectname=datahub environment=${ENV}
```

✅ **Verify:** List log groups
```bash
aws logs describe-log-groups --query 'logGroups[?contains(logGroupName, `datahub`)].logGroupName'
```

---

### Step 10: Deploy SQS Stack
**Time:** 5 minutes

Creates SQS queues for message processing.

```bash
aws cloudformation deploy \
  --stack-name DataHub-SQS-${ENV} \
  --template-file modules/SQS.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --tags projectname=datahub environment=${ENV}
```

---

### Step 11: Deploy ECR Stack
**Time:** 5 minutes

Creates ECR repositories for container images.

```bash
aws cloudformation deploy \
  --stack-name DataHub-ECR-${ENV} \
  --template-file modules/ECR.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --tags projectname=datahub environment=${ENV}
```

**What's created:**
- ECR repositories for each microservice:
  - `datahub-user-service/{ENV}`
  - `datahub-submission-service/{ENV}`
  - `datahub-report-service/{ENV}`
  - `datahub-download-service/{ENV}`
  - `datahub-entity-service/{ENV}`
  - `datahub-search-service/{ENV}`
  - `datahub-ui/{ENV}`

✅ **Verify:** List ECR repositories
```bash
aws ecr describe-repositories --query 'repositories[?contains(repositoryName, `datahub`)].repositoryName'
```

---

### Step 12: Deploy OpenSearch Stack
**Time:** 20 minutes

Creates OpenSearch domain for search and analytics.

#### Step 12a. Configure OpenSearch Credentials

⚠️ **Security:** Change the default OpenSearch password before deployment!

Edit `parameters-${ENV}.json` and update:
```json
{
  "OpenSearchUsername": "opensearch",
  "OpenSearchPassword": "YourSecurePassword@2026"
}
```

**Password requirements:**
- Minimum 8 characters
- Must contain uppercase, lowercase, number, and special character

#### Step 12b. Deploy OpenSearch Stack

⚠️ **First-time AWS account:** Uncomment lines 45-49 in [OpenSearch.yaml](https://github.com/bmir-datahub/datahub-cloud-replication/blob/feature/aws/modules/OpenSearch.yaml) to create the service-linked role.

```bash
aws cloudformation deploy \
  --stack-name DataHub-OpenSearch-${ENV} \
  --template-file modules/OpenSearch.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --tags projectname=datahub environment=${ENV}
```

⏳ **Wait time:** 15-20 minutes for OpenSearch domain to be created

✅ **Verify:** Get OpenSearch endpoint
```bash
aws opensearch describe-domain \
  --domain-name datahub-opensearch-${ENV} \
  --query 'DomainStatus.Endpoint' \
  --output text
```
**Expected:** Endpoint like `vpc-datahub-opensearch-dev-abc123.us-east-1.es.amazonaws.com`

---

### Step 13: Deploy SecretsManager Stack
**Time:** 5 minutes

Creates secrets for database credentials, API keys, and OpenSearch configuration.

⚠️ **Important:** This stack must be deployed AFTER the OpenSearch stack because it imports the OpenSearch endpoint.

```bash
aws cloudformation deploy \
  --stack-name DataHub-SecretsManager-${ENV} \
  --template-file modules/SecretsManager.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --tags projectname=datahub environment=${ENV}
```

✅ **Verify:** Check secret was created
```bash
aws secretsmanager describe-secret --secret-id application_${ENV}
```

---

### Step 14: OpenSearch Reindex Lambda 
📘 For detailed OpenSearch reindex lambda deployment documentation, see: [README_OPENSEARCH_REINDEX_LAMBDA_DEPLOYMENT.md](https://github.com/bmir-datahub/datahub-development/blob/feature/aws/opensearch/opensearch_reindex/README_OPENSEARCH_REINDEX_LAMBDA_DEPLOYMENT.md)

#### Step 14a: Create Lambda Layer
**Time:** 5 minutes

**⚠️ MUST BE DONE BEFORE Lambda Stack Deployment**

Lambda layers separate dependencies from code, enabling faster deployments.

#### Why Lambda Layers?

<details>
<summary>📘 Click to learn why layers are essential</summary>

**The Problem:**
- Lambda deployment package limit: 50 MB (zipped), 250 MB (unzipped)
- Our dependencies are ~80 MB:
  - `psycopg2-binary` (~25 MB) - Compiled PostgreSQL C libraries
  - `opensearch-py` (~15 MB)
  - Other dependencies (~10 MB)
- Uploading 80 MB every code change = 15+ minute deployments

**The Solution:**
- **Layers**: Dependencies (80 MB) - created once
- **Code package**: Your code (100 KB) - updated frequently
- **Result**: Deployments are 800x faster! (~5 seconds vs ~15 minutes)

**Additional Benefits:**
1. Native binary compatibility (ARM64 architecture)
2. Reusability across multiple Lambda functions
3. Separate version management for code vs dependencies
4. Cost efficiency (faster cold starts)

</details>

#### Create the Layer

```bash
cd ~/dataHub/datahub-development/opensearch/opensearch_reindex

# Create Lambda layer with ARM64-compatible dependencies
python create_layer.py dependency-layer us-east-1 datahub-rep
```

**What this does:**
1. Uses Docker to build ARM64-compatible Python dependencies
2. Packages dependencies: `psycopg2-binary`, `opensearch-py`, `requests-aws4auth`, `boto3`, `requests`
3. Publishes layer to AWS Lambda
4. Outputs Layer ARN for CloudFormation

⏳ **Wait time:** 3-5 minutes (Docker build + upload)

✅ **Verify:** Note the Layer ARN from the output
```
Layer ARN:
  arn:aws:lambda:us-east-1:123456789012:layer:dependency-layer:1
```

**⚠️ Important:** Ensure this ARN matches what's in `datahub-cloud-replication/modules/Lambda.yaml` at line 272. If different, update [Lambda.yaml](https://github.com/bmir-datahub/datahub-cloud-replication/blob/feature/aws/modules/Lambda.yaml) with the new ARN.

---

#### Step 14b: Upload OpenSearch Reindex Lambda Code
**Time:** 5 minutes

Package and upload Lambda function code to S3.

```bash
# Still in opensearch_reindex directory
python deploy_lambda.py ${ENV} ${DataHubUniqueId}
```

**What this does:**
1. Validates required files exist
2. Packages Lambda code with mapping files:
   - `opensearch_reindex_aws.py`
   - `search_index_mapping.json`
   - `variable_index_mapping.json`
   - `autocomplete_index_mapping.json`
3. Creates ZIP (should be <100KB)
4. Uploads to S3: `s3://datahub-lambda-artifacts-{DataHubUniqueId}-{ENV}/opensearch-refresh/`

⚠️ **Note:** Dependencies are NOT included (they're in the layer from Step 14a)

✅ **Verify:** Check S3 upload
```bash
aws s3 ls s3://datahub-lambda-artifacts-${DataHubUniqueId}-${ENV}/opensearch-refresh/
```
**Expected:** Should show `opensearch-refresh-lambda.zip`

### Step 15: Upload Email Service Lambda Code

```bash
cd ~/dataHub/datahub-service-email

# Build Spring Boot application for Lambda
mvn clean package -DskipTests

# Upload to S3
aws s3 cp target/datahub-service-email-0.0.1-SNAPSHOT-aws.jar \
  s3://datahub-lambda-artifacts-${DataHubUniqueId}-${ENV}/email-service/
```

---

### Step 16: Deploy Lambda Stack
**Time:** 10 minutes

Creates Lambda functions for OpenSearch indexing and email service.

```bash
cd ~/dataHub/datahub-cloud-replication

aws cloudformation deploy \
  --stack-name DataHub-Lambda-${ENV} \
  --template-file modules/Lambda.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --tags projectname=datahub environment=${ENV}
```

**What's created:**
- Lambda function: `DataHub-OpenSearchRefresh`
  - Runtime: Python 3.11 on ARM64
  - VPC: Deployed in private subnets
  - Attached layer from Step 14a
  - Code from S3 (Step 14b)
- Lambda function: `DataHub-EmailService`
  - Runtime: Java 17
  - Code from S3
- IAM roles and permissions
- VPC configuration
- Security group access

✅ **Verify:** Check Lambda functions exist
```bash
aws lambda list-functions --query 'Functions[?contains(FunctionName, `DataHub`)].FunctionName'
```

#### Test OpenSearch Lambda (Optional)

```bash
aws lambda invoke \
  --function-name DataHub-OpenSearchRefresh \
  --payload '{}' \
  response.json

cat response.json
```

---

### Step 17: Deploy ECS Stack
**Time:** 20 minutes

Creates ECS cluster, services, and container definitions for all microservices.

```bash
aws cloudformation deploy \
  --stack-name DataHub-ECS-${ENV} \
  --template-file modules/ECS.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --tags projectname=datahub environment=${ENV}
```

⏳ **Wait time:** 15-20 minutes for all services to start

**What's created:**
- ECS services (1 per microservice):
  - User Service
  - Submission Service
  - Report Service
  - Download Service
  - Entity Service
  - Search Service
  - Frontend

✅ **Verify:** Check ECS services are running
```bash
aws ecs list-services --cluster datahub-Services-${ENV}

# Check running tasks
aws ecs describe-services \
  --cluster datahub-Services-${ENV} \
  --services $(aws ecs list-services --cluster datahub-Services-${ENV} --query 'serviceArns' --output text) \
  --query 'services[*].{Name:serviceName,Running:runningCount,Desired:desiredCount}'
```

**Expected:** All services should show `Running: 1, Desired: 1`

---

### Step 18: Deploy SES Stack
**Time:** 10 minutes

#### Step 18a: Deploy SES Stack

Creates SES email identities for sending emails.

##### 1. Configure Email Identities (Optional)

By default, the SES stack creates three email identities:
- `datahub@stanford.edu` - Primary sender email
- `datahub.dev@stanford.edu` - Development/testing email
- `stanford.edu` - Domain identity (allows sending from any @stanford.edu address)

**To use your own email addresses:**

Edit [`modules/SES.yaml`](https://github.com/bmir-datahub/datahub-cloud-replication/blob/feature/aws/modules/SES.yaml) and replace the default email identities with your organization's emails.

💡 **Tip:** Using a domain identity (e.g., `yourdomain.com`) allows you to send emails from any address at that domain without verifying each individual email.

##### 2. Deploy SES Stack

```bash
aws cloudformation deploy \
  --stack-name DataHub-SES-${ENV} \
  --template-file modules/SES.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --tags projectname=datahub environment=${ENV}
```

⚠️ **Important:** Email identities must be verified before emails can be sent. Complete Step 18b below.

---

#### Step 18b: Configure SES Email Verification
**Time:** 10 minutes

Verify email identities for SES (required for sending emails).

##### For Development/Test Environments (Sandbox Mode)

In sandbox mode, both sender AND recipient emails must be verified:

**Verify via AWS Console:**
1. Go to AWS Console → SES → Verified identities (in `${AWS_DEFAULT_REGION}`)
2. Click on each email identity
3. Click "Verify" button
4. Check email inbox for verification link
5. Click the verification link

⚠️ **Note:** In sandbox mode, emails can only be sent to verified addresses.

##### For Production Environment

Request to move out of sandbox mode:

1. Go to AWS Console → SES → Account dashboard
2. Click "Request production access"
3. Fill out the form with your use case
4. Wait for AWS approval (usually 24-48 hours)

✅ **Verify:** Check verification status
```bash
aws ses get-identity-verification-attributes \
  --identities datahub@stanford.edu \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --query 'VerificationAttributes.*.VerificationStatus' \
  --output text
```
**Expected:** `Success`

---

### Step 19: Deploy EventBridge Stack
**Time:** 5 minutes

Creates EventBridge rules to automatically trigger SQS processing when files are uploaded to S3 via SFTP.

```bash
aws cloudformation deploy \
  --stack-name DataHub-EventBridge-${ENV} \
  --template-file modules/EventBridge.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ${AWS_DEFAULT_REGION} \
  --profile ${AWS_PROFILE} \
  --tags projectname=datahub environment=${ENV}
```

**What's created:**
- EventBridge rule SFTP-{Environment} that monitors S3 for SFTP file uploads
- Automatic routing of S3 events to SQS SFTP Queue

---

### Step 20: Build and Deploy Services
**Time:** 50 minutes (for all 7 services)

**Now that all AWS infrastructure is deployed, build and deploy your application services to ECS.**

#### Prerequisites Check

Before proceeding, ensure:
- ✅ All CloudFormation stacks are complete
- ✅ ECS cluster is running (Step 17)
- ✅ ECR repositories exist (Step 11)
- ✅ Docker is installed and running
- ✅ Maven is installed (for backend services)
- ✅ Python 3.7+ with `boto3` installed

#### Install Python Dependencies

```bash
# Install boto3 for deploy.py script
pip install boto3

# Or use requirements file
pip install -r requirements-deploy.txt
```

#### Deploy All Services

The `deploy.py` script automates the entire build and deployment process for each service:

```bash
cd ~/dataHub

# Deploy each service (one at a time)
python deploy.py user-service ${ENV}
python deploy.py submission-service ${ENV}
python deploy.py report-service ${ENV}
python deploy.py download-service ${ENV}
python deploy.py entity-service ${ENV}
python deploy.py search-service ${ENV}
python deploy.py ui ${ENV}
```

**What the script does for EACH service:**
1. ✅ Verifies source directory and Dockerfile exist
2. ✅ Builds the application:
   - **Backend services**: Runs `mvn clean package -DskipTests`
   - **Frontend (UI)**: Build handled by Docker multi-stage
3. ✅ Authenticates with AWS ECR
4. ✅ Builds Docker image for `linux/amd64` platform
5. ✅ Pushes image to ECR repository
6. ✅ Forces ECS service redeployment with new image
7. ✅ Verifies deployment was triggered

⏳ **Wait time:** ~5-7 minutes per service (~50 minutes total)

#### Progress Tracking

You can check deployment progress in AWS Console:
- **ECS Console** → Clusters → `datahub-cluster-{ENV}` → Services
- Watch for "Running count" to reach "Desired count" (1/1)

✅ **Verify:** Check all services are deployed
```bash
aws ecs describe-services \
  --cluster datahub-cluster-${ENV} \
  --services $(aws ecs list-services --cluster datahub-Services-${ENV} --query 'serviceArns' --output text) \
  --query 'services[*].{Name:serviceName,Running:runningCount,Desired:desiredCount,Status:status}' \
  --output table
```

**Expected output:** All services should show `Running: 1, Desired: 1, Status: ACTIVE`


## Post-Deployment Verification

### Step 21: Verify Deployment
**Time:** 5 minutes

Check that all stacks deployed successfully:

```bash
# List all DataHub stacks
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE \
  --query 'StackSummaries[?contains(StackName, `DataHub`)].{Name:StackName,Status:StackStatus,Created:CreationTime}' \
  --output table
```

**Expected:** All stacks should show `CREATE_COMPLETE` or `UPDATE_COMPLETE`

**Stacks you should see (14 total):**
1. DataHub-Networking-{ENV}
2. DataHub-S3-{ENV}
3. DataHub-LoadBalancer-{ENV}
4. DataHub-RDS-{ENV}
5. DataHub-Route53-{ENV} (optional)
6. DataHub-CloudWatch-{ENV}
7. DataHub-SQS-{ENV}
8. DataHub-ECR-{ENV}
9. DataHub-OpenSearch-{ENV}
10. DataHub-SecretsManager-{ENV}
11. DataHub-ECS-{ENV}
12. DataHub-Lambda-{ENV}
13. DataHub-SES-{ENV}
14. DataHub-EventBridge-{ENV}

**Services you should see running (7 total):**
```bash
# Check ECS services
aws ecs list-services --cluster datahub-Services-${ENV} --query 'serviceArns' --output table
```

Expected services:
1. datahub-UserService
2. datahub-SubmissionService
3. datahub-ReportService
4. datahub-DownloadService
5. datahub-EntityService
6. datahub-SearchService
7. datahub-Frontend (UI)

---

### Step 22: Test Application

Test that the application is accessible and functioning:

```bash
# Get ALB DNS name
ALB_DNS=$(aws elbv2 describe-load-balancers \
  --query 'LoadBalancers[?contains(LoadBalancerName, `datahub`)].DNSName' \
  --output text)

echo "Application URL: http://$ALB_DNS"

# Test frontend
curl -I http://$ALB_DNS

# Test API endpoints
curl http://$ALB_DNS/api/entity/v1/health
curl http://$ALB_DNS/api/search/v1/health
curl http://$ALB_DNS/api/user/v1/health
curl http://$ALB_DNS/api/submission/v1/health
```

**Expected responses:**
- Frontend: `HTTP/1.1 200 OK`
- APIs: `{"status":"UP"}` or similar health check response

#### Access the Application

Open your browser and navigate to: `http://{ALB_DNS}`

✅ **Success!** You should see the DataHub login page.

---

## Troubleshooting

### Common Issues

#### Stack Creation Failed

```bash
# Check stack events for errors
aws cloudformation describe-stack-events \
  --stack-name DataHub-{ENV}-{STACK-NAME} \
  --query 'StackEvents[?ResourceStatus==`CREATE_FAILED`]'
```

#### S3 Bucket Already Exists

**Error:** `BucketAlreadyExists` or `BucketAlreadyOwnedByYou`

**Solution:** Go back to Step 4.1 and choose a different `DataHubUniqueId`.

#### ECS Service Fails to Start

**Symptoms:** ECS service shows 0 running tasks

**Check:**
1. Docker images are pushed to ECR
2. Task logs in CloudWatch Logs
3. Security groups allow traffic
4. Secrets Manager secret exists and is accessible

```bash
# Check task logs
aws logs tail /ecs/datahub-user-service --follow
```

#### Lambda Function Fails

**Symptoms:** Lambda returns errors or times out

**Check:**
1. Lambda code is uploaded to S3
2. Lambda has VPC access
3. Security groups allow connections to RDS/OpenSearch
4. Layer ARN is correct

```bash
# Check Lambda logs
aws logs tail /aws/lambda/DataHub-OpenSearchRefresh-${ENV} --follow
```

#### Database Connection Issues

**Symptoms:** ECS services can't connect to RDS

**Check:**
1. RDS endpoint is correct in Secrets Manager
2. Security group allows connections from ECS
3. Database credentials are correct
4. RDS instance is available

```bash
# Test connection from your machine
psql -h $RDS_ENDPOINT -U datahub_user -d datahub_${ENV}
```

#### OpenSearch Connection Issues

**Symptoms:** Lambda or services can't connect to OpenSearch

**Check:**
1. OpenSearch endpoint is correct in Secrets Manager
2. Security group allows connections
3. OpenSearch credentials are correct
4. OpenSearch domain is active (not red)

```bash
# Check OpenSearch domain status
aws opensearch describe-domain --domain-name datahub-opensearch-${ENV} \
  --query 'DomainStatus.{Status:Processing,Health:DomainEndpointOptions.EnforceHTTPS}'
```

### Getting Help

1. **Check CloudWatch Logs** - Most errors are logged
2. **Review CloudFormation Events** - Shows why stacks failed
3. **AWS Console** - Visual inspection of resources
4. **Documentation** - Check service-specific READMEs:
   - [RDS Deployment](https://github.com/bmir-datahub/datahub-development/blob/feature/aws/db/postgres/db-create-scripts/README_RDS_DEPLOYMENT.md)
   - [Lambda Deployment](https://github.com/bmir-datahub/datahub-development/blob/feature/aws/opensearch/opensearch_reindex/README_OPENSEARCH_REINDEX_LAMBDA_DEPLOYMENT.md)

---

## Cleanup

**⚠️ WARNING:** This will delete ALL resources and data. Make sure you have backups!

### Step 1: Empty S3 Buckets

S3 buckets with content cannot be deleted by CloudFormation:

```bash
# List all DataHub S3 buckets
aws s3 ls | grep datahub-${DataHubUniqueId}-${ENV}

# Empty each bucket
aws s3 ls | grep datahub-${DataHubUniqueId}-${ENV} | awk '{print $3}' | while read bucket; do
  echo "Emptying bucket: $bucket"
  aws s3 rm s3://$bucket --recursive
done
```

### Step 2: Delete Stacks in Reverse Order

```bash
# Delete stacks in reverse dependency order
stacks=(
  "DataHub-EventBridge-${ENV}"
  "DataHub-Lambda-${ENV}"
  "DataHub-SES-${ENV}"
  "DataHub-ECS-${ENV}"
  "DataHub-SecretsManager-${ENV}"
  "DataHub-OpenSearch-${ENV}"
  "DataHub-ECR-${ENV}"
  "DataHub-SQS-${ENV}"
  "DataHub-CloudWatch-${ENV}"
  "DataHub-Route53-${ENV}"
  "DataHub-RDS-${ENV}"
  "DataHub-LoadBalancer-${ENV}"
  "DataHub-S3-${ENV}"
  "DataHub-Networking-${ENV}"
)

for stack in "${stacks[@]}"; do
  echo "Deleting stack: $stack"
  aws cloudformation delete-stack --stack-name $stack
  
  echo "Waiting for $stack deletion to complete..."
  aws cloudformation wait stack-delete-complete --stack-name $stack
  
  echo "✅ $stack deleted successfully"
done

echo "🎉 Cleanup complete!"
```

### Step 3: Verify Cleanup

```bash
# Check for remaining stacks
aws cloudformation list-stacks \
  --query 'StackSummaries[?contains(StackName, `DataHub`) && StackStatus != `DELETE_COMPLETE`].{Name:StackName,Status:StackStatus}' \
  --output table

# Check for remaining S3 buckets
aws s3 ls | grep datahub

# Check for remaining ECR repositories
aws ecr describe-repositories --query 'repositories[?contains(repositoryName, `datahub`)].repositoryName'
```

---

## Summary

🎉 **Congratulations!** You've successfully deployed DataHub to AWS!

**What you've deployed:**
- ✅ Complete VPC network infrastructure
- ✅ PostgreSQL RDS database with full schema
- ✅ OpenSearch domain for search functionality
- ✅ 7 microservices running on ECS Fargate
- ✅ Application Load Balancer with routing
- ✅ Lambda functions for sending emails and reindexing OpenSearch
- ✅ S3 buckets for file storage
- ✅ SES for email notifications
- ✅ CloudWatch for monitoring and logging

**Next steps:**
1. Configure custom domain name (optional)
2. Set up SSL/TLS certificates (recommended for production)

---

**Document Version:** 1.0  
**Last Updated:** January 2026  
**Maintained by:** Stanford DataHub Team

