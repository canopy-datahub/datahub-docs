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
- [ ] **Step 9:** [Request SSL/TLS Certificate (ACM)](#step-9-request-ssltls-certificate-acm) (10 min)
  - [ ] **Step 9a:** [Request a Certificate in ACM](#step-9a-request-a-certificate-in-acm)
  - [ ] **Step 9b:** [Add the ACM Validation CNAME to DNS](#step-9b-add-the-acm-validation-cname-to-dns)
  - [ ] **Step 9c:** [Configure the CertificateId Parameter](#step-9c-configure-the-certificateid-parameter)
- [ ] **Step 10:** [Deploy LoadBalancer Stack](#step-10-deploy-loadbalancer-stack) (5 min)
- [ ] **Step 11:** [Deploy RDS Stack](#step-11-deploy-rds-stack) (15 min)
  - [ ] **Step 11a:** [Configure RDS Security Group](#step-11a-configure-rds-security-group)
  - [ ] **Step 11b:** [Set RDS Master Password](#step-11b-set-rds-master-password)
  - [ ] **Step 11c:** [Deploy RDS Stack](#step-11c-deploy-rds-stack)
- [ ] **Step 12:** [Database Configuration](#step-12-database-configuration)
  - [ ] **Step 12a:** [Setup Database Schema](#step-12a-setup-database-schema) (30 min)
  - [ ] **Step 12b:** [Configure pg_cron Scheduler](#step-12b-configure-pg_cron-scheduler-optional) (Optional, 10 min)
- [ ] **Step 13:** [Deploy Route53 Stack](#step-13-deploy-route53-stack-optional) (Optional) (5 min)
- [ ] **Step 14:** [Deploy CloudWatch Stack](#step-14-deploy-cloudwatch-stack) (5 min)
- [ ] **Step 15:** [Deploy SQS Stack](#step-15-deploy-sqs-stack) (5 min)
- [ ] **Step 16:** [Deploy ECR Stack](#step-16-deploy-ecr-stack) (5 min)
- [ ] **Step 17:** [Deploy OpenSearch Stack](#step-17-deploy-opensearch-stack) (20 min)
  - [ ] **Step 17a:** [Configure OpenSearch Credentials](#step-17a-configure-opensearch-credentials)
  - [ ] **Step 17b:** [Deploy OpenSearch Stack](#step-17b-deploy-opensearch-stack)
- [ ] **Step 18:** [Deploy SecretsManager Stack](#step-18-deploy-secretsmanager-stack) (5 min)
- [ ] **Step 19:** [OpenSearch Reindex Lambda](#step-19-opensearch-reindex-lambda)
  - [ ] **Step 19a:** [Create Lambda Layer](#step-19a-create-lambda-layer) (5 min)
  - [ ] **Step 19b:** [Upload OpenSearch Reindex Lambda Code](#step-19b-upload-opensearch-reindex-lambda-code) (5 min)
- [ ] **Step 20:** [Upload Email Service Lambda Code](#step-20-upload-email-service-lambda-code) (5 min)
- [ ] **Step 21:** [Deploy Lambda Stack](#step-21-deploy-lambda-stack) (10 min)
- [ ] **Step 22:** [Deploy ECS Stack](#step-22-deploy-ecs-stack) (20 min)
- [ ] **Step 23:** [Deploy Keycloak](#step-23-deploy-keycloak) (30 min)
  - [ ] **Step 23a:** [Build and Push Keycloak Image](#step-23a-build-and-push-keycloak-image) (5 min)
  - [ ] **Step 23b:** [Deploy Keycloak ECS Stack](#step-23b-deploy-keycloak-ecs-stack) (5 min)
  - [ ] **Step 23c:** [Verify the Auto-Imported Realm](#step-23c-verify-the-auto-imported-realm) (2 min)
- [ ] **Step 24:** [Deploy SES Stack](#step-24-deploy-ses-stack) (20 min)
  - [ ] **Step 24a:** [Deploy SES Stack](#step-24a-deploy-ses-stack) (10 min)
  - [ ] **Step 24b:** [Configure SES Email Verification](#step-24b-configure-ses-email-verification) (10 min)
- [ ] **Step 25:** [Deploy EventBridge Stack](#step-25-deploy-eventbridge-stack) (5 min)
- [ ] **Step 26:** [Deploy TransferFamily Stack](#step-26-deploy-transferfamily-stack-sftp-server) (10 min)
  - [ ] **Step 26a:** [Allocate Elastic IP](#step-26a-optional-allocate-elastic-ip) (Optional)
  - [ ] **Step 26b:** [Deploy TransferFamily Stack](#step-26b-deploy-transferfamily-stack)
  - [ ] **Step 26c:** [Create SFTP User Account](#step-26c-create-an-sftp-user-account) (repeat per submitter)
  - [ ] **Step 26d:** [Register SFTP User in Database](#step-26d-register-the-sftp-user-in-the-database--required) (repeat per submitter)
- [ ] **Step 27:** [Build and Deploy Services](#step-27-build-and-deploy-services) (50 min)
  - [ ] **Step 27a:** [Configure UI Environment Variables](#step-27a-configure-ui-environment-variables)
  - [ ] **Step 27b:** [Deploy All Services](#step-27b-deploy-all-services)

### Post-Deployment (10 min)
- [ ] **Step 28:** [Verify Deployment](#step-28-verify-deployment)
- [ ] **Step 29:** [Test Application](#step-29-test-application)

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
canopycli init cli
```

✅ **Verify:** As a result, these 3 files will show up in your Canopy home directory:
```bash
aws-parameters-${CANOPY_ENV}-${USERNAME}.json
canopy-profile-native-develop.sh
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

- **`AWSServiceRoleForAmazonOpenSearchService`** — the IAM service-linked role Amazon OpenSearch Service needs in order to manage VPC network interfaces on your behalf. Without this role, the OpenSearch stack (Step 17) will fail with an `Invalid request` error on fresh AWS accounts.

**Why a separate stack:**

- The service-linked role is **account-wide** (shared by every IAM user and every project in the account), so it does not belong inside a per-environment stack.

**How it fits the overall deployment:**

The rest of the deployment is split into per-environment CloudFormation stacks under `canopy-cloud-replication/modules/` (`Networking.yaml`, `S3.yaml`, `LoadBalancer.yaml`, `RDS.yaml`, `OpenSearch.yaml`, `SecretsManager.yaml`, `Lambda.yaml`, `ECS.yaml`, `ECS-Keycloak.yaml`, `SES.yaml`, `EventBridge.yaml`, `TransferFamily.yaml`, …). Each of those stacks follows the same pattern:

- Named `${CANOPY_PROJECT_NAME}-<Module>-${CANOPY_ENV}` and tagged with `projectname` + `environment`.
- Configured from the shared parameter file `${CANOPY_AWS_PARAMETER_FILE}` generated by `canopycli init cli`.
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

### Step 9: Request SSL/TLS Certificate (ACM)
**Time:** 10 minutes (plus a few minutes for DNS validation)

Before deploying the Application Load Balancer, you must have a validated public ACM certificate for the domain that will serve Canopy. The ALB's HTTPS listener references this certificate by ARN, so the certificate **must exist and be issued before Step 10**.

#### Why HTTPS (and therefore this certificate) is required

- **Keycloak mandates HTTPS.** In production mode Keycloak refuses to issue tokens or accept logins over plain HTTP. Any authentication flow that reaches Keycloak over `http://` will fail with SSL/redirect errors, breaking the entire login experience.
- **User data must not travel over HTTP.** Authentication tokens, session cookies, personal details, and submission payloads all flow through the ALB. HTTP provides no confidentiality and no integrity protection — anyone on the network path could read or tamper with these. TLS termination at the ALB is the minimum bar for protecting user data.
- **Browser policy.** Modern browsers block mixed content and refuse to persist `Secure` cookies over HTTP, which would break both the UI and Keycloak session handling even if the backend tolerated it.

For these reasons, the ALB **must** terminate HTTPS using an ACM-issued certificate, and HTTP/80 must 301-redirect to HTTPS/443. The LoadBalancer stack deployed in Step 10 is already configured to do this — but only if a valid `CertificateId` is provided, which is what this step produces.

#### Step 9a: Request a Certificate in ACM

1. Open the [AWS Certificate Manager console](https://console.aws.amazon.com/acm/home?region=us-east-1) **in the same region as the ELB**.
2. Click **Request a certificate** → choose **Public certificate**.
3. Enter the domain name you will use to serve Canopy: `<YOUR-SUBDOMAIN-NAME>.<YOUR-DOMAIN-NAME>` (for example, `canopy-dev.yourdomain.com`).
4. Choose **DNS validation**. ACM will produce a CNAME record (a name and a value) that you must add to your DNS to prove domain ownership.

#### Step 9b: Add the ACM Validation CNAME to DNS

1. In the DNS manager for `<YOUR-DOMAIN-NAME>`, create a CNAME record using the name and value ACM provided. It will look something like:
   - **Hostname:** `_abcdef1234.canopy-dev` - You will probably need to leave out the <YOUR-DOMAIN-NAME> part from the end of the name.
   - **Alias to:** `_xyz789.acm-validations.aws`
2. Wait a few minutes — ACM will detect the record and flip the certificate status to **Issued**.
3. Copy the certificate ARN (e.g., `arn:aws:acm:${AWS::Region}:${AWS::AccountId}:certificate/<CERT-UUID>`).

✅ **Verify:** the certificate status in the ACM console must show **Issued** before continuing. A status of *Pending validation* means DNS has not yet propagated — wait and refresh.

#### Step 9c: Configure the CertificateId and PublicHostname Parameters

Add the following two values to `aws-parameters-${CANOPY_ENV}-{USERNAME}.json`:

```json
"CertificateId": "<CERT-UUID>",
"PublicHostname": "https://<YOUR-SUBDOMAIN-NAME>.<YOUR-DOMAIN-NAME>"
```

- **`CertificateId`** — the UUID segment of the certificate ARN (not the full ARN). `LoadBalancer.yaml` reads this and attaches the ACM certificate to the HTTPS listener that Step 10 creates. Without it, the LoadBalancer stack deploy will fail.
- **`PublicHostname`** — the full public HTTPS URL the users will access Canopy at (same domain the certificate was issued for, with `https://` scheme). It is consumed by two stacks later in the deployment:
  - `ECS-Keycloak.yaml` passes it as `KC_HOSTNAME`, so Keycloak advertises this URL in its OIDC discovery document and uses it for redirects.
  - `SecretsManager.yaml` uses it to populate `HostURL`, `DATAHUB_KEYCLOAK_ISSUER_URI`, and `DATAHUB_KEYCLOAK_JWK_SET_URI` in the application secret — these must match Keycloak's advertised hostname exactly, otherwise token validation and JWKS fetches will fail.

> **Note — pointing the domain at the ALB happens *after* Step 10.** The ALB DNS name does not exist until the LoadBalancer stack has been deployed, so the `<YOUR-SUBDOMAIN-NAME>` → `<ALB-DOMAIN-NAME>` CNAME is added as a post-deployment task at the end of Step 10.

---

### Step 10: Deploy LoadBalancer Stack
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

> You don't need to save this ALB DNS name anywhere — downstream stacks (`SecretsManager.yaml`, `ECS-Keycloak.yaml`) consume `PublicHostname` (configured in Step 9c), not the raw ALB DNS. The ALB DNS is only used for the DNS CNAME below.

#### Post-deployment: Point the Domain at the ALB

Now that the ALB exists, finish the HTTPS wiring started in Step 9 by pointing your domain at it. In the DNS manager for `<YOUR-DOMAIN-NAME>`, add a CNAME record:

| Field        | Value                   |
|--------------|-------------------------|
| **Type**     | CNAME                   |
| **Hostname** | `<YOUR-SUBDOMAIN-NAME>` |
| **Alias to** | `<ALB-DOMAIN-NAME>`     |
| **TTL**      | Default                 |

This makes `<YOUR-SUBDOMAIN-NAME>.<YOUR-DOMAIN-NAME>` resolve to the load balancer. Because the ACM certificate from Step 9 covers exactly this domain, HTTPS will work as soon as DNS propagates.

✅ **Verify** after DNS propagates (usually a few minutes). At this point the ALB exists but no backend services are registered yet (the UI and backend don't deploy until Step 27), so a working HTTPS setup is signalled by the TLS handshake succeeding — a `503 Service Unavailable` from the ALB is expected and correct:

```bash
# TLS handshake must succeed (cert valid for this domain). Expected status: HTTP/2 503 from awselb/2.0
curl -I https://<YOUR-SUBDOMAIN-NAME>.<YOUR-DOMAIN-NAME>

# Should 301 redirect to HTTPS
curl -I http://<YOUR-SUBDOMAIN-NAME>.<YOUR-DOMAIN-NAME>

# Explicit cert-chain check (no 503 noise — just verifies the cert is trusted and matches the domain)
curl -Iv https://<YOUR-SUBDOMAIN-NAME>.<YOUR-DOMAIN-NAME> 2>&1 | grep -E "subject:|issuer:|SSL certificate verify"
```

If you see a TLS error (e.g. `SSL certificate problem: unable to get local issuer certificate` or `certificate verify failed`), the ACM certificate is not attached correctly — revisit Step 9c (`CertificateId` parameter) and re-deploy the LoadBalancer stack. A real end-to-end HTTPS check against the UI service happens in Step 28.

---

### Step 11: Deploy RDS Stack
**Time:** 15 minutes

Creates RDS PostgreSQL database instance.

#### Step 11a. Configure RDS Security Group

Before deploying, get your public IP and save it as `MyIP` in `parameters-${CANOPY_ENV}.json`. `RDS.yaml` reads this value as a CloudFormation parameter and adds it to the RDS security group, so no manual file editing is required.

```bash
# Get your public IP
curl -4 ifconfig.me
```

Then set the value in `aws-parameters-${CANOPY_ENV}-${USERNAME}.json`:

```json
"MyIP": "REPLACEME/32"
```

> Replace the IP above with the one returned by the `curl` command. The `/32` suffix is required.

#### Step 11b. Set RDS Master Password and Keycloak DB Credentials

The parameter file ships with placeholder passwords. **Set them before deploying** — CloudFormation passes them to `RDS.yaml` / `SecretsManager.yaml` via `--parameter-overrides`, and `deploy_to_rds.py` in Step 12a reads them to create the corresponding roles.

In `aws-parameters-${CANOPY_ENV}-${USERNAME}.json`, replace the placeholders:

```json
"DbMasterPassword": "REPLACEME",
"CanopyAppDbName": "canopy_${CANOPY_ENV}",
"KeycloakDbName": "canopy_keycloak_${CANOPY_ENV}",
"KeycloakDbUsername": "keycloak_user",
"KeycloakDbPassword": "REPLACEME_kc_db_password",
"KeycloakAdminUsername": "administrator",
"KeycloakAdminPassword": "REPLACEME_kc_admin_password"
```

- **`DbMasterPassword`** — RDS master user password, used to bootstrap the cluster and run the schema-init SQL scripts.
- **`CanopyAppDbName`** — name of the initial application database RDS creates at provision time. `RDS.yaml` passes it as `DBName`, and `SecretsManager.yaml` writes it into the application secret's `dbname` field. Kept as an explicit parameter for symmetry with `KeycloakDbName`.
- **`KeycloakDbName`** / **`KeycloakDbUsername`** / **`KeycloakDbPassword`** — Keycloak gets its own PostgreSQL database and role so its tables are not interwoven with the application schema. These values are consumed by `06_create_keycloak_db.sql` (via Step 12a) and by `SecretsManager.yaml` (so the ECS-Keycloak service can authenticate to the isolated DB).
- **`KeycloakAdminUsername`** / **`KeycloakAdminPassword`** — bootstrap credentials for the Keycloak **master** realm admin console (`${PublicHostname}/admin/master/console/`). Keycloak only reads these on the first container boot against an empty DB to seed the initial admin user — changing them later does not rotate the credentials (use the admin console or `kcadm.sh` for that). Pick values you're willing to commit to the param file.

> ⚠️ Use a different value for `KeycloakDbPassword` than `DbMasterPassword`. That separation is the reason Keycloak has its own role in the first place.

#### Step 11c. Deploy RDS Stack

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
"RDSEndpoint": "REPLACEME:....amazonaws.com"
```

> Replace the placeholder above with the actual endpoint shown at the end of Step 11b.
---

### Step 12: Database Configuration

---

### Step 12a: Set up Database Schema
**Time:** 30 minutes

This step creates the database schema, tables, views, and initial data in your RDS PostgreSQL instance.

#### 📘 Detailed Documentation

**See:** [README_RDS_DEPLOYMENT.md](https://github.com/canopy-datahub/canopy-development/blob/feature/aws/db/postgres/db-create-scripts/README_RDS_DEPLOYMENT.md)


#### Quick Setup (Automated)

🖥️ **Execute**:

```bash
canopycli aws rds deploy-schema
```

This wraps `canopy-development/db/postgres/db-create-scripts/deploy_to_rds.py` and passes `--project-name`, `--env`, `--region`, and `--profile` from your sourced `set-canopy-env.sh` (i.e. `${CANOPY_PROJECT_NAME}`, `${CANOPY_ENV}`, `${AWS_REGION}`, `${AWS_PROFILE}`). Pass `--dry-run` to print the underlying command without executing it.

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
   - `06_create_keycloak_db.sql` - Creates the isolated Keycloak database and role (reads `KeycloakDbName` / `KeycloakDbUsername` / `KeycloakDbPassword` from `${CANOPY_AWS_PARAMETER_FILE}`)
5. ✅ Provides next steps for Secrets Manager update

⏳ **Wait time:** ~5-10 minutes for all scripts to complete

✅ **Verify:** The script will confirm the successful deployment and show table counts.
---

### Step 12b: Configure pg_cron Scheduler *(Optional)*
**Time:** 10 minutes

Automates the weekly `hub_content_metrics` snapshot by scheduling `sp_generate_hub_content_metrics()` inside PostgreSQL. Skip this if you prefer to run the procedure manually.

📄 **See full setup guide:** [PGCRON_METRICS_SETUP.md](./PGCRON_METRICS_SETUP.md)

---

### Step 13: Deploy Route53 Stack (Optional)
**Time:** 5 minutes
💡 **Skip this step** if you don't have Route53 hosted zones configured.

Creates DNS records and routing configurations.

⚠️ **Note:** This stack is organization-dependent. Consult with your DNS administrator before deploying.

```bash
canopycli aws cloudformation deploy Route53
```

---

### Step 14: Deploy CloudWatch Stack
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

### Step 15: Deploy SQS Stack
**Time:** 5 minutes

Creates SQS queues for message processing.

```bash
canopycli aws cloudformation deploy SQS
```

---

### Step 16: Deploy ECR Stack
**Time:** 5 minutes

Creates ECR repositories for container images.

```bash
canopycli aws cloudformation deploy ECR
```

**What's created:**
- ECR repositories for each microservice (one per service × env):
  - `${CANOPY_PROJECT_NAME}-download-service/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-entityservice/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-keycloak/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-report-service/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-search/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-submission-service/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-ui/{CANOPY_ENV}`
  - `${CANOPY_PROJECT_NAME}-user-service/{CANOPY_ENV}`

✅ **Verify:** List ECR repositories
```bash
canopycli aws ecr list
```

---

### Step 17: Deploy OpenSearch Stack
**Time:** 20 minutes

Creates OpenSearch domain for search and analytics.

#### Step 17a. Configure OpenSearch Credentials

⚠️ **Security:** Change the default OpenSearch password before deployment!

Edit `aws-parameters-${CANOPY_ENV}-${USERNAME}.json` and update:
```json
{
  "OpenSearchUsername": "opensearch",
  "OpenSearchPassword": "REPLACEME"
}
```

**Password requirements:**
- Minimum 8 characters
- Must contain uppercase, lowercase, number, and special character

#### Step 17b. Deploy OpenSearch Stack

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

> Replace the placeholder above with the actual endpoint shown at the end of Step 17b.

⚠️ **Note:** Step 17b will display the OpenSearch endpoint. Save it in `parameters-${CANOPY_ENV}.json` as shown above — do **not** edit Secrets Manager directly.

---

### Step 18: Deploy SecretsManager Stack
**Time:** 5 minutes

Creates secrets for database credentials, API keys, and OpenSearch configuration.

⚠️ **Important:** Deploy this stack only after **RDS**, **Load Balancer**, and **OpenSearch** are deployed. The secret stores endpoints from those stacks:

- **host** — RDS endpoint (from Step 11)
- **SEARCH_HOST** — OpenSearch endpoint (from Step 17)
- **HostURL**, **DATAHUB_KEYCLOAK_ISSUER_URI**, **DATAHUB_KEYCLOAK_JWK_SET_URI** — all built from `PublicHostname` (configured in Step 9c). This must be the public HTTPS URL, not the raw ALB DNS, so that the values match the `iss` claim and JWKS endpoint Keycloak will advertise.
- **KC_DB_URL**, **KC_DB_USERNAME**, **KC_DB_PASSWORD** — pre-built JDBC URL + credentials for Keycloak's isolated database (the values configured in Step 11b and materialized by Step 12a's `06_create_keycloak_db.sql`). The ECS-Keycloak task reads these at start-up so Keycloak authenticates to its own DB, not the application DB.

Also update these email values for your environment before deployment:

- **SupportEmail** — Address used as the sender (From) for system emails and often in Cc (e.g., study registration, support requests). Use an address on a domain verified in SES (e.g. `*@stanford.edu` if that domain is verified).
- **StakeholderEmailsStudyReg** — Comma-separated list of additional recipients (Cc) for study registration and study approval emails (e.g., NIH officers). Leave empty or set as needed.

```bash
canopycli aws cloudformation deploy SecretsManager
```

✅ **Verify:** Check secret was created

```bash
canopycli aws secrets describe
```

---

### Step 19: OpenSearch Reindex Lambda 
📘 For detailed OpenSearch reindex lambda deployment documentation, see: [README_OPENSEARCH_REINDEX_LAMBDA_DEPLOYMENT.md](https://github.com/canopy-datahub/canopy-development/blob/feature/aws/opensearch/opensearch_reindex/README_OPENSEARCH_REINDEX_LAMBDA_DEPLOYMENT.md)

#### Step 19a: Create Lambda Layer
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

**Ensure Docker is running** (required for the ARM64 cross-compile step):

```bash
# macOS: open Docker Desktop if installed (or start it from Applications)
open -a Docker

# Wait for the whale icon in the menu bar, then verify the daemon is up:
docker info

# Build + publish the layer via canopycli
canopycli aws lambda create-layer
```

The default layer name is `dependency-layer`; override with `--name <other>`. Pass `--dry-run` to preview without running Docker or publishing.

**What this does:**
1. Runs `docker run --platform linux/arm64 public.ecr.aws/lambda/python:3.11-arm64` and pip-installs the pinned dependency set into a temp directory: `psycopg2-binary==2.9.9`, `opensearch-py`, `requests-aws4auth`, `boto3`, `requests`.
2. Verifies `psycopg2` ships an `aarch64` binary (warns loudly if it somehow doesn't).
3. Zips the result and calls `aws lambda publish-layer-version --compatible-runtimes python3.11 --compatible-architectures arm64`.
4. Prints the **Layer ARN** for the param file.

⏳ **Wait time:** 3–5 minutes (Docker image pull on first run, pip install, upload).

✅ **Verify:** the output panel shows the `Layer ARN`, e.g.:
```
Layer ARN:
  arn:aws:lambda:us-east-1:<acct>:layer:dependency-layer:1
```

#### Post-Operation: Update parameters file

Copy the ARN into `LambdaDependencyLayerArn` in `aws-parameters-${CANOPY_ENV}-${USERNAME}.json`, then redeploy the Lambda stack:

```json
"LambdaDependencyLayerArn": "arn:aws:lambda:us-east-1:<acct>:layer:dependency-layer:1"
```

```bash
canopycli aws cloudformation deploy Lambda
```

---

#### Step 19b: Upload OpenSearch Reindex Lambda Code
**Time:** 5 minutes

Package and upload the OpenSearch-reindex Lambda code + index mapping JSONs to the project's `lambda-artifacts` S3 bucket. Pass `--dry-run` to preview without building/uploading.

```bash
canopycli aws lambda deploy opensearch-reindex
```

**What this does:**
1. Zips the four files from `${CANOPY_HOME}/canopy-development/opensearch/opensearch_reindex/`:
   - `opensearch_reindex_aws.py`
   - `search_index_mapping.json`
   - `variable_index_mapping.json`
   - `autocomplete_index_mapping.json`
2. Uploads to `s3://${CANOPY_PROJECT_NAME}-lambda-artifacts-${DeploymentId}-${CANOPY_ENV}/opensearch-refresh/opensearch-refresh-lambda.zip`.
3. Prints a hint with the exact `aws lambda update-function-code` command for subsequent hot updates (only needed after the Lambda stack has been deployed at least once).

⚠️ **Note:** Dependencies are NOT included — they come from the layer built in Step 19a. A warning appears if the zip exceeds 1 MB (usually a sign the layer content leaked in).

✅ **Verify:** Check S3 upload
```bash
aws s3 ls s3://${CANOPY_PROJECT_NAME}-lambda-artifacts-${DeploymentId}-${CANOPY_ENV}/opensearch-refresh/
```
**Expected:** Should show `opensearch-refresh-lambda.zip`

### Step 20: Upload Email Service Lambda Code

Maven-build the `datahub-service-email` Spring Boot project and upload the resulting AWS-bundled jar to the `lambda-artifacts` bucket:

```bash
canopycli aws lambda deploy email-service
```

**What this does:**
1. Runs `mvn clean package -DskipTests -q` in `${CANOPY_HOME}/datahub-service-email`.
2. Picks the `*-aws.jar` artifact from `target/` (falling back to a plain `*.jar` if no `-aws` variant is produced).
3. Uploads to `s3://${CANOPY_PROJECT_NAME}-lambda-artifacts-${DeploymentId}-${CANOPY_ENV}/email-service/<jar-name>`.
4. Prints the `aws lambda update-function-code` command for subsequent updates.

✅ **Verify:** Check S3 upload
```bash
aws s3 ls s3://${CANOPY_PROJECT_NAME}-lambda-artifacts-${DeploymentId}-${CANOPY_ENV}/email-service/
```
**Expected:** A single `datahub-service-email-*.jar`.
✅ **Verify:** Check S3 upload
```bash
aws s3 ls s3://${CANOPY_PROJECT_NAME}-lambda-artifacts-${CANOPY_DEPLOYMENT_ID}-${CANOPY_ENV}/email-service/
```
**Expected:** Should show `datahub-service-email-0.0.1-SNAPSHOT-aws.jar`

---

### Step 21: Deploy Lambda Stack
**Time:** 10 minutes

Creates Lambda functions for OpenSearch indexing and email service.

```bash
canopycli aws cloudformation deploy Lambda
```

**What's created:**
- Lambda function: `${CANOPY_PROJECT_NAME}-OpenSearchRefresh`
  - Runtime: Python 3.11 on ARM64
  - VPC: Deployed in private subnets
  - Attached layer from Step 19a
  - Code from S3 (Step 19b)
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

### Step 22: Deploy ECS Stack
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

### Step 23: Deploy Keycloak
**Time:** 30 minutes

Deploys Keycloak as a dedicated ECS service for identity and access management.

#### Step 23a: Build and Push Keycloak Image
**Time:** 5 minutes

Build the Keycloak Docker image and push it to ECR **before** creating the ECS service. The ECS service in Step 23b references this image by URI; if the image isn't in ECR when the service starts, every task will fail with `CannotPullContainerError` and the ECS deployment circuit breaker will trip.

```bash
canopycli aws ecs deploy keycloak
```

For `keycloak` the `--tag` flag is optional — canopycli reads `KeycloakImageTag` from `aws-parameters-${CANOPY_ENV}-${USERNAME}.json` (currently `26.5.4`) and uses it as the image tag. Pass `--tag <other>` to override, or `--dry-run` to preview without building.

✅ **Verify:** the image is in ECR:

```bash
canopycli aws ecr list-images keycloak
```

Expected: `26.5.4` (and possibly `latest`). Run without the `keycloak` argument to list tags across **all** project ECR repos.

---

#### Step 23b: Deploy Keycloak ECS Stack
**Time:** 5 minutes

Deploy the Keycloak-specific ECS service via CloudFormation. Run this **after** Step 23a has pushed the image to ECR:

```bash
canopycli aws cloudformation deploy ECS-Keycloak
```

> **Database isolation:** Keycloak authenticates to its own PostgreSQL database (configured in Step 11b, created by `06_create_keycloak_db.sql` in Step 12a, and surfaced to the container via the `KC_DB_URL` / `KC_DB_USERNAME` / `KC_DB_PASSWORD` keys in the application secret).
> The application database and Keycloak's internal state are fully separated.

---

#### Step 23c: Verify the Auto-Imported Realm
**Time:** 2 minutes

The `canopy-keycloak` image ships with the `CANOPY` realm baked in (with hostnames sanitized to a sentinel that the container's entrypoint substitutes with `${KC_HOSTNAME}` at start-up). On first boot, `kc.sh start --optimized --import-realm` creates the realm, `canopy-client` public client, realm roles, and the two test users. On subsequent boots, `--import-realm` is idempotent — it sees the realm already exists and skips.

Open the Keycloak master-realm admin console in your default browser:

```bash
canopycli aws open keycloak-admin
```

This resolves to `${PublicHostname}/admin/master/console/` (e.g. `https://canopy-prod.example.com/admin/master/console/`). Log in with the bootstrap admin credentials — `${KeycloakAdminUsername}` / `${KeycloakAdminPassword}` values from `aws-parameters-${CANOPY_ENV}-${USERNAME}.json`.

**Verify:**

| Check | Where | Expected |
|---|---|---|
| Realm exists | Top-left realm dropdown | `CANOPY` is listed alongside `master` |
| Client exists | `CANOPY` → Clients | `canopy-client` is present, **Client authentication: Off** (public client) |
| Redirect URIs rendered | `canopy-client` → Settings → Valid redirect URIs | Starts with `${PublicHostname}/...`, **not** `@@PUBLIC_HOSTNAME@@` and **not** `https://canopy.stanford.edu/*` |
| Test user exists | `CANOPY` → Users | `test@test.com` (password `Password123`, `Temporary: Off`, UUID `333e4567-e89b-12d3-a456-426614174002` — pinned to match `users.uuid` for `id=3` in the canopy DB) |
| Theme applied | `canopycli aws open keycloak-realm` (takes you to the CANOPY realm console) | The custom `canopy` login theme renders, not the default Keycloak one |

If redirect URIs still contain `@@PUBLIC_HOSTNAME@@`, the entrypoint substitution didn't run — check the ECS task logs for the `Rendered realm-import` line; if absent, verify `KC_HOSTNAME` is actually set in the task environment (it's derived from the `PublicHostname` stack parameter). If redirect URIs still contain `canopy.stanford.edu`, an unsanitized realm JSON got baked into the image — rebuild with Step 23b.

> **To refresh the realm** (e.g. after updating `CANOPY-realm.json`): delete the `CANOPY` realm in the admin console, then force a new ECS task deployment. Because `--import-realm` only skips **existing** realms, the fresh container will rebuild it from the image. The application DB (and its users in the `user_entity` table mirrored from the backend) is untouched.

---

### Step 24: Deploy SES Stack
**Time:** 10 minutes

#### Step 24a: Deploy SES Stack

Creates the SES verified identities the email-lambda relies on to send mail. Read this section end-to-end before running the deploy — the decision about `SenderDomain` depends on which DNS zones you control.

##### How Canopy uses SES (conceptual model)

Three pieces of email configuration flow through the deployment, and they serve **different purposes**. Conflating them is the #1 source of confusion.

| Param (in `aws-parameters-${CANOPY_ENV}-${USERNAME}.json`) | Role | Consumer | Needs to be verified? |
|---|---|---|---|
| **`SupportEmail`** | The **sender** — the `From:` address on every email the platform sends. | Email-lambda (via `supportEmail` key in the application secret). | **Yes** — either as an email identity (always created by this stack) OR under a verified domain identity. |
| **`SenderDomain`** *(optional)* | The **sender domain** — enables *any* `*@<SenderDomain>` to send without per-email verification. | SES. This stack creates a domain identity only if `SenderDomain` is non-empty. | **Yes, via DKIM DNS records.** See below for the DNS requirement. |
| **`StakeholderEmailsStudyReg`** | Comma-separated **recipients** CC'd on study-registration / approval emails (NIH officers, PMs, etc.). | Email-lambda. | **Only in sandbox mode**, and only if you want mail to actually be delivered to them. Not created by this stack — verified manually in the SES console if needed. See Step 24b. |

Key semantic split: **`SupportEmail` is a sender, `StakeholderEmailsStudyReg` is a recipient.** They may hold the same value in a dev setup — that's coincidental, not semantic.

##### Which identities need verification — and where you need DNS access

`SES.yaml` always creates an `AWS::SES::EmailIdentity` for `SupportEmail`, and — if `SenderDomain` is non-empty — a domain-level identity for `SenderDomain`. After `canopycli aws cloudformation deploy SES` the identities *exist* but are `Pending` until verified:

| Identity | How SES verifies it | What you need |
|---|---|---|
| **Email identity for `SupportEmail`** | Click-the-link: AWS sends a "Verify your email" message to `SupportEmail`. | An inbox that actually receives mail at that address (or any alias that routes there). |
| **Domain identity for `SenderDomain`** | DKIM: AWS generates 3 CNAME records; they must be present in the public DNS for `SenderDomain`. | **Write access to the DNS zone for `SenderDomain`.** |
| **Recipient identities** (sandbox mode only) | Click-the-link, one per address (via SES console). | An inbox for each recipient. |

> ⚠️ **Domain verification is NOT "just declare it and it works".**
> AWS needs DNS-control proof via the three DKIM CNAME records.
> Until those records resolve, the domain identity stays `PendingVerification` indefinitely.
> **Do not set `SenderDomain` to a domain whose DNS zone you cannot modify**.

##### Decision guide — what to set

| You own DNS for the domain of `SupportEmail`? | Recommendation |
|---|---|
| **Yes** (e.g. you're deploying to `support@your-org.example` and control `your-org.example` DNS) | Set `SenderDomain = "your-org.example"`. Verify once via DKIM; any future `*@your-org.example` sender works without extra config. Email identity for `SupportEmail` also gets created and will verify via its link if the mailbox receives mail — redundant belt-and-suspenders, harmless. |
| **No — but your sender address is in a domain someone else owns** (e.g. you want `From: you@big-university.edu` but you're not the DNS admin) | Leave `SenderDomain = ""`. Only the email identity for `SupportEmail` is created; it verifies by email link. **No DKIM dance, no DNS access needed.** Works. Downside: future senders from the same domain have to be verified individually via the console. |
| **Mismatched domains** (e.g. `SupportEmail=you@bigcorp.com`, but `SenderDomain=your-side-project.io`) | Don't do this. A domain identity for `your-side-project.io` does **not** cover `you@bigcorp.com` — the sender's domain and the verified domain must match. Either align them (change `SupportEmail` to be on `your-side-project.io`) or drop `SenderDomain` to `""`. |

For Canopy prod on `example.com` since you control `example.com` DNS, the coherent setup is:

```json
"SupportEmail":               "replaceme-support@example.com",
"SenderDomain":               "example.com",
"StakeholderEmailsStudyReg":  "<comma-separated recipients — sender domain independent>"
```

`replaceme-support@example.com` doesn't need to be a real mailbox for sending to work — the domain identity authorizes SES to send as any `*@example.com`.
If you want the per-email identity to *also* reach `Verified` status (nice-to-have for belt-and-suspenders), route `replaceme-support@example.com` somewhere you can click the link.

##### Deploy

```bash
canopycli aws cloudformation deploy SES
```

⚠️ **The identities exist as soon as the stack deploys, but SES will refuse to actually *send* from them until each reaches `Verified` status.** Proceed to Step 24b.

---

#### Step 24b: Verify the SES Identities
**Time:** 5 minutes (+ however long your DNS takes to propagate for `SenderDomain`)

SES's rule: a send is allowed if **either** the sender's email identity **or** the sender's domain identity is `Verified`. Pick whichever path fits your setup — you don't need both.

##### Option 1 (recommended): Verify `SenderDomain`

Verifying the domain covers every `*@${SenderDomain}` sender in one shot (including `SupportEmail`, if `SupportEmail` is under that domain). The email identity for `SupportEmail` is still created by the stack but doesn't need to reach `Verified` — domain verification is sufficient.

Domain identities are verified via DKIM DNS records (not email clicks). canopycli prints the three CNAMEs ready to paste into your DNS manager:

```bash
canopycli aws ses dkim
```

With no argument the command defaults to the `SenderDomain` value from your param file. Pass an explicit identity (`canopycli aws ses dkim example2.com`) to query a different one.

Example output:

```
           DKIM CNAME records for 'example.com' — add all three to your DNS zone
┏━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Type  ┃ Host                               ┃ Value                             ┃
┡━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ CNAME │ abc123…._domainkey.example.com     │ abc123….dkim.amazonses.com        │
│ CNAME │ def456…._domainkey.example.com     │ def456….dkim.amazonses.com        │
│ CNAME │ ghi789…._domainkey.example.com     │ ghi789….dkim.amazonses.com        │
└───────┴────────────────────────────────────┴───────────────────────────────────┘
DkimEnabled: True   DkimVerificationStatus: Pending
```

Steps:

1. Run `canopycli aws ses dkim` — copy the three rows.
2. Add all three as `CNAME` records in the DNS zone for `${SenderDomain}` (Route 53, Cloudflare, Namecheap — whichever hosts your zone). Heads-up: some DNS UIs automatically append the root domain to the host field; if so, enter the host as `abc123…._domainkey` rather than `abc123…._domainkey.example.com` to avoid doubling it up.
3. Wait for DNS to propagate (usually minutes, sometimes up to 72 hours).
4. Re-run `canopycli aws ses dkim` — when `DkimVerificationStatus` flips from `Pending` → `Success`, you're done. AWS auto-detects the records; nothing further to trigger.

You can now send from any `*@${SenderDomain}` — skip Option 2.

##### Option 2 (fallback): Verify `SupportEmail` individually

Use this path only when:
- `SenderDomain` is empty (you chose not to verify a domain), **or**
- `SupportEmail` is outside `SenderDomain` (e.g. `SupportEmail=you@stanford.edu` but `SenderDomain=your-side-project.io` — mismatched domains, which Step 24a warns against).

AWS automatically sends a "Verify your email" link to the `SupportEmail` address the moment the CFN identity is created. Open that inbox, click the link, and the status flips to `Verified`. Confirm:

```bash
canopycli aws ses verification
```

With no argument the command defaults to `SupportEmail` from your param file. Pass an explicit identity (`canopycli aws ses verification other@example.com`) to check a different one.

**Expected:** `VerificationStatus: Success`. `Pending` means the verify link hasn't been clicked yet.

##### Recipients — sandbox vs. production

- **Sandbox mode** (every new AWS account starts here): SES will also refuse to deliver to **unverified recipients**. That means every address in `StakeholderEmailsStudyReg`, plus any test-user email you want to receive mail, must be individually verified via the SES console → Verified identities → Create identity → email. This stack does **not** auto-verify recipients.
- **Production mode**: recipients do not need verification. To leave sandbox:
  1. AWS Console → SES → Account dashboard → **Request production access**.
  2. Fill out the form; AWS typically responds within 24–48 hours.

---

### Step 25: Deploy EventBridge Stack
**Time:** 5 minutes

Creates EventBridge rules to automatically trigger SQS processing when files are uploaded to S3 via SFTP.

```bash
canopycli aws cloudformation deploy EventBridge
```

**What's created:**
- EventBridge rule {ProjectName}-SFTP-{Environment} that monitors this project’s SFTP S3 bucket for file uploads (one rule per project when using the same account)
- Automatic routing of S3 events to SQS SFTP Queue

---

### Step 26: Deploy TransferFamily Stack (SFTP Server)
**Time:** 10 minutes

Creates an AWS Transfer Family SFTP server with custom Lambda-based authentication for secure file uploads.

#### Step 26a: (Optional) Allocate Elastic IP

The Transfer Family server uses `EndpointType: VPC`, which means it sits inside your VPC with no public DNS by default. Without an Elastic IP, the server has no stable public address — the `s-xxxx.server.transfer.amazonaws.com` hostname won't resolve from the internet, and any assigned IP is ephemeral and changes on redeploy.

An Elastic IP gives the server a **fixed public IP** that:
- Resolves the `s-xxxx.server.transfer.amazonaws.com` hostname publicly
- Survives stack redeployments unchanged
- Can be whitelisted by submitters whose institutions require firewall rules by IP

It is strongly recommended for any environment where external submitters will connect. Only skip it for short-lived dev testing where you connect from within the VPC.

For a stable, static SFTP endpoint address that doesn't change on redeploy:

```bash
canopycli aws ec2 allocate-eip
```

The command tags the EIP with `projectname=${CANOPY_PROJECT_NAME}`, `environment=${CANOPY_ENV}`, and `Name=${CANOPY_PROJECT_NAME}-sftp-${CANOPY_ENV}` so it's easy to find later.

##### Save the AllocationId into the param file

The TransferFamily stack in Step 26b reads the EIP through the `ElasticIPAllocationId` CFN parameter. Copy the `AllocationId` from the output (format: `eipalloc-xxxxxxxxxxxxxxxxx`) into `aws-parameters-${CANOPY_ENV}-${USERNAME}.json`:

```json
"ElasticIPAllocationId": "eipalloc-0abc1234def567890"
```

If the param stays at the placeholder value (`"REPLACEME:eipalloc-xxxxxx"`) or is left as the empty string `""`, `TransferFamily.yaml` will deploy the server **without** a fixed Elastic IP — the whole point of Step 26a. Make sure to update the param file before proceeding to Step 26b.

##### Re-query the EIP later

If you lose the `AllocationId` output or want to double-check which EIP is tagged for this project / environment:

```bash
canopycli aws ec2 describe-eip
```

Example output:

```
Elastic IPs tagged projectname='canopy', environment='prod'
┌─────────────────────┬────────────────────────────┬────────────────┬───────────────┐
│ Name                │ AllocationId               │ PublicIp       │ AssociationId │
├─────────────────────┼────────────────────────────┼────────────────┼───────────────┤
│ canopy-sftp-prod    │ eipalloc-0abc1234def567890 │ 54.10.20.30    │ None          │
└─────────────────────┴────────────────────────────┴────────────────┴───────────────┘
```

`AssociationId: None` means the EIP is allocated but not attached to anything yet — normal state between 26a and 26b. After Step 26b, re-running this will show the EIP associated with the Transfer Family server's ENI.

#### Step 26b: Deploy TransferFamily Stack

```bash
canopycli aws cloudformation deploy TransferFamily
```

The stack reads `ElasticIPAllocationId` from your param file:
- **Non-empty** (e.g. `eipalloc-0abc…`) — attaches the EIP to the Transfer Family server. The server comes up with a fixed, whitelist-able public IP.
- **Empty (`""`)** — no EIP is attached. The server still works but gets an ephemeral IP that changes on redeploy.

**What's created:**
- AWS Transfer Family SFTP server (VPC endpoint, public subnet AZ1)
- Lambda + API Gateway for custom Secrets Manager-based authentication
- `CustomSFTPTransferRole` IAM role for S3 access

✅ **Verify:** Get the SFTP server endpoint

```bash
canopycli aws transfer endpoint
# Expected format: s-1234567890abcdef0.server.transfer.us-east-1.amazonaws.com
```

✅ **Verify:** the Elastic IP is now associated with the server (if you set `ElasticIPAllocationId`):

```bash
canopycli aws ec2 describe-eip
# AssociationId should now be non-None — the EIP is attached to the server's ENI.
```

#### Step 26c: Create an SFTP User Account

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

#### Step 26d: Register the SFTP User in the Database ⚠️ Required

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
| Password | password set in Step 26c |
| Port | `22` |

**Using the command line:**

```bash
sftp -P 22 ${CANOPY_SFTP_USERNAME}@<PUBLIC_IP>
# Enter the password set in Step 26c

pwd
# Expected: /{project}-sftp-{id}-{env}/{env}/{CANOPY_SFTP_USERNAME}

put myfiles.zip
exit
```

---

### Step 27: Build and Deploy Services
**Time:** 50 minutes (for all 7 services)

**Now that all AWS infrastructure is deployed, build and deploy your application services to ECS.**

#### Prerequisites Check

Before proceeding, ensure:
- ✅ All CloudFormation stacks are complete
- ✅ ECS cluster is running (Step 22)
- ✅ ECR repositories exist (Step 16)
- ✅ Docker is installed and running
- ✅ Maven is installed (for backend services)

#### Step 27a: Configure UI Environment Variables

The UI build bakes `NEXT_PUBLIC_*` values into the Next.js bundle at `docker build` time — they're injected as `--build-arg` from `${CANOPY_HOME}/datahub-ui-main/.env.${CANOPY_ENV}`. Generate that file from the template with:

```bash
canopycli init ui-env
```

(Pass `--force` to overwrite an existing `.env.${CANOPY_ENV}`.)

**What this does:**
1. Reads `${CANOPY_HOME}/datahub-ui-main/.env.example` line by line (preserving comments).
2. Auto-fills any key whose value is already in the param file:
   - `NEXT_PUBLIC_BACKEND_URL`        ← `PublicHostname`
   - `NEXT_PUBLIC_KEYCLOAK_URL`       ← `PublicHostname`
   - `NEXT_PUBLIC_KEYCLOAK_REALM`     ← `KeycloakRealm`
   - `NEXT_PUBLIC_KEYCLOAK_CLIENT_ID` ← `KeycloakClientId`
   - `NODE_TLS_REJECT_UNAUTHORIZED`   ← `1` (hardcoded)
3. Leaves un-mapped keys (e.g. `NEXT_PUBLIC_GTAG` — the optional Google Analytics Measurement ID) with their `.env.example` defaults for you to fill in manually if needed.
4. Writes the result to `${CANOPY_HOME}/datahub-ui-main/.env.${CANOPY_ENV}` (e.g. `.env.prod`).

✅ **Verify:** open the generated file — `cat ${CANOPY_HOME}/datahub-ui-main/.env.${CANOPY_ENV}` — and confirm the auto-filled values line up with your deployment. If you want GA tracking, fill in `NEXT_PUBLIC_GTAG` before the UI build in Step 27b.

> When `canopycli aws ecs deploy ui` runs in Step 27b, it reads this same `.env.${CANOPY_ENV}` and passes every `NEXT_PUBLIC_*` value as a `--build-arg` to the Dockerfile's multi-stage Next.js build.

#### Step 27b: Deploy All Services

`canopycli aws ecs deploy <service>` automates the full build-and-deploy pipeline for every service. For the UI it automatically reads `.env.production` and passes the values as Docker build args — no extra flags needed.

```bash
# Deploy each service (one at a time; --tag defaults to 'latest' for all except keycloak)
canopycli aws ecs deploy user-service
canopycli aws ecs deploy submission-service
canopycli aws ecs deploy report-service
canopycli aws ecs deploy download-service
canopycli aws ecs deploy entity-service
canopycli aws ecs deploy search-service
canopycli aws ecs deploy ui
```

Pass `--tag <value>` to override the image tag and `--dry-run` to preview without executing.

**What the pipeline does for EACH service:**
1. ✅ Verifies source directory and Dockerfile exist under `${CANOPY_HOME}`
2. ✅ Builds the application:
   - **Backend services**: runs `mvn clean package -DskipTests -q`
   - **Frontend (ui)** and **keycloak**: build handled inside the Dockerfile (no local build)
3. ✅ Authenticates Docker with AWS ECR (via STS + ECR auth token)
4. ✅ `docker buildx build --platform linux/amd64 --push` (falls back to `docker build` + `docker push` if buildx isn't available)
5. ✅ Triggers ECS `UpdateService --force-new-deployment` on the running service
6. ✅ Polls the PRIMARY deployment for up to 60 s and reports when it's accepted
7. ℹ️ If the ECS service isn't ACTIVE yet (first-run or post-teardown), prints a clear "create the CFN stack first" message and exits 0 — the image is already in ECR

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

### Step 28: Verify Deployment
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

### Step 29: Test Application

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
   - [RDS Deployment](https://github.com/canopy-datahub/canopy-development/blob/feature/aws/db/postgres/db-create-scripts/README_RDS_DEPLOYMENT.md)
   - [Lambda Deployment](https://github.com/canopy-datahub/canopy-development/blob/feature/aws/opensearch/opensearch_reindex/README_OPENSEARCH_REINDEX_LAMBDA_DEPLOYMENT.md)

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

