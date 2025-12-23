---
description: Main dashboard and command center for AWS Docusaurus plugin
---

# AWS Docusaurus: Action Center

Interactive dashboard and command center for the AWS Docusaurus plugin. This is the recommended entry point when using the plugin for the first time or as a central hub to access all features.

## Configuration Storage

All configuration is stored in `.claude/yaccp/aws-docusaurus/config.json`:

```json
{
  "environments": {
    "dev": { ... },
    "staging": { ... },
    "prod": { ... }
  },
  "currentEnvironment": "dev",
  "defaults": {
    "AWS_REGION": "eu-west-1",
    "BUILD_COMMAND": "npm run build",
    "BUILD_DIR": "build"
  },
  "init": {
    "PROJECT_NAME": "...",
    "SITE_TITLE": "..."
  },
  "localServer": {
    "PORT": "3000",
    "PID": null
  }
}
```

## Interactive Flow

### Step 1: Load Configuration and Detect State

Read existing config:
```bash
cat .claude/yaccp/aws-docusaurus/config.json 2>/dev/null
```

Determine the current state:
- **NO_CONFIG**: No config file exists → First time setup
- **NO_ENVIRONMENT**: Config exists but no environments → Environment setup needed
- **NO_INFRA**: Environment exists but no S3/CloudFront → Infrastructure needed
- **READY**: Full configuration exists → Show dashboard

### Step 2: Display Welcome Banner

```
╔══════════════════════════════════════════════════════════════╗
║           AWS Docusaurus - Action Center                     ║
╠══════════════════════════════════════════════════════════════╣
║  Deploy static sites to AWS with S3, CloudFront & SSL        ║
╚══════════════════════════════════════════════════════════════╝
```

### Step 3: Display Current Status Summary

If configuration exists, show:

```
Current Status
══════════════
Environment:    ${CURRENT_ENV} (${ENV_NAME})
AWS Profile:    ${AWS_PROFILE}
AWS Region:     ${AWS_REGION}
AWS Account:    ${AWS_ACCOUNT_ID}

Infrastructure:
├── S3 Bucket:     ${S3_BUCKET:-"Not configured"}
├── CloudFront:    ${CLOUDFRONT_DISTRIBUTION_ID:-"Not configured"}
├── Domain:        ${DOMAIN:-"Not configured"}
└── Certificate:   ${CERT_STATUS:-"Not configured"}

Local Server:
└── Status:        ${LOCAL_SERVER_STATUS}

Project:
├── Name:          ${PROJECT_NAME:-"Not initialized"}
├── Build Dir:     ${BUILD_DIR}
└── Last Deploy:   ${LAST_DEPLOY:-"Never"}
```

### Step 4: First Time Setup Wizard

If state is **NO_CONFIG** or **NO_ENVIRONMENT**, prompt:

Use AskUserQuestion:
"Welcome! Let's set up your AWS Docusaurus deployment. What would you like to do first?"
- "Create a new Docusaurus project" - Run `/yaccp-aws-docusaurus:yaccp-aws-docusaurus-init`
- "Configure AWS environment" - Run `/yaccp-aws-docusaurus:yaccp-aws-docusaurus-env`
- "I have an existing project to deploy" - Configure for existing project

#### If "I have an existing project to deploy":

Collect essential information using AskUserQuestion:

**Step 4a: Project Detection**

Check for common framework indicators:
```bash
ls -la package.json docusaurus.config.* next.config.* vite.config.* astro.config.* 2>/dev/null
```

Use AskUserQuestion:
"What framework is your project using?"
- "Docusaurus" - Set BUILD_COMMAND="npm run build", BUILD_DIR="build"
- "Next.js (Static Export)" - Set BUILD_COMMAND="npm run build && npm run export", BUILD_DIR="out"
- "Vite / Vue / React" - Set BUILD_COMMAND="npm run build", BUILD_DIR="dist"
- "Other" - Prompt for custom values

**Step 4b: AWS Configuration**

Use AskUserQuestion for each:
- "AWS Profile name?" (suggest profiles from `~/.aws/credentials`)
- "AWS Region?" with options: eu-west-1, us-east-1, ap-northeast-1, etc.
- "Site domain?" (e.g., docs.example.com)

**Step 4c: Environment Name**

Use AskUserQuestion:
"What should we call this environment?"
- "dev" (Development)
- "staging" (Staging)
- "prod" (Production)
- "custom" - Prompt for custom name

**Step 4d: Save Initial Configuration**

```bash
mkdir -p .claude/yaccp/aws-docusaurus
```

Write config.json with collected values.

### Step 5: Main Action Menu

Use AskUserQuestion:
"What would you like to do?"

**Quick Actions (most common):**
- "Deploy site" - Build and deploy to current environment
- "Check status" - View infrastructure and deployment status
- "Start local server" - Run development server

**Project Management:**
- "Initialize new project" - Create new Docusaurus site
- "Configure build settings" - Change build command/directory

**Environment Management:**
- "Manage environments" - Switch, create, edit environments
- "View current configuration" - Display all settings

**Infrastructure:**
- "Create AWS infrastructure" - Provision S3, CloudFront, SSL
- "Destroy infrastructure" - Remove all AWS resources

**Development:**
- "Start local server" - npm start for local development
- "Stop local server" - Stop running server
- "Server status" - Check local server

**Troubleshooting:**
- "Run diagnostics" - Check plugin and AWS setup
- "Report an issue" - Create GitHub issue

### Step 6: Execute Selected Action

Based on selection, either:
1. Run the corresponding slash command
2. Or handle inline for simple actions

#### Quick Actions Mapping:

| Selection | Action |
|-----------|--------|
| Deploy site | `/yaccp-aws-docusaurus:yaccp-aws-docusaurus-deploy` |
| Check status | `/yaccp-aws-docusaurus:yaccp-aws-docusaurus-status` |
| Initialize new project | `/yaccp-aws-docusaurus:yaccp-aws-docusaurus-init` |
| Manage environments | `/yaccp-aws-docusaurus:yaccp-aws-docusaurus-env` |
| Create AWS infrastructure | `/yaccp-aws-docusaurus:yaccp-aws-docusaurus-infra` |
| Destroy infrastructure | `/yaccp-aws-docusaurus:yaccp-aws-docusaurus-destroy-infra` |
| Start local server | `/yaccp-aws-docusaurus:yaccp-aws-docusaurus-start-local-server` |
| Stop local server | `/yaccp-aws-docusaurus:yaccp-aws-docusaurus-stop-local-server` |
| Server status | `/yaccp-aws-docusaurus:yaccp-aws-docusaurus-status-local-server` |
| Run diagnostics | `/yaccp-aws-docusaurus:yaccp-aws-docusaurus-doctor` |
| Report an issue | `/yaccp-aws-docusaurus:yaccp-aws-docusaurus-issues` |

#### Configure Build Settings (inline):

Use AskUserQuestion:
"What build command should be used?"
- Input: Custom build command

Use AskUserQuestion:
"What is the output directory?"
- "build" (Docusaurus)
- "out" (Next.js)
- "dist" (Vite/Vue/React)
- "public" (Hugo)
- "Other" - Custom directory

Save to `config.defaults.BUILD_COMMAND` and `config.defaults.BUILD_DIR`.

#### View Current Configuration (inline):

Display full configuration:
```
Full Configuration
══════════════════

Environments:
┌─────────┬────────────────┬───────────┬──────────────────┐
│ ID      │ AWS Profile    │ Region    │ Domain           │
├─────────┼────────────────┼───────────┼──────────────────┤
│ dev*    │ company-dev    │ eu-west-1 │ dev.example.com  │
│ staging │ company-staging│ eu-west-1 │ stg.example.com  │
│ prod    │ company-prod   │ eu-west-1 │ example.com      │
└─────────┴────────────────┴───────────┴──────────────────┘
* = current environment

Defaults:
├── Build Command: ${BUILD_COMMAND}
├── Build Dir:     ${BUILD_DIR}
└── AWS Region:    ${AWS_REGION}

Config File: .claude/yaccp/aws-docusaurus/config.json
```

### Step 7: Post-Action Menu

After completing an action, offer to return to the dashboard:

Use AskUserQuestion:
"What would you like to do next?"
- "Back to Action Center" - Return to main menu (Step 5)
- "Exit" - End the session

## Contextual Suggestions

Based on the current state, prioritize suggestions:

**If NO_INFRA:**
```
Recommended: Create AWS infrastructure first
Your environment is configured but AWS resources haven't been created yet.
```
Highlight "Create AWS infrastructure" option.

**If INFRA_EXISTS but never deployed:**
```
Recommended: Deploy your site
Infrastructure is ready! Build and deploy your site.
```
Highlight "Deploy site" option.

**If recently deployed:**
```
Last deployment: 2 hours ago to dev.example.com
Site is live at: https://${DOMAIN}
```
Highlight "Check status" option.

## Keyboard Shortcuts Reference

Display at bottom of menu:
```
Quick Commands:
• /yaccp-aws-docusaurus:yaccp-aws-docusaurus-deploy   - Deploy directly
• /yaccp-aws-docusaurus:yaccp-aws-docusaurus-status   - Check status
• /yaccp-aws-docusaurus:yaccp-aws-docusaurus-env      - Switch environment
• export PLUGIN_ENV=prod   - Override environment
```

## Error Handling

If AWS credentials are not configured:
```
AWS credentials not found for profile: ${AWS_PROFILE}

Please ensure:
1. AWS CLI is installed: brew install awscli
2. Profile is configured: aws configure --profile ${AWS_PROFILE}
3. Or set environment variables: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY
```

Use AskUserQuestion:
- "Run diagnostics" - `/yaccp-aws-docusaurus:yaccp-aws-docusaurus-doctor`
- "Configure different profile"
- "Exit"

## Full Workflow Example

```
1. User runs: /yaccp-aws-docusaurus:yaccp-aws-docusaurus-action

2. Action Center detects: NO_CONFIG state

3. Wizard prompts:
   → "I have an existing project to deploy"
   → Framework: "Docusaurus"
   → AWS Profile: "my-profile"
   → Region: "eu-west-1"
   → Domain: "docs.mysite.com"
   → Environment: "dev"

4. Config saved, state changes to: NO_INFRA

5. Main menu shows with recommendation:
   "Recommended: Create AWS infrastructure"
   → User selects "Create AWS infrastructure"
   → Runs /yaccp-aws-docusaurus:yaccp-aws-docusaurus-infra

6. After infra created, returns to Action Center:
   "Recommended: Deploy your site"
   → User selects "Deploy site"
   → Runs /yaccp-aws-docusaurus:yaccp-aws-docusaurus-deploy

7. Deployment complete:
   "Your site is live at https://docs.mysite.com"
   → "Back to Action Center" or "Exit"
```
