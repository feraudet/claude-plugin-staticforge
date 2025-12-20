# AWS Docusaurus: Create AWS Infrastructure

Create complete AWS infrastructure for static site hosting.

## Interactive Flow

Execute these steps in order:

### Step 1: Check and Prompt for Required Variables

For each missing variable, use AskUserQuestion to prompt the user:

1. **SITE_NAME** (required)
   - Check: `echo $SITE_NAME`
   - If empty, ask: "What is the site/bucket name?" (e.g., "my-docs")

2. **DOMAIN** (required)
   - Check: `echo $DOMAIN`
   - If empty, ask: "What is the custom domain?" (e.g., "docs.example.com")

3. **HOSTED_ZONE_ID** (required)
   - Check: `echo $HOSTED_ZONE_ID`
   - If empty, ask: "What is the Route53 Hosted Zone ID?" (e.g., "Z1234567890ABC")
   - Hint: Run `aws route53 list-hosted-zones` to find it

4. **AWS_PROFILE** (optional)
   - Check: `echo $AWS_PROFILE`
   - If empty, ask: "Which AWS CLI profile to use?"
   - Default: "default"

5. **AWS_REGION** (optional)
   - Check: `echo $AWS_REGION`
   - If empty, ask: "Which AWS region?"
   - Default: "eu-west-3"

### Step 2: Basic Authentication (Optional)

Use AskUserQuestion to ask:

"Do you want to enable Basic Authentication?"
- Options: "Yes", "No"

If yes, prompt for:
- **AUTH_USERNAME**: "Enter the username"
- **AUTH_PASSWORD**: "Enter the password" (min 8 characters)

### Step 3: Display Summary and Confirm

Show complete configuration summary:

```
Infrastructure Configuration
============================
Site Name:       ${SITE_NAME}
Domain:          ${DOMAIN}
Hosted Zone ID:  ${HOSTED_ZONE_ID}
AWS Profile:     ${AWS_PROFILE}
AWS Region:      ${AWS_REGION}
Basic Auth:      ${AUTH_ENABLED} (username: ${AUTH_USERNAME})

Resources to create:
• S3 Bucket: ${SITE_NAME}
• ACM Certificate: ${DOMAIN} (us-east-1)
• CloudFront Distribution
• Route53 A Record: ${DOMAIN}
• Lambda@Edge: ${SITE_NAME}-basic-auth (if auth enabled)

Proceed with infrastructure creation?
```

Use AskUserQuestion with options:
- "Yes, create infrastructure"
- "No, let me change something"

### Step 4: Execute Infrastructure Creation

Only after user confirms:

1. Create S3 bucket (private, block public access)
2. Request ACM certificate in us-east-1
3. Wait for certificate validation
4. Create CloudFront OAI
5. Configure S3 bucket policy
6. Create Lambda@Edge (if auth enabled)
7. Create CloudFront distribution
8. Create Route53 alias record

### Step 5: Show Results

After completion, display:

```
Infrastructure created successfully!

S3 Bucket:              ${SITE_NAME}
CloudFront ID:          ${CLOUDFRONT_DISTRIBUTION_ID}
CloudFront Domain:      ${CF_DOMAIN}
Site URL:               https://${DOMAIN}

Export these for deployment:
export S3_BUCKET="${SITE_NAME}"
export CLOUDFRONT_DISTRIBUTION_ID="${CF_ID}"

Next step: /aws-docusaurus:deploy
```

## Architecture

```
Route53 → CloudFront (+ Lambda@Edge) → S3 (private)
              ↓
         ACM Certificate (HTTPS)
```
