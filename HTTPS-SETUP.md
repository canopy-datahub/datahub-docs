# HTTPS Setup for canopy-dev.egyedia.com

## Step 1 — AWS: Request a Certificate in ACM

1. Go to the [AWS Certificate Manager console](https://console.aws.amazon.com/acm/home?region=us-east-1) (must be in the same region as the ELB)
2. Click **Request a certificate** → choose **Public certificate**
3. Enter domain name: `<YOUR-SUBDOMAIN-NAME>.<YOUR-DOMAIN-NAME>`
4. Choose **DNS validation** — ACM will provide a CNAME record (a name and a value) that you need to add to your DNS to prove domain ownership

## Step 2 — Your Domain's DNS Manager: Add the ACM Validation CNAME

1. In your **DNS manager** for `<YOUR-DOMAIN-NAME>`, create a new CNAME record using the name and value ACM gave you
2. It will look something like:
   - **Hostname:** `_abcdef1234.canopy-dev`
   - **Alias to:** `_xyz789.acm-validations.aws`
3. Wait a few minutes — ACM will detect the record and flip the certificate status to **Issued**
4. Note the certificate ARN (e.g., `arn:aws:acm:${AWS::Region}:${AWS::AccountId}:certificate/<CERT-UUID>`)

## Step 3 — AWS: Add HTTPS Listener to ELB (CloudFormation)

The load balancer is managed via CloudFormation in `modules/LoadBalancer.yaml`.

### 3a. Update the `CertificateId` parameter

In `parameters-dev-*.json`, add:

```json
"CertificateId": "<CERT-UUID>"
```

### 3b. Deploy (two-step to avoid port conflict)

Since both the old and new listeners use port 80, deploy in two steps:

**Step 1** — Deploy with only `HttpsLoadBalancerListener` (remove `HttpLoadBalancerListener` temporarily). This converts the existing listener from HTTP/80 to HTTPS/443, freeing port 80.

```bash
aws cloudformation deploy \
  --stack-name ${CANOPY_PROJECT_NAME}-LoadBalancer-${CANOPY_ENV} \
  --template-file modules/LoadBalancer.yaml \
  --parameter-overrides file://${CANOPY_AWS_PARAMETER_FILE} \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${CANOPY_PROJECT_NAME} environment=${CANOPY_ENV}
```

**Step 2** — Add `HttpLoadBalancerListener` back into the template and deploy again. Port 80 is now free, so the redirect listener will be created.

```bash
aws cloudformation deploy \
  --stack-name ${CANOPY_PROJECT_NAME}-LoadBalancer-${CANOPY_ENV} \
  --template-file modules/LoadBalancer.yaml \
  --parameter-overrides file://${CANOPY_AWS_PARAMETER_FILE} \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile ${AWS_PROFILE} \
  --tags projectname=${CANOPY_PROJECT_NAME} environment=${CANOPY_ENV}
```

## Step 4 — Your DNS Manager: Point the Domain at the ELB

In your **DNS manager** for `<YOUR-DOMAIN-NAME>`, add a CNAME record:

| Field | Value               |
|-------|---------------------|
| **Type** | CNAME               |
| **Hostname** | `<YOUR-SUBDOMAIN-NAME>`        |
| **Alias to** | `<ALB-DOMAIN-NAME>` |
| **TTL** | Default             |

This tells DNS that `<YOUR-SUBDOMAIN-NAME>.<YOUR-DOMAIN-NAME>` resolves to the load balancer.

## Verify

After DNS propagates (usually a few minutes):

```bash
# Should return HTTPS response from UI service
curl -I https://<YOUR-SUBDOMAIN-NAME>.<YOUR-DOMAIN-NAME>

# Should 301 redirect to HTTPS
curl -I http://<YOUR-SUBDOMAIN-NAME>.<YOUR-DOMAIN-NAME>
```
