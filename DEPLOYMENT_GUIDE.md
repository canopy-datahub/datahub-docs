# Canopy Platform AWS Deployment Guide

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
- [ ] **Step 1:** [Create Canopy Home](#step-1-create-canopy-home) (1 min)
- [ ] **Step 2:** [Set Up Canopy CLI](#step-2-set-up-canopy-cli) (5 min)
- [ ] **Step 3:** [Clone Repositories](#step-3-clone-repositories) (5 min)
- [ ] **Step 4:** [Install and Configure AWS CLI](#step-4-install-and-configure-aws-cli) (10 min)
  - [ ] **Step 4a:** [Install AWS CLI](#step-4a-install-aws-cli)
  - [ ] **Step 4b:** [Set up AWS Credentials](#step-4b-set-up-aws-credentials-for-deployment)
  - [ ] **Step 4c:** [Grant IAM Permissions](#step-4c-grant-iam-permissions)
- [ ] **Step 5:** [Set Environment Variables](#step-5-set-environment-variables) (5 min)

### Core AWS Infrastructure (~3 hours)
- [ ] **Step 6:** [Deploy the Bootstrap Stack](#step-6-deploy-the-bootstrap-stack) (2 min)
- [ ] **Step 7:** [Deploy Networking Stack](#step-7-deploy-networking-stack) (5 min)
- [ ] **Step 8:** [Deploy S3 Stack](#step-8-deploy-s3-stack) (5 min)
  - [ ] **Step 8a:** [Configure DeploymentId](#step-8a-configure-datahubuniqueid-for-s3-buckets)
  - [ ] **Step 8b:** [Deploy into AWS](#step-8b-deploy-into-aws)
- [ ] **Step 9:** [Deploy LoadBalancer Stack](#step-9-deploy-loadbalancer-stack) (5 min)
- [ ] **Step 10:** [Deploy RDS Stack](#step-10-deploy-rds-stack) (15 min)
  - [ ] **Step 10a:** [Configure RDS Security Group](#step-10a-configure-rds-security-group)
  - [ ] **Step 10b:** [Set RDS Master Password](#step-10b-set-rds-master-password)
  - [ ] **Step 10c:** [Deploy RDS Stack](#step-10c-deploy-rds-stack)
- [ ] **Step 11:** [Database Configuration](#step-11-database-configuration)
  - [ ] **Step 11a:** [Setup Database Schema](#step-11a-setup-database-schema) (30 min)
  - [ ] **Step 11b:** [Configure pg_cron Scheduler](#step-11b-configure-pg_cron-scheduler-optional) (Optional, 10 min)
- [ ] **Step 12:** [Deploy Route53 Stack](#step-12-deploy-route53-stack-optional) (Optional) (5 min)
- [ ] **Step 13:** [Deploy CloudWatch Stack](#step-13-deploy-cloudwatch-stack) (5 min)
- [ ] **Step 14:** [Deploy SQS Stack](#step-14-deploy-sqs-stack) (5 min)
- [ ] **Step 15:** [Deploy ECR Stack](#step-15-deploy-ecr-stack) (5 min)
- [ ] **Step 16:** [Deploy OpenSearch Stack](#step-16-deploy-opensearch-stack) (20 min)
  - [ ] **Step 16a:** [Configure OpenSearch Credentials](#step-16a-configure-opensearch-credentials)
  - [ ] **Step 16b:** [Deploy OpenSearch Stack](#step-16b-deploy-opensearch-stack)
- [ ] **Step 17:** [Deploy SecretsManager Stack](#step-17-deploy-secretsmanager-stack) (5 min)
- [ ] **Step 18:** [OpenSearch Reindex Lambda](#step-18-opensearch-reindex-lambda)
  - [ ] **Step 18a:** [Create Lambda Layer](#step-18a-create-lambda-layer) (5 min)
  - [ ] **Step 18b:** [Upload OpenSearch Reindex Lambda Code](#step-18b-upload-opensearch-reindex-lambda-code) (5 min)
- [ ] **Step 19:** [Upload Email Service Lambda Code](#step-19-upload-email-service-lambda-code) (5 min)
- [ ] **Step 20:** [Deploy Lambda Stack](#step-20-deploy-lambda-stack) (10 min)
- [ ] **Step 21:** [Deploy ECS Stack](#step-21-deploy-ecs-stack) (20 min)
- [ ] **Step 22:** [Deploy Keycloak](#step-22-deploy-keycloak) (30 min)
  - [ ] **Step 22a:** [Deploy Keycloak ECS Stack](#step-22a-deploy-keycloak-ecs-stack) (5 min)
  - [ ] **Step 22b:** [Build and Push Keycloak Image](#step-22b-build-and-push-keycloak-image) (5 min)
  - [ ] **Step 22c:** [Initial Keycloak Configuration](#step-22c-initial-keycloak-configuration) (10 min)
- [ ] **Step 23:** [Deploy SES Stack](#step-23-deploy-ses-stack) (20 min)
  - [ ] **Step 23a:** [Deploy SES Stack](#step-23a-deploy-ses-stack) (10 min)
  - [ ] **Step 23b:** [Configure SES Email Verification](#step-23b-configure-ses-email-verification) (10 min)
- [ ] **Step 24:** [Deploy EventBridge Stack](#step-24-deploy-eventbridge-stack) (5 min)
- [ ] **Step 25:** [Deploy TransferFamily Stack](#step-25-deploy-transferfamily-stack-sftp-server) (10 min)
  - [ ] **Step 25a:** [Allocate Elastic IP](#step-25a-optional-allocate-elastic-ip) (Optional)
  - [ ] **Step 25b:** [Deploy TransferFamily Stack](#step-25b-deploy-transferfamily-stack)
  - [ ] **Step 25c:** [Create SFTP User Account](#step-25c-create-an-sftp-user-account) (repeat per submitter)
  - [ ] **Step 25d:** [Register SFTP User in Database](#step-25d-register-the-sftp-user-in-the-database--required) (repeat per submitter)
- [ ] **Step 26:** [Build and Deploy Services](#step-26-build-and-deploy-services) (50 min)
  - [ ] **Step 26a:** [Configure UI Environment Variables](#step-26a-configure-ui-environment-variables)
  - [ ] **Step 26b:** [Install Python Dependencies](#step-26b-install-python-dependencies)
  - [ ] **Step 26c:** [Deploy All Services](#step-26c-deploy-all-services)

### Post-Deployment (10 min)
- [ ] **Step 27:** [Verify Deployment](#step-27-verify-deployment)
- [ ] **Step 28:** [Test Application](#step-28-test-application)

---

## Step-by-Step Deployment

## Pre-Deployment Setup

### Step 1: Create Canopy Home
**Time:** 1 minute

To work with the Canopy DataHub infrastructure, we strongly recommend that you create a dedicated directory for all the repositories you will be working with.
This guide will assume that you are following this best practice.

🖥️ **Execute**:
```bash
# Create workspace directory
mkdir ~/CANOPY
```

### Step 2: Set Up Canopy CLI
**Time:** 5 minutes

Canopy CLI is a command-line tool that you can use to interact with the Canopy DataHub infrastructure.
It hides the complexity of several commands during this setup and allows you to focus on the actual deployment.
We will use it extensively throughout this guide.

You can follow the guide directly from the [Canopy CLI repository](https://github.com/canopy-datahub/canopy-cli), or you can install it using the following commands:

🖥️ **Execute**:
```bash
export CANOPY_HOME=~/CANOPY

cd ${CANOPY_HOME}
git clone https://github.com/canopy-datahub/canopy-cli

cd canopy-cli
# checkout the develop branch for the latest features
git checkout develop

# create venv
python -m venv ./.venv
source .venv/bin/activate
pip install -r requirements.txt

# create an alias
alias canopycli='source $CANOPY_HOME/canopy-cli/cli.sh'

# test it
canopycli
# or
python cli.py
```

### Step 3: Clone Repositories
**Time:** 5 minutes

Clone all necessary repositories from the Canopy DataHub GitHub organization:

🖥️ **Execute**:
```bash
canopycli git clone all

# Verify all repositories are cloned
cd ${CANOPY_HOME}
ls -la
```

✅ **Verify:** You should see 14 directories in `~/CANOPY`

---

### Step 4: Install and Configure AWS CLI
**Time:** 10 minutes

You need to install the AWS CLI and have an IAM user with the required permissions set up so you can run AWS commands from your machine.

#### Step 4a: Install AWS CLI
🖥️ **Execute**:
```bash
# For MacOS:
brew install awscli
# Verify installation
aws --version
```

#### Step 4b: Set up AWS credentials for deployment:

**Get AWS credentials:**
1. Log into AWS Console
2. If you don't have an IAM user yet, go to **IAM → Users → Create user**, enter a username (e.g. `canopy-dev`), and click **Next**. On the permissions screen, select **Attach policies directly**, search for **AdministratorAccess**, select it, and click **Next**. Review the summary, then click **Create user**.
3. Go to IAM → Users → Select your user
4. Security Credentials tab → Create Access Key. On the **Access key best practices & alternatives** page choose **Command Line Interface (CLI)**.
5. Download and save the credentials securely
6. Replace `YOUR_ACCESS_KEY_ID` and `YOUR_SECRET_ACCESS_KEY` below

⚠️ **Important:** In the following guide `canopy-profile` is the AWS profile name. Feel free to choose another name but use it consistently throughout the guide.

🖥️ **Execute**:
```bash
# Create AWS credentials file
mkdir -p ~/.aws

# Add your credentials
cat <<EOL >> ~/.aws/credentials
[canopy-profile]
aws_access_key_id=YOUR_ACCESS_KEY_ID
aws_secret_access_key=YOUR_SECRET_ACCESS_KEY
EOL

# Set default region
cat <<EOL >> ~/.aws/config
[profile canopy-profile]
region=us-east-1
output=json
EOL
```
**IMPORTANT: Unset any existing AWS credentials from the environment**

🖥️ **Execute**:
```bash
unset AWS_ACCESS_KEY_ID
unset AWS_SECRET_ACCESS_KEY
unset AWS_SESSION_TOKEN

# Now set the profile and region
export AWS_PROFILE=canopy-profile
export AWS_REGION=us-east-1
```

✅ **Verify:** Test AWS connectivity
```bash
aws sts get-caller-identity --no-cli-pager
```

**Expected output:** Should show your AWS account ID, user ARN, and user ID.
*Please make sure you are using the correst aws profile*

#### Step 4c: Grant IAM Permissions

The IAM user performing the deployment must have permissions for the following AWS services: CloudFormation, VPC, EC2, S3, Elastic Load Balancing (ALB), RDS, Route53, CloudWatch, SQS, ECR, OpenSearch Service, Secrets Manager, ECS, SES, Lambda, EventBridge, and IAM.

If you followed Step 4b and attached **AdministratorAccess** during user creation, you can skip the command below. Otherwise, run it now to grant the required permissions:

🖥️ **Execute**:
```bash
# Change to your own user name below
aws iam attach-user-policy \
  --user-name canopy-dev \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```
---

### Step 5: Set Environment Variables
**Time:** 5 minutes

⚠️ **SHORTCUT:**
This guide will introduce several environment variables that will be used throughout the deployment.
You can bootstrap them by running the following command:

⚠️ **IMPORTANT:**
Change `dev` to `test` or `prod` to deploy to a different environment.
During this guide, `${CANOPY_ENV}` is just a string value, but it is recommended to use lowercase only. It is used to suffix the AWS resource names.

🖥️ **Execute**:
```bash
export CANOPY_ENV=dev
canopycli init
```

✅ **Verify:** As a result, these 3 files will show up in your Canopy home directory:
```bash
aws-parameters-${CANOPY_ENV}-${USERNAME}.json                                                                                   │
canopy-profile-native-develop.sh                                                                               │
set-canopy-env.sh
```

Feel free to rename `aws-parameters-${CANOPY_ENV}-${USERNAME}.json` based on your needs.
BUT make sure to update `set-canopy-env.sh` as well, because it references the json file.

⚠️ **IMPORTANT:** Reload `set-canopy-env.sh` to set all the environment variables, any time when you make a significant change to the `aws-parameters-${CANOPY_ENV}--${USERNAME}.json` file.

🖥️ **Use**:
```bash
source ${CANOPY_HOME}/set-canopy-env.sh
```
---

💡 **Tip:** Add these to your `~/.bashrc` or `~/.zshrc` to persist across sessions.

---

## Core AWS Infrastructure

### Step 6: Deploy the Bootstrap Stack
**Time:** 2 minutes

Before deploying any service-specific stacks, you need to deploy a one-time **Bootstrap** stack at the AWS account level.

**What the Bootstrap stack does:**

`Bootstrap.yaml` creates shared prerequisites that only need to exist **once per AWS account**, not once per project or environment. Currently, it provisions:

- **`AWSServiceRoleForAmazonOpenSearchService`** — the IAM service-linked role Amazon OpenSearch Service needs in order to manage VPC network interfaces on your behalf. Without this role, the OpenSearch stack (Step 16) will fail with an `Invalid request` error on fresh AWS accounts.

**Why a separate stack:**

- The service-linked role is **account-wide** (shared by every IAM user and every project in the account), so it does not belong inside a per-environment stack.

**How it fits the overall deployment:**

The rest of the deployment is split into per-environment CloudFormation stacks under `datahub-cloud-replication/modules/` (`Networking.yaml`, `S3.yaml`, `LoadBalancer.yaml`, `RDS.yaml`, `OpenSearch.yaml`, `SecretsManager.yaml`, `Lambda.yaml`, `ECS.yaml`, `ECS-Keycloak.yaml`, `SES.yaml`, `EventBridge.yaml`, `TransferFamily.yaml`, …). Each of those stacks follows the same pattern:

- Named `${CANOPY_PROJECT_NAME}-<Module>-${CANOPY_ENV}` and tagged with `projectname` + `environment`.
- Configured from the shared parameter file `${CANOPY_AWS_PARAMETER_FILE}` generated by `canopycli init`.
- Deployed with `aws cloudformation deploy`, which is idempotent — re-running updates the stack in place via a change set.

Bootstrap sits **before** all of them because it provisions the account-level scaffolding they rely on.

**Skip condition:** If the `AWSServiceRoleForAmazonOpenSearchService` role already exists in your AWS account (e.g. someone else deployed OpenSearch in this account before), you can skip the deploy below — Bootstrap will simply be a no-op. To check:

🖥️ **Execute**:
```bash
aws iam get-role \
  --role-name AWSServiceRoleForAmazonOpenSearchService \
  --profile ${AWS_PROFILE}
```

- If the command returns role details → the role exists, you can skip this step.
- If the command returns `NoSuchEntity` → run the deploy below.

**Deploy:**

🖥️ **Execute**:
```bash
canopycli aws cloudformation deploy Bootstrap
```

✅ **Verify:**
```bash
canopycli aws cloudformation status Bootstrap
```
**Expected:** `CREATE_COMPLETE`

---

### Step 7: Deploy Networking Stack
**Time:** 5 minutes

Creates VPC, subnets, security groups, and networking infrastructure.

🖥️ **Execute**:
```bash
canopycli aws cloudformation deploy Networking
```

✅ **Verify:** 
```bash
canopycli aws cloudformation status Networking
```
**Expected:** `CREATE_COMPLETE`

---

### Step 8: Deploy S3 Stack
**Time:** 5 minutes

Creates S3 buckets for data files, metadata files, data dictionary files and lambda functions code.

#### step 4a. Configure DeploymentId for S3 Buckets 

S3 bucket names must be globally unique. Update the `DeploymentId` parameter, and **reload the env vars**:

🖥️ **Execute**:
```bash
# Open parameters file for your environment
nano ${CANOPY_AWS_PARAMETER_FILE} 

# edit your `DeploymentId` parameter
# important: use lowercase only, as this value will also be used in s3 bucket names, which only allow lowercase

source ${CANOPY_HOME}/set-canopy-env.sh
```

**Update the `DeploymentId` value:**
- Use lowercase letters, numbers, and hyphens only
- Keep it short (5-20 characters)
- Make it unique (e.g., your institution name + random string)
- **Do not repeat the `ProjectName` or the `Environment` values!** Those will be used anyways everywhere where `DeploymentId` is used.
- Examples: `stanford-abc123`, `0001`, `acme-ABC`

**Bucket names that will be created:**
- `${PROJECT_NAME}-upload-portal-${CANOPY_DEPLOYMENT_ID}-$CANOPY_ENV}`
- `${PROJECT_NAME}-sftp-${CANOPY_DEPLOYMENT_ID}-${CANOPY_ENV}`
- `${PROJECT_NAME}-review-{$CANOPY_DEPLOYMENT_ID}-{$CANOPY_ENV}`
- `${PROJECT_NAME}-default-${CANOPY_DEPLOYMENT_ID}-${CANOPY_ENV}`
- `${PROJECT_NAME}-lambda-artifacts-{$CANOPY_DEPLOYMENT_ID}-${CANOPY_ENV}`
- `${PROJECT_NAME}-resources-${CANOPY_DEPLOYMENT_ID}-${CANOPY_ENV}`

#### Step 8b. Deploy into AWS
🖥️ **Execute**:
```bash
canopycli aws cloudformation deploy S3
```

⚠️ **Troubleshooting:** If deployment fails with `BucketAlreadyExists`, go back to Step 8a and choose a different `DeploymentId`.

✅ **Verify:** 
```bash
aws s3 ls | grep ${CANOPY_PROJECT_NAME}
```
**Expected:** Should list 6 buckets

---

### Step 9: Deploy LoadBalancer Stack
**Time:** 5 minutes

Creates Application Load Balancer, target groups, and routing rules.

🖥️ **Execute**:
```bash
canopycli aws cloudformation deploy LoadBalancer
```

✅ **Verify:** Get ALB DNS name
```bash
canopycli aws elb dns
```
**Expected:** DNS name like `${CANOPY_PROJECT_NAME}-${CANOPY_ENV}-123456789.us-east-1.elb.amazonaws.com`

#### Post-deployment: Set ALB URL for SecretsManager and UI

The ALB DNS from Step 9 must be used in two places later, so the application and backend use the correct URL:

1. **SecretsManager (Step 17)** — Save the ALB DNS name in `aws-parameters-${CANOPY_ENV}-{USERNAME}.json` now, before deploying the SecretsManager stack. `SecretsManager.yaml` reads `HostURL` as a CloudFormation parameter and injects it directly into the secret, so no manual editing of the secret is required.

   ```json
   "LoadBalancerDomainName": "http://${CANOPY_PROJECT_NAME}-${CANOPY_ENV}-123456789.us-east-1.elb.amazonaws.com"
   ```

   > Replace the placeholder above with the actual DNS name returned by the verify command above.

---

### Step 10: Deploy RDS Stack
**Time:** 15 minutes

Creates RDS PostgreSQL database instance.

#### Step 10a. Configure RDS Security Group

Before deploying, get your public IP and save it as `MyIP` in `parameters-${CANOPY_ENV}.json`. `RDS.yaml` reads this value as a CloudFormation parameter and adds it to the RDS security group, so no manual file editing is required.

```bash
# Get your public IP
curl -4 ifconfig.me
```

Then set the value in `parameters-${CANOPY_ENV}.json`:

```json
"MyIP": "203.0.113.5/32"
```

> Replace the IP above with the one returned by the `curl` command. The `/32` suffix is required.

#### Step 10b. Set RDS Master Password

The parameter file ships with a placeholder password. **Set it before deploying** — CloudFormation passes it to `RDS.yaml` via `--parameter-overrides` and uses it as the RDS master password at creation time.

In `aws-parameters-${CANOPY_ENV}-${USERNAME}.json`, replace the placeholder:

```json
"CanopyParamDbMasterPassword": "YourSecurePassword123!"
```

#### Step 10c. Deploy RDS Stack

```bash
canopycli aws cloudformation deploy RDS
```

⏳ **Wait time:** 10-15 minutes for RDS instance to be created

✅ **Verify:** Get RDS endpoint
```bash
canopycli aws rds endpoint
```
**Expected:** Endpoint like `${CANOPY_PROJECT_NAME}-postgresql-${CANOPY_ENV}.abc123.us-east-1.rds.amazonaws.com`

#### Post-Deployment: Update Secrets Manager

Save the RDS endpoint as `RDSEndpoint` in `aws-parameters-${CANOPY_ENV}-${USERNAME}.json` before deploying the SecretsManager stack. `SecretsManager.yaml` reads `RDSEndpoint` as a CloudFormation parameter and injects it directly into the secret, so no manual editing of the secret is required.

```json
"RDSEndpoint": "my-db.myserver.us-east-1.rds.amazonaws.com"
```

> Replace the placeholder above with the actual endpoint shown at the end of Step 10b.
---

### Step 11: Database Configuration

---

### Step 11a: Set up Database Schema
**Time:** 30 minutes

This step creates the database schema, tables, views, and initial data in your RDS PostgreSQL instance.

#### 📘 Detailed Documentation

**See:** [README_RDS_DEPLOYMENT.md](https://github.com/canopy-datahub/datahub-development/blob/feature/aws/db/postgres/db-create-scripts/README_RDS_DEPLOYMENT.md)


#### Quick Setup (Automated)

Use the automated Python deployment script:

```bash
# Navigate to the python script
cd ${CANOPY_HOME}/datahub-development/db/postgres/db-create-scripts

python deploy_to_rds.py --project-name ${CANOPY_PROJECT_NAME} --env ${CANOPY_ENV} --region ${AWS_REGION} --profile ${AWS_PROFILE}

# Examples for different environments:
# python deploy_to_rds.py --project-name canopy --env dev --region us-east-1 --profile datahub-rep
# python deploy_to_rds.py --project-name canopy --env test --region us-east-1 --profile datahub-rep
# python deploy_to_rds.py --project-name canopy --env prod --region us-east-1 --profile datahub-rep
```

**Script Parameters:**
- `--project-name`: Project name (e.g., `canopy`) - **REQUIRED**
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

✅ **Verify:** The script will confirm the successful deployment and show table counts.
---

### Step 11b: Configure pg_cron Scheduler *(Optional)*
**Time:** 10 minutes

Automates the weekly `hub_content_metrics` snapshot by scheduling `sp_generate_hub_content_metrics()` inside PostgreSQL. Skip this if you prefer to run the procedure manually.

📄 **See full setup guide:** [PGCRON_METRICS_SETUP.md](./PGCRON_METRICS_SETUP.md)

---

### Step 12: Deploy Route53 Stack (Optional)
**Time:** 5 minutes
💡 **Skip this step** if you don't have Route53 hosted zones configured.

Creates DNS records and routing configurations.

⚠️ **Note:** This stack is organization-dependent. Consult with your DNS administrator before deploying.

```bash
canopycli aws cloudformation deploy Route53
```

---

### Step 13: Deploy CloudWatch Stack
**Time:** 5 minutes

Creates monitoring, logging, and alerting infrastructure.

```bash
canopycli aws cloudformation deploy CloudWatch
```

✅ **Verify:** List log groups
```bash
canopycli aws logs list
```

---

### Step 14: Deploy SQS Stack
**Time:** 5 minutes

Creates SQS queues for message processing.

```bash
canopycli aws cloudformation deploy SQS
```

---

### Step 15: Deploy ECR Stack
**Time:** 5 minutes

Creates ECR repositories for container images.

```bash
canopycli aws cloudformation deploy ECR
```

**What's created:**
- ECR repositories for each microservice:
  - `${CANOPY_PROJECT_NAME}-approved-data-service/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-download-service/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-entityservice/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-keycloak/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-opensearchrefresh/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-publicationscript/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-report-service/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-reporter/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-search/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-submission-service/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-ui/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-user-service/{CANOPY_ENV}`

✅ **Verify:** List ECR repositories
```bash
canopycli aws ecr list
```

---

### Step 16: Deploy OpenSearch Stack
**Time:** 20 minutes

Creates OpenSearch domain for search and analytics.

#### Step 16a. Configure OpenSearch Credentials

⚠️ **Security:** Change the default OpenSearch password before deployment!

Edit `aws-parameters-${CANOPY_ENV}-${USERNAME}.json` and update:
```json
{
  "OpenSearchUsername": "opensearch",
  "OpenSearchPassword": "ChangeMe@2026"
}
```

**Password requirements:**
- Minimum 8 characters
- Must contain uppercase, lowercase, number, and special character

#### Step 16b. Deploy OpenSearch Stack

> **Prerequisite:** The `AWSServiceRoleForAmazonOpenSearchService` service-linked role must exist in your AWS account before this stack can deploy. If you ran [Step 6: Deploy the Bootstrap Stack](#step-6-deploy-the-bootstrap-stack), this is already taken care of. If you skipped Step 6, go back and run it now — otherwise this deploy will fail with an `Invalid request` error on fresh accounts.

```bash
canopycli aws cloudformation deploy OpenSearch
```

⏳ **Wait time:** 15-20 minutes for OpenSearch domain to be created

✅ **Verify:** Get OpenSearch endpoint
```bash
canopycli aws opensearch endpoint
```
**Expected:** Endpoint like `vpc-datahub-opensearch-dev-abc123.us-east-1.es.amazonaws.com`

#### Post-Deployment: Update Secrets Manager

After OpenSearch stack deployment, save the OpenSearch endpoint as `OpenSearchEndpoint` in `parameters-${CANOPY_ENV}.json` before deploying the SecretsManager stack. `SecretsManager.yaml` reads `OpenSearchEndpoint` as a CloudFormation parameter and injects it directly into the secret as `SEARCH_HOST`, so no manual editing of the secret is required.

```json
"OpenSearchEndpoint": "vpc-my-domain-abc123.us-east-1.es.amazonaws.com"
```

> Replace the placeholder above with the actual endpoint shown at the end of Step 16b.

⚠️ **Note:** Step 16b will display the OpenSearch endpoint. Save it in `parameters-${CANOPY_ENV}.json` as shown above — do **not** edit Secrets Manager directly.

---

### Step 17: Deploy SecretsManager Stack
**Time:** 5 minutes

Creates secrets for database credentials, API keys, and OpenSearch configuration.

⚠️ **Important:** Deploy this stack only after **RDS**, **Load Balancer**, and **OpenSearch** are deployed. The secret stores endpoints from those stacks:

- **host** — RDS endpoint (from Step 10)
- **SEARCH_HOST** — OpenSearch endpoint (from Step 16)
- **HostURL** — Load Balancer URL (from Step 9)

Also update these email values for your environment before deployment:

- **supportEmail** — Address used as the sender (From) for system emails and often in Cc (e.g., study registration, support requests). Use an address on a domain verified in SES (e.g. `*@stanford.edu` if that domain is verified).
- **StakeholderEmailsStudyReg** — Comma-separated list of additional recipients (Cc) for study registration and study approval emails (e.g., NIH officers). Leave empty or set as needed.

```bash
canopycli aws cloudformation deploy SecretsManager
```

✅ **Verify:** Check secret was created

```bash
canopycli aws secrets describe
```

---

### Step 18: OpenSearch Reindex Lambda 
📘 For detailed OpenSearch reindex lambda deployment documentation, see: [README_OPENSEARCH_REINDEX_LAMBDA_DEPLOYMENT.md](https://github.com/canopy-datahub/datahub-development/blob/feature/aws/opensearch/opensearch_reindex/README_OPENSEARCH_REINDEX_LAMBDA_DEPLOYMENT.md)

#### Step 18a: Create Lambda Layer
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

**Ensure Docker is running** (required for ARM64 layer build):

```bash
# macOS: open Docker Desktop if installed (or start Docker from Applications)
open -a Docker

# Wait for Docker to finish starting (whale icon in menu bar), then verify:
docker info

# Navigate to python script folder 
cd ${CANOPY_HOME}/datahub-development/opensearch/opensearch_reindex

# Create Lambda layer with ARM64-compatible dependencies
python create_layer.py dependency-layer ${AWS_REGION} ${AWS_PROFILE}
```

**What this does:**
1. Uses Docker to build ARM64-compatible Python dependencies
2. Packages dependencies: `psycopg2-binary`, `opensearch-py`, `requests-aws4auth`, `boto3`, `requests`
3. Publishes layer to AWS Lambda
4. Outputs Layer ARN for CloudFormation

⏳ **Wait time:** 3-5 minutes (Docker build + upload)

✅ **Verify:** Note the Layer ARN from the output, such as:
```
Layer ARN:
  arn:aws:lambda:us-east-1:accountId:layer:dependency-layer:1
```
#### Post-Operation: Update parameters file

After the dependency layer was published, save the ARN as `LambdaDependencyLayerArn` in `parameters-${CANOPY_ENV}.json`.

```json
"LambdaDependencyLayerArn": "REPLACEME"
```

> Replace the placeholder above with the actual arn shown at the end of Step 18a.

---

#### Step 18b: Upload OpenSearch Reindex Lambda Code
**Time:** 5 minutes

Package and upload Lambda function code to S3.

**Script Usage:**
```bash
python deploy_lambda.py ${CANOPY_PROJECT_NAME} ${CANOPY_ENV} ${CANOPY_DEPLOYMENT_ID}
```

**Parameters:**

- **project-name**: Project name (e.g., `datahub`, `canopy`) - **REQUIRED**
- **env**: `dev`, `test`, or `prod` - **REQUIRED**
- **DeploymentId**: Unique identifier for S3 bucket (e.g., `stanford`) - **REQUIRED**
  - S3 bucket: `${CANOPY_PROJECT_NAME}-lambda-artifacts-${CANOPY_DEPLOYMENT_ID}-${CANOPY_ENV}`

**What this does:**
1. Validates required files exist
2. Packages Lambda code with mapping files:
   - `opensearch_reindex_aws.py`
   - `search_index_mapping.json`
   - `variable_index_mapping.json`
   - `autocomplete_index_mapping.json`
3. Creates ZIP (should be <100KB)
4. Uploads to S3: `s3://${CANOPY_PROJECT_NAME}-lambda-artifacts-${CANOPY_DEPLOYMENT_ID}-${CANOPY_ENV}/opensearch-refresh/`

⚠️ **Note:** Dependencies are NOT included (they're in the layer from Step 18a)

✅ **Verify:** Check S3 upload
```bash
aws s3 ls s3://${CANOPY_PROJECT_NAME}-lambda-artifacts-${CANOPY_DEPLOYMENT_ID}-${CANOPY_ENV}/opensearch-refresh/
```
**Expected:** Should show `opensearch-refresh-lambda.zip`

### Step 19: Upload Email Service Lambda Code

```bash
cd ${CANOPY_HOME}/datahub-service-email

# Build Spring Boot application for Lambda
mvn clean package -DskipTests

# Upload to S3
aws s3 cp target/datahub-service-email-0.0.1-SNAPSHOT-aws.jar \
  s3://${CANOPY_PROJECT_NAME}-lambda-artifacts-${CANOPY_DEPLOYMENT_ID}-${CANOPY_ENV}/email-service/
```
✅ **Verify:** Check S3 upload
```bash
aws s3 ls s3://${CANOPY_PROJECT_NAME}-lambda-artifacts-${CANOPY_DEPLOYMENT_ID}-${CANOPY_ENV}/email-service/
```
**Expected:** Should show `datahub-service-email-0.0.1-SNAPSHOT-aws.jar`
---

### Step 20: Deploy Lambda Stack
**Time:** 10 minutes

Creates Lambda functions for OpenSearch indexing and email service.

```bash
canopycli aws cloudformation deploy Lambda
```

**What's created:**
- Lambda function: `${CANOPY_PROJECT_NAME}-OpenSearchRefresh`
  - Runtime: Python 3.11 on ARM64
  - VPC: Deployed in private subnets
  - Attached layer from Step 18a
  - Code from S3 (Step 18b)
- Lambda function: `${CANOPY_PROJECT_NAME}-EmailService`
  - Runtime: Java 17
  - Code from S3
- IAM roles and permissions
- VPC configuration
- Security group access

✅ **Verify:** Check Lambda functions exist
```bash
canopycli aws lambda list
```

#### Test OpenSearch Lambda (Optional)

```bash
canopycli aws lambda invoke OpenSearchRefresh
```

---

### Step 21: Deploy ECS Stack
**Time:** 20 minutes

Creates ECS cluster, services, and container definitions for all microservices.

```bash
canopycli aws cloudformation deploy ECS
```

⏳ **Wait time:** 15-20 minutes for all services to start

**What's created:**
- ECSr sevices (1 per microservice):
  - Download Service
  - Entity Service
  - Report Service
  - Search Service
  - Submission Service
  - User Service
  - UI

✅ **Verify:** Check ECS services are running
```bash
canopycli aws ecs list-services
```

---

### Step 22: Deploy Keycloak
**Time:** 30 minutes

Deploys Keycloak as a dedicated ECS service for identity and access management.

#### Step 22a: Deploy Keycloak ECS Stack
**Time:** 5 minutes

Deploy the Keycloak-specific ECS service via CloudFormation:

```bash
canopycli aws cloudformation deploy ECS-Keycloak
```

---

#### Step 22b: Build and Push Keycloak Image
**Time:** 5 minutes

Build the Keycloak Docker image and push it to ECR:

```bash
cd ${CANOPY_HOME}/datahub-deployment-scripts
python deploy.py ${CANOPY_PROJECT_NAME} keycloak ${CANOPY_ENV} 26.5.4
```

> The `26.5.4` argument pins the Keycloak image to a specific version tag.

---

#### Step 22c: Initial Keycloak Configuration
**Time:** 10 minutes

> ⚠️ **Note:** These manual steps will be replaced later by an automatic realm import.

Log in to the Keycloak admin console (accessible via the ALB URL on the `/admin/master/console/` path).

##### Create Realm

1. In the top-left dropdown, click **Create realm**
2. Set **Realm name**: `CANOPY`
3. Click **Create**

##### Create Client

Inside the `CANOPY` realm, go to **Clients → Create client**:

**General settings:**

| Field | Value |
|---|---|
| Client type | OpenID Connect |
| Client ID | `canopy-client` |
| Name | *(leave empty)* |
| Description | *(leave empty)* |
| Always display in UI | Off |

**Capability config:**

| Field | Value |
|---|---|
| Authentication flow | Standard flow |
| PKCE Method | Choose... |

**Login settings:**

| Field | Value |
|---|---|
| Root URL | *(leave empty)* |
| Home URL | *(leave empty)* |
| Valid redirect URIs | `http://<YOUR_ALB_URL>/*` |
| Web origins | `http://<YOUR_ALB_URL>` |

Click **Save**.

##### Create Users

Inside the `CANOPY` realm, go to **Users → Create new user**.

**User 1:**

| Field | Value |
|---|---|
| Email verified | On |
| Username | `alice.smith@example.edu` |
| Email | `alice.smith@example.edu` |
| First name | Alice |
| Last name | Smith |

- Click **Create**, then note the generated **UUID**.
- Go to the **Credentials** tab → **Set password**: `******`, Temporary: **Off**

**User 2:**

| Field | Value |
|---|---|
| Email verified | On |
| Username | `bob.johnson@example.edu` |
| Email | `alice.smith@example.edu` |
| First name | Bob |
| Last name | Johnson |

- Click **Create**, then note the generated **UUID**.
- Go to the **Credentials** tab → **Set password**: `******`, Temporary: **Off**

---

### Step 23: Deploy SES Stack
**Time:** 10 minutes

#### Step 23a: Deploy SES Stack

Creates SES email identities for sending emails.

##### 1. Configure Email Identities (Optional)

By default, the SES stack creates three email identities:
- `datahub@stanford.edu` - Primary sender email
- `datahub.dev@stanford.edu` - Development/testing email
- `stanford.edu` - Domain identity (allows sending from any @stanford.edu address)

**To use your own email addresses:**

Edit [`modules/SES.yaml`](https://github.com/canopy-datahub/datahub-cloud-replication/blob/feature/aws/modules/SES.yaml) and replace the default email identities with your organization's emails.

💡 **Tip:** Using a domain identity (e.g., `yourdomain.com`) allows you to send emails from any address at that domain without verifying each individual email.

##### 2. Deploy SES Stack

```bash
canopycli aws cloudformation deploy SES
```

⚠️ **Important:** Email identities must be verified before emails can be sent. Complete Step 23b below.

---

#### Step 23b: Configure SES Email Verification
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

### Step 24: Deploy EventBridge Stack
**Time:** 5 minutes

Creates EventBridge rules to automatically trigger SQS processing when files are uploaded to S3 via SFTP.

```bash
canopycli aws cloudformation deploy EventBridge
```

**What's created:**
- EventBridge rule {ProjectName}-SFTP-{Environment} that monitors this project’s SFTP S3 bucket for file uploads (one rule per project when using the same account)
- Automatic routing of S3 events to SQS SFTP Queue

---

### Step 25: Deploy TransferFamily Stack (SFTP Server)
**Time:** 10 minutes

Creates an AWS Transfer Family SFTP server with custom Lambda-based authentication for secure file uploads.

#### Step 25a: (Optional) Allocate Elastic IP

The Transfer Family server uses `EndpointType: VPC`, which means it sits inside your VPC with no public DNS by default. Without an Elastic IP, the server has no stable public address — the `s-xxxx.server.transfer.amazonaws.com` hostname won't resolve from the internet, and any assigned IP is ephemeral and changes on redeploy.

An Elastic IP gives the server a **fixed public IP** that:
- Resolves the `s-xxxx.server.transfer.amazonaws.com` hostname publicly
- Survives stack redeployments unchanged
- Can be whitelisted by submitters whose institutions require firewall rules by IP

It is strongly recommended for any environment where external submitters will connect. Only skip it for short-lived dev testing where you connect from within the VPC.

For a stable, static SFTP endpoint address that doesn't change on redeploy:

```bash
aws ec2 allocate-address \
  --domain vpc \
  --profile ${AWS_PROFILE} \
  --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value='${CANOPY_PROJECT_NAME}'-sftp-'${CANOPY_ENV}'},{Key=projectname,Value='${CANOPY_PROJECT_NAME}'},{Key=environment,Value='${CANOPY_ENV}'}]'

# Note the AllocationId from the output (format: eipalloc-xxxxxxxx)
```

#### Step 25b: Deploy TransferFamily Stack

```bash
canopycli aws cloudformation deploy TransferFamily
```

**What's created:**
- AWS Transfer Family SFTP server (VPC endpoint, public subnet AZ1)
- Lambda + API Gateway for custom Secrets Manager-based authentication
- `CustomSFTPTransferRole` IAM role for S3 access

✅ **Verify:** Get the SFTP server endpoint

```bash
canopycli aws transfer endpoint
# Expected format: s-1234567890abcdef0.server.transfer.us-east-1.amazonaws.com
```

#### Step 25c: Create an SFTP User Account

Each data submitter needs their own SFTP account. **Repeat this step for every submitter**, including the first one. No stack redeployment is needed — the SFTP server looks up any secret under the `SFTP/` namespace at login time.

```bash
# Set these for each submitter
CANOPY_SFTP_USERNAME="testuser-${CANOPY_ENV}"       # becomes the SFTP login name, e.g. jsmith-dev
CANOPY_SFTP_PASSWORD="TheirStrongPassword"

# Get the shared IAM role ARN (same for all submitters)
ROLE_ARN=$(aws iam get-role \
  --role-name ${CANOPY_PROJECT_NAME}-CustomSFTPTransferRole-${CANOPY_ENV} \
  --profile ${AWS_PROFILE} \
  --query 'Role.Arn' --output text)

# Create the secret in Secret Manager
aws secretsmanager create-secret \
  --name "SFTP/${CANOPY_PROJECT_NAME}-${CANOPY_SFTP_USERNAME}" \
  --description "SFTP credentials for ${CANOPY_SFTP_USERNAME}" \
  --secret-string "{
    \"Password\": \"${CANOPY_SFTP_PASSWORD}\",
    \"Role\": \"${ROLE_ARN}\",
    \"HomeDirectory\": \"/${CANOPY_PROJECT_NAME}-sftp-${CANOPY_DEPLOYMENT_ID}-${CANOPY_ENV}/${CANOPY_ENV}/${CANOPY_SFTP_USERNAME}\",
    \"Policy\": \"{\\\"Version\\\":\\\"2012-10-17\\\",\\\"Statement\\\":[{\\\"Sid\\\":\\\"AllowListingOfUserFolder\\\",\\\"Action\\\":[\\\"s3:ListBucket\\\"],\\\"Effect\\\":\\\"Allow\\\",\\\"Resource\\\":[\\\"arn:aws:s3:::\${transfer:HomeBucket}\\\"],\\\"Condition\\\":{\\\"StringLike\\\":{\\\"s3:prefix\\\":[\\\"\${transfer:HomeFolder}/*\\\",\\\"\${transfer:HomeFolder}\\\"]}}},{\\\"Sid\\\":\\\"HomeDirObjectAccess\\\",\\\"Effect\\\":\\\"Allow\\\",\\\"Action\\\":[\\\"s3:PutObject\\\",\\\"s3:GetObject\\\",\\\"s3:DeleteObject\\\",\\\"s3:DeleteObjectVersion\\\",\\\"s3:GetObjectVersion\\\",\\\"s3:GetObjectACL\\\",\\\"s3:PutObjectACL\\\"],\\\"Resource\\\":\\\"arn:aws:s3:::\${transfer:HomeDirectory}*\\\"}]}\"
  }" \
  --profile ${AWS_PROFILE} \
  --tags Key=projectname,Value=${CANOPY_PROJECT_NAME} Key=environment,Value=${CANOPY_ENV}
```

> - The `HomeDirectory` path `/{bucket}/{env}/{username}` is required — it determines the S3 key prefix that EventBridge watches for.
> - The `Policy` scopes the session to the user's own folder only. The `${transfer:...}` placeholders are resolved by Transfer Family at login time.
> - The SFTP login username equals `CANOPY_SFTP_USERNAME` (the secret name minus the `SFTP/` prefix).

#### Step 25d: Register the SFTP User in the Database ⚠️ Required

**Repeat this step for every submitter.** The platform identifies which user made an upload by matching the S3 path against `sftp_path` in the `users` table. The submitter must already have a platform user account (registered on the platform) before this step.

```sql
-- Pattern: '{CANOPY_ENV}/{CANOPY_SFTP_USERNAME}'
UPDATE users
SET sftp_path = 'dev/testuser-dev'
WHERE email_address = 'test@test.com';
```

#### Testing SFTP Access

First, get the public IP of the Elastic IP attached to the server:

```bash
aws ec2 describe-addresses \
  --profile ${AWS_PROFILE} \
  --filters "Name=tag:Name,Values=${CANOPY_PROJECT_NAME}-sftp-${CANOPY_ENV}" \
  --query 'Addresses[0].PublicIp' \
  --output text \
  --no-cli-pager
# e.g. 54.210.167.43
```

**Using an SFTP desktop app** (e.g. FileZilla):

| Field | Value |
|---|---|
| Host | `54.210.167.43` (public IP only — no `sftp://`, no username prefix) |
| Username | `${CANOPY_SFTP_USERNAME}` |
| Password | password set in Step 25c |
| Port | `22` |

**Using the command line:**

```bash
sftp -P 22 ${CANOPY_SFTP_USERNAME}@<PUBLIC_IP>
# Enter the password set in Step 25c

pwd
# Expected: /{project}-sftp-{id}-{env}/{env}/{CANOPY_SFTP_USERNAME}

put myfiles.zip
exit
```

---

### Step 26: Build and Deploy Services
**Time:** 50 minutes (for all 7 services)

**Now that all AWS infrastructure is deployed, build and deploy your application services to ECS.**

#### Prerequisites Check

Before proceeding, ensure:
- ✅ All CloudFormation stacks are complete
- ✅ ECS cluster is running (Step 21)
- ✅ ECR repositories exist (Step 15)
- ✅ Docker is installed and running
- ✅ Maven is installed (for backend services)

#### Step 26a: Configure UI Environment Variables

The UI build requires environment-specific values baked into the Next.js bundle. Set these up **before** running `deploy.py`.

```bash
cd ${CANOPY_HOME}/datahub-ui-main

# Copy the example file and fill in your values
cp .env.example .env.local
```

Edit `.env.local`:

```bash
# Required — your load balancer URL from Step 9
NEXT_PUBLIC_DEV_URL=https://<your-alb>.us-east-1.elb.amazonaws.com

# Optional — Google Analytics Measurement ID (leave empty to disable)
# See GOOGLE_ANALYTICS_SETUP.md for how to obtain this
NEXT_PUBLIC_GTAG=G-XXXXXXXXXX

# Keep as 1 for production (0 only for local dev with self-signed certs)
NODE_TLS_REJECT_UNAUTHORIZED=1
```

#### Step 26b: Install Python Dependencies

```bash
cd ${CANOPY_HOME}/datahub-deployment-scripts

# Install boto3 for deploy.py script
pip install boto3

# Or use requirements file
pip install -r requirements-deploy.txt
```

#### Step 26c: Deploy All Services

The `deploy.py` script automates the entire build and deployment process. For the UI service it automatically reads `.env.local` and passes the values as Docker build args — no extra flags needed.

```bash
cd ${CANOPY_HOME}/datahub-deployment-scripts

# Usage: python deploy.py <project-name> <service-name> <environment> [image-tag]
# project-name, service-name and environment are required; image-tag defaults to 'latest'.

# Deploy each service (one at a time)
python deploy.py ${CANOPY_PROJECT_NAME} user-service ${CANOPY_ENV}
python deploy.py ${CANOPY_PROJECT_NAME} submission-service ${CANOPY_ENV}
python deploy.py ${CANOPY_PROJECT_NAME} report-service ${CANOPY_ENV}
python deploy.py ${CANOPY_PROJECT_NAME} download-service ${CANOPY_ENV}
python deploy.py ${CANOPY_PROJECT_NAME} entity-service ${CANOPY_ENV}
python deploy.py ${CANOPY_PROJECT_NAME} search-service ${CANOPY_ENV}
python deploy.py ${CANOPY_PROJECT_NAME} ui ${CANOPY_ENV}     
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

**Common Error** : Docker build/push failed. Please try again. 

#### Progress Tracking

You can check deployment progress in AWS Console:
- **ECS Console** → Clusters → `${CANOPY_PROJECT_NAME}-Services-{CANOPY_ENV}` → Services
- Watch for "Running count" to reach "Desired count" (1/1)

✅ **Verify:** Check all services are deployed
```bash
aws ecs describe-services \
  --cluster ${CANOPY_PROJECT_NAME}-Services-${CANOPY_ENV} \
  --services $(aws ecs list-services --cluster ${CANOPY_PROJECT_NAME}-Services-${CANOPY_ENV} --query 'serviceArns' --output text) \
  --query 'services[*].{Name:serviceName,Running:runningCount,Desired:desiredCount,Status:status}' \
  --output table
```

**Expected output:** All services should show `Running: 1, Desired: 1, Status: ACTIVE`


## Post-Deployment Verification

### Step 27: Verify Deployment
**Time:** 5 minutes

Check that all stacks deployed successfully:

```bash
# List all ${PROJECT_NAME} stacks
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE \
  --query "sort_by(StackSummaries[?contains(StackName, \`${CANOPY_PROJECT_NAME}\`)], &StackName)[].{Name:StackName,Status:StackStatus,Created:CreationTime}" \
  --output table \
  --no-cli-pager
```

**Expected:** All stacks should show `CREATE_COMPLETE` or `UPDATE_COMPLETE`

**Stacks you should see (14 total):**
1. ${CANOPY_PROJECT_NAME}-CloudWatch-{CANOPY_ENV}
2. ${CANOPY_PROJECT_NAME}-ECR-{CANOPY_ENV}
3. ${CANOPY_PROJECT_NAME}-ECS-{CANOPY_ENV}
4. ${CANOPY_PROJECT_NAME}-EventBridge-{CANOPY_ENV}
5. ${CANOPY_PROJECT_NAME}-Keycloak-{CANOPY_ENV}
6. ${CANOPY_PROJECT_NAME}-Lambda-{CANOPY_ENV}
7. ${CANOPY_PROJECT_NAME}-LoadBalancer-{CANOPY_ENV}
8. ${CANOPY_PROJECT_NAME}-Networking-{CANOPY_ENV}
9. ${CANOPY_PROJECT_NAME}-OpenSearch-{CANOPY_ENV}
10. ${CANOPY_PROJECT_NAME}-RDS-{CANOPY_ENV}
11. ${CANOPY_PROJECT_NAME}-Route53-{CANOPY_ENV} (optional)
12. ${CANOPY_PROJECT_NAME}-S3-{CANOPY_ENV}
13. ${CANOPY_PROJECT_NAME}-SES-{CANOPY_ENV}
14. ${CANOPY_PROJECT_NAME}-SQS-{CANOPY_ENV}
15. ${CANOPY_PROJECT_NAME}-SecretsManager-{CANOPY_ENV}
16. ${CANOPY_PROJECT_NAME}-TransferFamily-{CANOPY_ENV}

**Services you should see running (8 total):**
```bash
# Check ECS services
aws ecs list-services \
  --cluster ${CANOPY_PROJECT_NAME}-Services-${CANOPY_ENV} \
  --query 'sort(serviceArns)' \
  --output table \
  --no-cli-pager
```

Expected services:
1. ${CANOPY_PROJECT_NAME}-DownloadService
2. ${CANOPY_PROJECT_NAME}-EntityService
3. ${CANOPY_PROJECT_NAME}-Keycloak
4. ${CANOPY_PROJECT_NAME}-ReportService
5. {CANOPY_PROJECT_NAME}-Search
6. ${CANOPY_PROJECT_NAME}-SubmissionService
7. ${CANOPY_PROJECT_NAME}-UI
8. ${CANOPY_PROJECT_NAME}-UserService

---

### Step 28: Test Application

Test that the application is accessible and functioning:

```bash
# Get ALB DNS name
ALB_DNS=$(aws elbv2 describe-load-balancers \
  --query "LoadBalancers[?contains(LoadBalancerName, \`${CANOPY_PROJECT_NAME}\`)].DNSName" \
  --output text)

echo "Application URL: http://$ALB_DNS"

# Test frontend
curl -I http://$ALB_DNS

# Test API endpoints
curl http://$ALB_DNS/api/entity/v1/actuator/health
curl http://$ALB_DNS/api/search/v1/actuator/health
curl http://$ALB_DNS/api/user/v1/actuator/health
curl http://$ALB_DNS/api/submission-service/v1/actuator/health
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
  --stack-name ${CANOPY_PROJECT_NAME}-${CANOPY_ENV}-{STACK-NAME} \
  --query 'StackEvents[?ResourceStatus==`CREATE_FAILED`]'
```

#### S3 Bucket Already Exists

**Error:** `BucketAlreadyExists` or `BucketAlreadyOwnedByYou`

**Solution:** Go back to Step 8a and choose a different `DeploymentId`.

#### ECS Service Fails to Start

**Symptoms:** ECS service shows 0 running tasks

**Check:**
1. Docker images are pushed to ECR
2. Task logs in CloudWatch Logs
3. Security groups allow traffic
4. Secrets Manager secret exists and is accessible

```bash
# Check task logs
aws logs tail /aws/ecs/${CANOPY_PROJECT_NAME}-microservices-${CANOPY_ENV} \
  --log-stream-name-prefix entity-service \
  --follow \
  --profile ${AWS_PROFILE}
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
aws logs tail /aws/lambda/${CANOPY_PROJECT_NAME}-OpenSearchRefresh-${CANOPY_ENV} --follow
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
psql -h $RDS_ENDPOINT -U datahub_user -d ${CANOPY_PROJECT_NAME}_${CANOPY_ENV}
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
aws opensearch describe-domain --domain-name ${CANOPY_PROJECT_NAME}-opensearch-${CANOPY_ENV} \
  --query 'DomainStatus.{Status:Processing,Health:DomainEndpointOptions.EnforceHTTPS}'
```

### Getting Help

1. **Check CloudWatch Logs** - Most errors are logged
2. **Review CloudFormation Events** - Shows why stacks failed
3. **AWS Console** - Visual inspection of resources
4. **Documentation** - Check service-specific READMEs:
   - [RDS Deployment](https://github.com/canopy-datahub/datahub-development/blob/feature/aws/db/postgres/db-create-scripts/README_RDS_DEPLOYMENT.md)
   - [Lambda Deployment](https://github.com/canopy-datahub/datahub-development/blob/feature/aws/opensearch/opensearch_reindex/README_OPENSEARCH_REINDEX_LAMBDA_DEPLOYMENT.md)

---

## Cleanup

**⚠️ WARNING:** This will delete ALL resources and data. Make sure you have backups!

### Step 1: Empty Containers

Before deleting the stack, empty S3 buckets, RDS, and ECR so CloudFormation can remove these resources.

#### Step 1a: Empty S3 Buckets

S3 buckets with content cannot be deleted by CloudFormation:

```bash
# Match all buckets for this project/uniqueId/env
pattern="^${CANOPY_PROJECT_NAME}-.*-${CANOPY_DEPLOYMENT_ID}-${CANOPY_ENV}$"

# List matching buckets
aws s3 ls | awk '{print $3}' | grep -E "$pattern"

# Empty each bucket
aws s3 ls | awk '{print $3}' | grep -E "$pattern" | while read -r bucket; do
  echo "Emptying bucket: $bucket"
  aws s3 rm "s3://$bucket" --recursive
done
```

#### Step 1b: Empty RDS

Disable deletion protection on the RDS instance so the stack can delete it. If your RDS instance has a final snapshot or other retention settings that block deletion, adjust, or remove them.

```bash
# List RDS instances for this project
aws rds describe-db-instances --query "DBInstances[?contains(DBInstanceIdentifier, '${CANOPY_PROJECT_NAME}')].DBInstanceIdentifier" --output text

# Disable deletion protection (replace INSTANCE_ID with your RDS instance identifier)
aws rds modify-db-instance \
  --db-instance-identifier ${CANOPY_PROJECT_NAME}-${CANOPY_DEPLOYMENT_ID}-${CANOPY_ENV}-postgres \
  --no-deletion-protection \
  --apply-immediately
```

If the stack still cannot delete RDS (e.g., due to automated snapshots), you may need to delete the DB instance manually first, then delete the stack.

#### Step 1c: Empty ECR

ECR repositories with images cannot be deleted by CloudFormation. Delete all images in each repository:

```bash
# List all DataHub ECR repositories
aws ecr describe-repositories --query "repositories[?contains(repositoryName, '${CANOPY_PROJECT_NAME}')].repositoryName" --output text

for repo in $(aws ecr describe-repositories --query "repositories[?contains(repositoryName, '${CANOPY_PROJECT_NAME}')].repositoryName" --output text); do
  echo "Emptying ECR repository: $repo"
  aws ecr batch-delete-image --repository-name $repo --image-ids $(aws ecr list-images --repository-name $repo --query 'imageIds[*]' --output json) 2>/dev/null || true
done
```

### Step 2: Delete Stacks in Reverse Order

```bash
# Delete stacks in reverse dependency order
stacks=(
  "${CANOPY_PROJECT_NAME}-EventBridge-${CANOPY_ENV}"
  "${CANOPY_PROJECT_NAME}-Lambda-${CANOPY_ENV}"
  "${CANOPY_PROJECT_NAME}-SES-${CANOPY_ENV}"
  "${CANOPY_PROJECT_NAME}-ECS-${CANOPY_ENV}"
  "${CANOPY_PROJECT_NAME}-SecretsManager-${CANOPY_ENV}"
  "${CANOPY_PROJECT_NAME}-OpenSearch-${CANOPY_ENV}"
  "${CANOPY_PROJECT_NAME}-ECR-${CANOPY_ENV}"
  "${CANOPY_PROJECT_NAME}-SQS-${CANOPY_ENV}"
  "${CANOPY_PROJECT_NAME}-CloudWatch-${CANOPY_ENV}"
  "${CANOPY_PROJECT_NAME}-Route53-${ENCANOPY_ENVV}"
  "${CANOPY_PROJECT_NAME}-Keycloak-${CANOPY_ENV}"
  "${CANOPY_PROJECT_NAME}-RDS-${CANOPY_ENV}"
  "${CANOPY_PROJECT_NAME}-LoadBalancer-${CANOPY_ENV}"
  "${CANOPY_PROJECT_NAME}-S3-${CANOPY_ENV}"
  "${CANOPY_PROJECT_NAME}-Networking-${CANOPY_ENV}"
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
  --query "StackSummaries[?contains(StackName, \`${CANOPY_PROJECT_NAME}\`) && StackStatus != \`DELETE_COMPLETE\`].{Name:StackName,Status:StackStatus}" \
  --output table

# Check for remaining S3 buckets
aws s3 ls | grep ${CANOPY_PROJECT_NAME}

# Check for remaining ECR repositories
aws ecr describe-repositories --query "repositories[?contains(repositoryName, \`${CANOPY_PROJECT_NAME}\`)].repositoryName"
```

---

## Summary

🎉 **Congratulations!** You've successfully deployed DataHub to AWS!

**What you've deployed:**
- ✅ Complete VPC network infrastructure
- ✅ PostgreSQL RDS database with full schema
- ✅ OpenSearch domain for search functionality
- ✅ Keycloak identity provider running on ECS
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

**Document Version:** 1.1  
**Last Updated:** March 19, 2026 
**Maintained by:** Stanford Canopy Team

