# Redwood Platform AWS Deployment Guide

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
- [ ] **AWS Account** 
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

### Pre-Deployment Setup (20 min)
- [ ] **Step 0:** [Clone repositories](#step-0-clone-repositories) (5 min)
- [ ] **Step 1:** [Install and Configure AWS CLI](#step-1-configure-aws-cli) (10 min)
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
- [ ] **Step 20:** [Deploy TransferFamily Stack](#step-20-deploy-transferfamily-stack-sftp-server) (10 min)
  - [ ] **Step 20a:** [Create SFTP User Secret](#step-20a-create-sftp-user-secret)
  - [ ] **Step 20b:** [Allocate Elastic IP](#step-20b-optional-allocate-elastic-ip) (Optional)
  - [ ] **Step 20c:** [Deploy TransferFamily Stack](#step-20c-deploy-transferfamily-stack)
- [ ] **Step 21:** [Build and Deploy Services](#step-21-build-and-deploy-services)  (50 min)

### Post-Deployment (10 min)
- [ ] **Step 22:** [Verify deployment](#step-22-verify-deployment)
- [ ] **Step 23:** [Test application](#step-23-test-application)

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
git clone https://github.com/bmir-datahub/datahub-deployment-scripts.git

# Verify all repositories are cloned
ls -la
```

✅ **Verify:** You should see 12 directories in `~/dataHub/`

---

### Step 1: Install and Configure AWS CLI
**Time:** 10 minutes

#### Step 1a: Install AWS CLI
```bash
# For MacOS:
brew install awscli
# Verify installation
aws --version
```

#### Step 1b: Set up AWS credentials for deployment:

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
*IMPORTANT: Unset any existing AWS credentials from environment*
```bash
unset AWS_ACCESS_KEY_ID
unset AWS_SECRET_ACCESS_KEY
unset AWS_SESSION_TOKEN

# Now set the profile and region
export AWS_PROFILE=datahub-rep
export AWS_REGION=us-east-1
```

✅ **Verify:** Test AWS connectivity
```bash
aws sts get-caller-identity
```

**Expected output:** Should show your AWS account ID, user ARN, and user ID.
*Please make sure you are using the correst aws profile*

#### Step 1c: Grant IAM Permissions

The IAM user performing the deployment must have permissions for the following AWS services: CloudFormation, VPC, EC2, S3, Elastic Load Balancing (ALB), RDS, Route53, CloudWatch, SQS, ECR, OpenSearch Service, Secrets Manager, ECS, SES, Lambda, EventBridge, and IAM. Alternatively, for simplified setup, you may assign the IAM user the **AdministratorAccess** managed policy, which provides full access to all AWS services required by the system.

```bash
# Change to your own user name below
aws iam attach-user-policy \
  --user-name datahub-dev \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```
---

### Step 2: Set Environment Variables
**Time:** 5 minutes

Set environment variables that will be used throughout the deployment:

```bash
# Set environment (dev, test, or prod)
export ENV=dev

# Set project name (use a different project name as you want)
# Navigate to CloudFormation repository
cd ~/dataHub/datahub-cloud-replication

# Open parameters file for your environment
nano parameters-${ENV}.json  

# edit your `ProjectName` parameter

# save it in the environment variable
export PROJECT_NAME=$(cat parameters-${ENV}.json | grep -o '"ProjectName": "[^"]*"' | cut -d'"' -f4)

# Set AWS profile and region
export AWS_PROFILE=datahub-rep
export AWS_REGION=us-east-1

# Verify settings
echo "Environment: $ENV"
echo "ProjectName: $PROJECT_NAME"
echo "AWS Profile: $AWS_PROFILE"
echo "AWS Region: $AWS_REGION"
```
##### 💡 **Tip:** Add these to your `~/.bashrc` or `~/.zshrc` to persist across sessions.
---

## Core AWS Infrastructure
### Navigate to CloudFormation repository
```bash
cd ~/dataHub/datahub-cloud-replication
```

### Step 3: Deploy Networking Stack
**Time:** 5 minutes

Creates VPC, subnets, security groups, and networking infrastructure.

```bash
aws cloudformation deploy \
  --stack-name ${PROJECT_NAME}-Networking-${ENV} \
  --template-file modules/Networking.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
```

✅ **Verify:** 
```bash
aws cloudformation describe-stacks \
  --stack-name ${PROJECT_NAME}-Networking-${ENV} \
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
- `${PROJECT_NAME}-upload-portal-{DataHubUniqueId}-{ENV}`
- `sftp-${PROJECT_NAME}-{DataHubUniqueId}-{ENV}`
- `${PROJECT_NAME}-review-{DataHubUniqueId}-{ENV}`
- `${PROJECT_NAME}-default-{DataHubUniqueId}-{ENV}`
- `${PROJECT_NAME}-lambda-artifacts-{DataHubUniqueId}-{ENV}`

#### 2. Deploy into AWS
```bash
aws cloudformation deploy \
  --stack-name ${PROJECT_NAME}-S3-${ENV} \
  --template-file modules/S3.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
```

⚠️ **Troubleshooting:** If deployment fails with "BucketAlreadyExists", go back to Step 4.1 and choose a different `DataHubUniqueId`.

✅ **Verify:** 
```bash
aws s3 ls | grep ${PROJECT_NAME}-${DataHubUniqueId}-${ENV}
```
**Expected:** Should list 5 buckets

---

### Step 5: Deploy LoadBalancer Stack
**Time:** 5 minutes

Creates Application Load Balancer, target groups, and routing rules.

```bash
aws cloudformation deploy \
  --stack-name ${PROJECT_NAME}-LoadBalancer-${ENV} \
  --template-file modules/LoadBalancer.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
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

#### Step 6a. Configure RDS Security Group

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

#### Step 6b. Deploy RDS Stack

```bash
aws cloudformation deploy \
  --stack-name ${PROJECT_NAME}-RDS-${ENV} \
  --template-file modules/RDS.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
```

⏳ **Wait time:** 10-15 minutes for RDS instance to be created

✅ **Verify:** Get RDS endpoint
```bash
aws rds describe-db-instances \
  --query 'DBInstances[?DBInstanceIdentifier==`${PROJECT_NAME}-postgres-'${ENV}'`].Endpoint.Address' \
  --output text
```
#### **Expected:** Endpoint like `datahub-postgres-dev.abc123.us-east-1.rds.amazonaws.com`
---

### Step 7: Setup Database Schema
**Time:** 30 minutes

This step creates the database schema, tables, views, and initial data in your RDS PostgreSQL instance.

#### 📘 Detailed Documentation

**See:** [README_RDS_DEPLOYMENT.md](https://github.com/bmir-datahub/datahub-development/blob/feature/aws/db/postgres/db-create-scripts/README_RDS_DEPLOYMENT.md)


#### Quick Setup (Automated)

Use the automated Python deployment script:

```bash
# Navigate to the python script
cd ~/dataHub/datahub-development/db/postgres/db-create-scripts

python deploy_to_rds.py --project-name <project-name> --env <env> --region <region> --profile <profile>

# Deploy database schema to dev environment
python deploy_to_rds.py --project-name ${PROJECT_NAME} --env dev --region us-east-1 --profile datahub-rep

# Examples for other environments:
# python deploy_to_rds.py --project-name datahub --env test --region us-east-1 --profile datahub-rep
# python deploy_to_rds.py --project-name datahub --env prod --region us-east-1 --profile datahub-rep
```

**Script Parameters:**
- `--project-name`: Project name (e.g., `datahub`) - **REQUIRED**
- `--env`: Environment (`dev`, `test`, or `prod`) - default: `dev`
- `--region`: AWS region - default: `us-east-1`
- `--profile`: AWS profile name - default: `datahub-rep`

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



---

### Step 8: Deploy Route53 Stack (Optional)
**Time:** 5 minutes

Creates DNS records and routing configurations.

⚠️ **Note:** This stack is organization-dependent. Consult with your DNS administrator before deploying.

```bash
cd ~/dataHub/datahub-cloud-replication

aws cloudformation deploy \
  --stack-name ${PROJECT_NAME}-Route53-${ENV} \
  --template-file modules/Route53.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
```

💡 **Skip this step** if you don't have Route53 hosted zones configured.

---

### Step 9: Deploy CloudWatch Stack
**Time:** 5 minutes

Creates monitoring, logging, and alerting infrastructure.

```bash
cd ~/dataHub/datahub-cloud-replication 

aws cloudformation deploy \
  --stack-name ${PROJECT_NAME}-CloudWatch-${ENV} \
  --template-file modules/CloudWatch.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
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
  --stack-name ${PROJECT_NAME}-SQS-${ENV} \
  --template-file modules/SQS.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
```

---

### Step 11: Deploy ECR Stack
**Time:** 5 minutes

Creates ECR repositories for container images.

```bash
aws cloudformation deploy \
  --stack-name ${PROJECT_NAME}-ECR-${ENV} \
  --template-file modules/ECR.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
```

**What's created:**
- ECR repositories for each microservice:
  - `${PROJECT_NAME}-user-service/{ENV}`
  - `${PROJECT_NAME}-submission-service/{ENV}`
  - `${PROJECT_NAME}-report-service/{ENV}`
  - `${PROJECT_NAME}-download-service/{ENV}`
  - `${PROJECT_NAME}-entity-service/{ENV}`
  - `${PROJECT_NAME}-search-service/{ENV}`
  - `${PROJECT_NAME}-ui/{ENV}`

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
  --stack-name ${PROJECT_NAME}-OpenSearch-${ENV} \
  --template-file modules/OpenSearch.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
```

⏳ **Wait time:** 15-20 minutes for OpenSearch domain to be created

✅ **Verify:** Get OpenSearch endpoint
```bash
aws opensearch describe-domain \
  --domain-name ${PROJECT_NAME}-opensearch-${ENV} \
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
  --stack-name ${PROJECT_NAME}-SecretsManager-${ENV} \
  --template-file modules/SecretsManager.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
```

✅ **Verify:** Check secret was created

The application secret is named **`${PROJECT_NAME}_application_${ENV}`** (e.g., `datahub_application_dev`). ECS task definitions set the **`AWS_SECRET_NAME`** environment variable to this value so backend services load the correct secret via `spring.config.import: aws-secretsmanager:${AWS_SECRET_NAME}`. For a duplicate deployment in the same account, use a different `ProjectName` so each app has its own secret.

```bash
aws secretsmanager describe-secret --secret-id ${PROJECT_NAME}_application_${ENV}
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

**Script Usage:**
```bash
python deploy_lambda.py <project-name> <env> <unique-id>
```

**Parameters:**

- **project-name**: Project name (e.g., `datahub`, `Redwood`) - **REQUIRED**
- **env**: `dev`, `test`, or `prod` - **REQUIRED**
- **DataHubUniqueId**: Unique identifier for S3 bucket (e.g., `stanford`) - **REQUIRED**
  - S3 bucket: `${PROJECT_NAME}-lambda-artifacts-{DataHubUniqueId}-{env}`


```bash
# Still in opensearch_reindex directory

# Upload to dev environment
python deploy_lambda.py datahub dev stanford

# Upload to test environment
python deploy_lambda.py datahub test stanford

# Upload to prod environment
python deploy_lambda.py datahub prod stanford
```

**What this does:**
1. Validates required files exist
2. Packages Lambda code with mapping files:
   - `opensearch_reindex_aws.py`
   - `search_index_mapping.json`
   - `variable_index_mapping.json`
   - `autocomplete_index_mapping.json`
3. Creates ZIP (should be <100KB)
4. Uploads to S3: `s3://${PROJECT_NAME}-lambda-artifacts-{DataHubUniqueId}-{ENV}/opensearch-refresh/`

⚠️ **Note:** Dependencies are NOT included (they're in the layer from Step 14a)

✅ **Verify:** Check S3 upload
```bash
aws s3 ls s3://${PROJECT_NAME}-lambda-artifacts-${DataHubUniqueId}-${ENV}/opensearch-refresh/
```
**Expected:** Should show `opensearch-refresh-lambda.zip`

### Step 15: Upload Email Service Lambda Code

```bash
cd ~/dataHub/datahub-service-email

# Build Spring Boot application for Lambda
mvn clean package -DskipTests

# Upload to S3
aws s3 cp target/datahub-service-email-0.0.1-SNAPSHOT-aws.jar \
  s3://${PROJECT_NAME}-lambda-artifacts-${DataHubUniqueId}-${ENV}/email-service/
```

---

### Step 16: Deploy Lambda Stack
**Time:** 10 minutes

Creates Lambda functions for OpenSearch indexing and email service.

```bash
cd ~/dataHub/datahub-cloud-replication

aws cloudformation deploy \
  --stack-name ${PROJECT_NAME}-Lambda-${ENV} \
  --template-file modules/Lambda.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
```

**What's created:**
- Lambda function: `${PROJECT_NAME}-OpenSearchRefresh`
  - Runtime: Python 3.11 on ARM64
  - VPC: Deployed in private subnets
  - Attached layer from Step 14a
  - Code from S3 (Step 14b)
- Lambda function: `${PROJECT_NAME}-EmailService`
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
  --function-name ${PROJECT_NAME}-OpenSearchRefresh \
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
  --stack-name ${PROJECT_NAME}-ECS-${ENV} \
  --template-file modules/ECS.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
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
aws ecs list-services --cluster ${PROJECT_NAME}-Services-${ENV}

# Check running tasks
aws ecs describe-services \
  --cluster ${PROJECT_NAME}-Services-${ENV} \
  --services $(aws ecs list-services --cluster ${PROJECT_NAME}-Services-${ENV} --query 'serviceArns' --output text) \
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
  --stack-name ${PROJECT_NAME}-SES-${ENV} \
  --template-file modules/SES.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
```

⚠️ **Important:** Email identities must be verified before emails can be sent. Complete Step 18b below.

---

#### Step 18b: Configure SES Email Verification
**Time:** 10 minutes

Verify email identities for SES (required for sending emails).

##### For Development/Test Environments (Sandbox Mode)

In sandbox mode, both sender AND recipient emails must be verified:

**Verify via AWS Console:**
1. Go to AWS Console → SES → Verified identities (in `${AWS_REGION}`)
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
  --stack-name ${PROJECT_NAME}-EventBridge-${ENV} \
  --template-file modules/EventBridge.yaml \
  --parameter-overrides file://parameters-${ENV}.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
```

**What's created:**
- EventBridge rule SFTP-{Environment} that monitors S3 for SFTP file uploads
- Automatic routing of S3 events to SQS SFTP Queue

---

### Step 20: Deploy TransferFamily Stack (SFTP Server) (Need check)
**Time:** 10 minutes

Creates an AWS Transfer Family SFTP server with custom Lambda-based authentication for secure file uploads.

#### Prerequisites

Before deploying, you need to:

1. **Create an SFTP User Secret** in Secrets Manager with the required user credentials and permissions
2. **Obtain an Elastic IP** (optional, but recommended for static SFTP endpoint)

#### Step 20a: Create SFTP User Secret

Create a secret in Secrets Manager that contains the SFTP user configuration:

```bash
# Create a file with the user secret JSON
cat > sftp-user-secret.json <<EOF
{
  "Password": "YourSecurePasswordHere",
  "Role": "arn:aws:iam::${AWS_ACCOUNT_ID}:role/${PROJECT_NAME}-CustomSFTPTransferRole",
  "HomeDirectory": "/${PROJECT_NAME}-sftp-${UNIQUE_ID}-${ENV}/${PROJECT_NAME}-user",
  "PublicKeys": "",
  "Policy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Sid\":\"AllowListingOfUserFolder\",\"Action\":[\"s3:ListBucket\"],\"Effect\":\"Allow\",\"Resource\":[\"arn:aws:s3:::\${transfer:HomeBucket}\"],\"Condition\":{\"StringLike\":{\"s3:prefix\":[\"\${transfer:HomeFolder}/*\",\"\${transfer:HomeFolder}\"]}}},{\"Sid\":\"HomeDirObjectAccess\",\"Effect\":\"Allow\",\"Action\":[\"s3:PutObject\",\"s3:GetObject\",\"s3:DeleteObject\",\"s3:DeleteObjectVersion\",\"s3:GetObjectVersion\",\"s3:GetObjectACL\",\"s3:PutObjectACL\"],\"Resource\":\"arn:aws:s3:::\${transfer:HomeDirectory}*\"}]}"
}
EOF

# Create the secret in Secrets Manager
aws secretsmanager create-secret \
  --name "SFTP/${PROJECT_NAME}-user" \
  --description "${PROJECT_NAME} Transfer Family SFTP User" \
  --secret-string file://sftp-user-secret.json \
  --profile ${AWS_PROFILE} \
  --tags Key=projectname,Value=${PROJECT_NAME} Key=environment,Value=${ENV}

# Clean up the temporary file
rm sftp-user-secret.json
```

**Important:** Replace `YourSecurePasswordHere` with a strong password for SFTP authentication.

#### Step 20b: (Optional) Allocate Elastic IP

For a static SFTP endpoint address:

```bash
# Allocate an Elastic IP
aws ec2 allocate-address \
  --domain vpc \
  --profile ${AWS_PROFILE} \
  --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value='${PROJECT_NAME}'-sftp-'${ENV}'},{Key=projectname,Value='${PROJECT_NAME}'},{Key=environment,Value='${ENV}'}]'

# Note the AllocationId from the output - you'll need it for deployment
```

#### Step 20c: Deploy TransferFamily Stack

```bash
aws cloudformation deploy \
  --stack-name ${PROJECT_NAME}-TransferFamily-${ENV} \
  --template-file modules/TransferFamily.yaml \
  --parameter-overrides \
    file://parameters-${ENV}.json \
    ElasticIPAllocationId=<YOUR_ELASTIC_IP_ALLOCATION_ID> \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${PROJECT_NAME} environment=${ENV}
```

**Note:** If you skip the Elastic IP, remove the `ElasticIPAllocationId` parameter or set it to empty string `''`.

**What's created:**
- AWS Transfer Family SFTP server with VPC endpoints
- Lambda function for custom authentication against Secrets Manager
- API Gateway REST API for authentication integration
- IAM roles and policies for SFTP file operations
- CloudWatch log groups for SFTP access logs

✅ **Verify:** Get SFTP server endpoint
```bash
# Get the server ID
SERVER_ID=$(aws transfer list-servers \
  --profile ${AWS_PROFILE} \
  --query 'Servers[?Tags[?Key==`projectname` && Value==`'${PROJECT_NAME}'`]].ServerId' \
  --output text)

# Get the endpoint
echo "SFTP Endpoint: ${SERVER_ID}.server.transfer.${AWS_REGION}.amazonaws.com"
```

**Expected:** Server ID should be returned (format: `s-1234567890abcdef0`)

#### Testing SFTP Access

Test the SFTP connection:

```bash
# Test SFTP connection
sftp ${PROJECT_NAME}-user@${SERVER_ID}.server.transfer.${AWS_REGION}.amazonaws.com
# Enter the password you configured in the secret

# Once connected, verify your home directory
pwd
# Expected: /sftp-${PROJECT_NAME}-${UNIQUE_ID}-${ENV}/${PROJECT_NAME}-user

# Upload a test file
put test.txt

# Exit
exit
```

---

### Step 21: Build and Deploy Services
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
cd ~/dataHub/datahub-deployment-scripts

# Usage: python deploy.py <project-name> <service-name> <environment> [image-tag]
# project-name,  service-name and environment are required, while image-tag is optioanl
# image-tag with 'lastest' will be used, so the default value of image-tag is 'latest'.

# Deploy each service (one at a time)
python deploy.py ${PROJECT_NAME} user-service ${ENV}
python deploy.py ${PROJECT_NAME} submission-service ${ENV}
python deploy.py ${PROJECT_NAME} report-service ${ENV}
python deploy.py ${PROJECT_NAME} download-service ${ENV}
python deploy.py ${PROJECT_NAME} entity-service ${ENV}
python deploy.py ${PROJECT_NAME} search-service ${ENV}
python deploy.py ${PROJECT_NAME} ui ${ENV}

# Example with specific values:
# python deploy.py datahub user-service dev
# python deploy.py datahub ui prod latest
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
- **ECS Console** → Clusters → `${PROJECT_NAME}-Services-{ENV}` → Services
- Watch for "Running count" to reach "Desired count" (1/1)

✅ **Verify:** Check all services are deployed
```bash
aws ecs describe-services \
  --cluster ${PROJECT_NAME}-cluster-${ENV} \
  --services $(aws ecs list-services --cluster ${PROJECT_NAME}-Services-${ENV} --query 'serviceArns' --output text) \
  --query 'services[*].{Name:serviceName,Running:runningCount,Desired:desiredCount,Status:status}' \
  --output table
```

**Expected output:** All services should show `Running: 1, Desired: 1, Status: ACTIVE`


## Post-Deployment Verification

### Step 22: Verify Deployment
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
1. ${PROJECT_NAME}-Networking-{ENV}
2. ${PROJECT_NAME}-S3-{ENV}
3. ${PROJECT_NAME}-LoadBalancer-{ENV}
4. ${PROJECT_NAME}-RDS-{ENV}
5. ${PROJECT_NAME}-Route53-{ENV} (optional)
6. ${PROJECT_NAME}-CloudWatch-{ENV}
7. ${PROJECT_NAME}-SQS-{ENV}
8. ${PROJECT_NAME}-ECR-{ENV}
9. ${PROJECT_NAME}-OpenSearch-{ENV}
10. ${PROJECT_NAME}-SecretsManager-{ENV}
11. ${PROJECT_NAME}-ECS-{ENV}
12. ${PROJECT_NAME}-Lambda-{ENV}
13. ${PROJECT_NAME}-SES-{ENV}
14. ${PROJECT_NAME}-EventBridge-{ENV}

**Services you should see running (7 total):**
```bash
# Check ECS services
aws ecs list-services --cluster ${PROJECT_NAME}-Services-${ENV} --query 'serviceArns' --output table
```

Expected services:
1. ${PROJECT_NAME}-UserService
2. ${PROJECT_NAME}-SubmissionService
3. ${PROJECT_NAME}-ReportService
4. ${PROJECT_NAME}-DownloadService
5. ${PROJECT_NAME}-EntityService
6. ${PROJECT_NAME}-SearchService
7. ${PROJECT_NAME}-Frontend (UI)

---

### Step 23: Test Application

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
  --stack-name ${PROJECT_NAME}-{ENV}-{STACK-NAME} \
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
aws logs tail /ecs/${PROJECT_NAME}-user-service --follow
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
aws logs tail /aws/lambda/${PROJECT_NAME}-OpenSearchRefresh-${ENV} --follow
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
aws opensearch describe-domain --domain-name ${PROJECT_NAME}-opensearch-${ENV} \
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

### Step 1: Empty Containers

Before deleting the stack, empty S3 buckets, RDS, and ECR so CloudFormation can remove these resources.

#### Step 1a: Empty S3 Buckets

S3 buckets with content cannot be deleted by CloudFormation:

```bash
# List all DataHub S3 buckets
aws s3 ls | grep ${PROJECT_NAME}-${DataHubUniqueId}-${ENV}

# Empty each bucket
aws s3 ls | grep ${PROJECT_NAME}-${DataHubUniqueId}-${ENV} | awk '{print $3}' | while read bucket; do
  echo "Emptying bucket: $bucket"
  aws s3 rm s3://$bucket --recursive
done
```

#### Step 1b: Empty RDS

Disable deletion protection on the RDS instance so the stack can delete it. If your RDS instance has a final snapshot or other retention settings that block deletion, adjust or remove them.

```bash
# List RDS instances for this project
aws rds describe-db-instances --query "DBInstances[?contains(DBInstanceIdentifier, '${PROJECT_NAME}')].DBInstanceIdentifier" --output text

# Disable deletion protection (replace INSTANCE_ID with your RDS instance identifier)
aws rds modify-db-instance \
  --db-instance-identifier ${PROJECT_NAME}-${DataHubUniqueId}-${ENV}-postgres \
  --no-deletion-protection \
  --apply-immediately
```

If the stack still cannot delete RDS (e.g., due to automated snapshots), you may need to delete the DB instance manually first, then delete the stack.

#### Step 1c: Empty ECR

ECR repositories with images cannot be deleted by CloudFormation. Delete all images in each repository:

```bash
# List all DataHub ECR repositories
aws ecr describe-repositories --query "repositories[?contains(repositoryName, '${PROJECT_NAME}')].repositoryName" --output text

for repo in $(aws ecr describe-repositories --query "repositories[?contains(repositoryName, '${PROJECT_NAME}')].repositoryName" --output text); do
  echo "Emptying ECR repository: $repo"
  aws ecr batch-delete-image --repository-name $repo --image-ids $(aws ecr list-images --repository-name $repo --query 'imageIds[*]' --output json) 2>/dev/null || true
done
```

### Step 2: Delete Stacks in Reverse Order

```bash
# Delete stacks in reverse dependency order
stacks=(
  "${PROJECT_NAME}-EventBridge-${ENV}"
  "${PROJECT_NAME}-Lambda-${ENV}"
  "${PROJECT_NAME}-SES-${ENV}"
  "${PROJECT_NAME}-ECS-${ENV}"
  "${PROJECT_NAME}-SecretsManager-${ENV}"
  "${PROJECT_NAME}-OpenSearch-${ENV}"
  "${PROJECT_NAME}-ECR-${ENV}"
  "${PROJECT_NAME}-SQS-${ENV}"
  "${PROJECT_NAME}-CloudWatch-${ENV}"
  "${PROJECT_NAME}-Route53-${ENV}"
  "${PROJECT_NAME}-RDS-${ENV}"
  "${PROJECT_NAME}-LoadBalancer-${ENV}"
  "${PROJECT_NAME}-S3-${ENV}"
  "${PROJECT_NAME}-Networking-${ENV}"
)

for stack in "${stacks[@]}"; do
  echo "Deleting stack: $stack"
  aws cloudformation delete-stack --stack-name $stack
  
  echo "Waiting for $stack deletion to complete..."
  aws cloudformation wait stack-delete-complete --stack-name $stack
  
  echo "$stack deleted successfully"
done

echo "Cleanup complete!"
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

