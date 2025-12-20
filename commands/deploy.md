# AWS Docusaurus: Deploy Site

Deploy static site to AWS S3 + CloudFront.

## Interactive Flow

Execute these steps in order:

### Step 1: Auto-Detect Framework

Check for framework configuration files:

```bash
# Detect framework
ls docusaurus.config.* 2>/dev/null && echo "Docusaurus detected"
ls next.config.* 2>/dev/null && echo "Next.js detected"
ls vite.config.* 2>/dev/null && echo "Vite detected"
ls astro.config.* 2>/dev/null && echo "Astro detected"
ls gatsby-config.* 2>/dev/null && echo "Gatsby detected"
ls hugo.toml config.toml 2>/dev/null && echo "Hugo detected"
```

Set defaults based on detected framework:

| Framework | BUILD_COMMAND | BUILD_DIR |
|-----------|---------------|-----------|
| Docusaurus | `npm run build` | `build` |
| Next.js | `npm run build` | `out` |
| Vite | `npm run build` | `dist` |
| Astro | `npm run build` | `dist` |
| Gatsby | `gatsby build` | `public` |
| Hugo | `hugo --minify` | `public` |

### Step 2: Check and Prompt for Required Variables

For each missing variable, use AskUserQuestion:

1. **S3_BUCKET** (required)
   - Check: `echo $S3_BUCKET`
   - If empty, ask: "What is the S3 bucket name?"

2. **CLOUDFRONT_DISTRIBUTION_ID** (required)
   - Check: `echo $CLOUDFRONT_DISTRIBUTION_ID`
   - If empty, ask: "What is the CloudFront distribution ID?" (e.g., "E1234567890ABC")

3. **BUILD_COMMAND** (auto-detected or ask)
   - Check: `echo $BUILD_COMMAND`
   - If empty and not auto-detected, ask: "What is the build command?"
   - Default based on framework

4. **BUILD_DIR** (auto-detected or ask)
   - Check: `echo $BUILD_DIR`
   - If empty and not auto-detected, ask: "What is the build output directory?"
   - Default based on framework

5. **AWS_PROFILE** (optional)
   - Default: "default"

### Step 3: Display Summary and Confirm

Show deployment configuration:

```
Deployment Configuration
========================
Framework:       ${DETECTED_FRAMEWORK}
Build Command:   ${BUILD_COMMAND}
Build Directory: ${BUILD_DIR}
S3 Bucket:       ${S3_BUCKET}
CloudFront ID:   ${CLOUDFRONT_DISTRIBUTION_ID}
AWS Profile:     ${AWS_PROFILE}

Cache Strategy:
• JS/CSS/Images: 1 year (immutable)
• HTML/JSON:     No cache (always fresh)

Proceed with deployment?
```

Use AskUserQuestion with options:
- "Yes, build and deploy"
- "No, let me change something"

### Step 4: Execute Deployment

Only after user confirms:

1. Build the site:
   ```bash
   ${BUILD_COMMAND}
   ```

2. Verify build directory exists

3. Upload static assets (1-year cache):
   ```bash
   aws s3 sync ${BUILD_DIR}/ s3://${S3_BUCKET}/ \
     --delete \
     --cache-control "public, max-age=31536000, immutable" \
     --exclude "*.html" --exclude "*.json" --exclude "sw.js"
   ```

4. Upload HTML (no cache):
   ```bash
   aws s3 sync ${BUILD_DIR}/ s3://${S3_BUCKET}/ \
     --exclude "*" --include "*.html" \
     --cache-control "public, max-age=0, must-revalidate"
   ```

5. Invalidate CloudFront:
   ```bash
   aws cloudfront create-invalidation \
     --distribution-id ${CLOUDFRONT_DISTRIBUTION_ID} \
     --paths "/*"
   ```

### Step 5: Show Results

After completion, display:

```
Deployment successful!

Site URL:        https://${DOMAIN}
Files uploaded:  ${FILE_COUNT}
Invalidation:    ${INVALIDATION_ID}

Check status: /aws-docusaurus:status
```
