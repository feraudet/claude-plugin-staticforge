# AWS Docusaurus: Check Status

Check AWS infrastructure and deployment status.

## Interactive Flow

Execute these steps in order:

### Step 1: Check and Prompt for Required Variables

For each missing variable, use AskUserQuestion:

1. **CLOUDFRONT_DISTRIBUTION_ID** (required)
   - Check: `echo $CLOUDFRONT_DISTRIBUTION_ID`
   - If empty, ask: "What is the CloudFront distribution ID?" (e.g., "E1234567890ABC")

2. **S3_BUCKET** (required)
   - Check: `echo $S3_BUCKET`
   - If empty, ask: "What is the S3 bucket name?"

3. **AWS_PROFILE** (optional)
   - Default: "default"

4. **DOMAIN** (optional, for health checks)
   - Check: `echo $DOMAIN`
   - If empty, ask: "What is the site domain? (optional, for health check)"

### Step 2: Display Summary and Confirm

Show configuration:

```
Status Check Configuration
==========================
CloudFront ID:   ${CLOUDFRONT_DISTRIBUTION_ID}
S3 Bucket:       ${S3_BUCKET}
AWS Profile:     ${AWS_PROFILE}
Domain:          ${DOMAIN}

Run status checks?
```

Use AskUserQuestion with options:
- "Yes, check status"
- "No, cancel"

### Step 3: Execute Status Checks

Only after user confirms, run all checks:

#### CloudFront Status
```bash
aws cloudfront get-distribution \
  --id ${CLOUDFRONT_DISTRIBUTION_ID} \
  --query '{Status: Distribution.Status, Enabled: Distribution.DistributionConfig.Enabled}'
```

#### S3 Bucket Status
```bash
aws s3api head-bucket --bucket ${S3_BUCKET}
aws s3 ls s3://${S3_BUCKET}/ --recursive --summarize | tail -2
```

#### Recent Invalidations
```bash
aws cloudfront list-invalidations \
  --distribution-id ${CLOUDFRONT_DISTRIBUTION_ID} \
  --query 'InvalidationList.Items[0:3]'
```

#### Certificate Status
```bash
CERT_ARN=$(aws cloudfront get-distribution \
  --id ${CLOUDFRONT_DISTRIBUTION_ID} \
  --query 'Distribution.DistributionConfig.ViewerCertificate.ACMCertificateArn' \
  --output text)

aws acm describe-certificate \
  --region us-east-1 \
  --certificate-arn ${CERT_ARN} \
  --query '{Status: Certificate.Status, Expiry: Certificate.NotAfter}'
```

#### Site Health (if DOMAIN set)
```bash
curl -sI https://${DOMAIN} | head -1
curl -w "TTFB: %{time_starttransfer}s\n" -o /dev/null -s https://${DOMAIN}
```

### Step 4: Display Results

Show formatted status report:

```
AWS Docusaurus Status
=====================

CloudFront: ${CLOUDFRONT_DISTRIBUTION_ID}
├── Status:  Deployed ✓
├── Enabled: true ✓
└── Domain:  ${CF_DOMAIN}

S3 Bucket: ${S3_BUCKET}
├── Status:  Accessible ✓
├── Objects: ${OBJECT_COUNT}
└── Size:    ${BUCKET_SIZE}

Certificate:
├── Status:  ISSUED ✓
└── Expires: ${CERT_EXPIRY}

Site Health: https://${DOMAIN}
├── Status:  200 OK ✓
└── TTFB:    ${TTFB}s

Last Invalidations:
├── ${INV_1_ID} - ${INV_1_STATUS}
└── ${INV_2_ID} - ${INV_2_STATUS}
```

### Step 5: Offer Quick Actions

Use AskUserQuestion to offer actions:

"What would you like to do next?"
- "Invalidate CloudFront cache"
- "Redeploy site"
- "Nothing, I'm done"

Execute selected action if requested.
