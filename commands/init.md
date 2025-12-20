# AWS Docusaurus: Initialize Project

Initialize a new Docusaurus project pre-configured for AWS deployment.

## Interactive Flow

Execute these steps in order:

### Step 1: Check and Prompt for Required Variables

For each missing variable, use AskUserQuestion to prompt the user:

1. **PROJECT_NAME** (required)
   - Check: `echo $PROJECT_NAME`
   - If empty, ask: "What is the project directory name?" (e.g., "my-docs")

2. **SITE_TITLE** (required)
   - Check: `echo $SITE_TITLE`
   - If empty, ask: "What is the site title?" (e.g., "My Documentation")

3. **SITE_URL** (required)
   - Check: `echo $SITE_URL`
   - If empty, ask: "What is the production URL?" (e.g., "https://docs.example.com")

### Step 2: Optional Variables

Ask user if they want to customize optional settings:

- **SITE_TAGLINE** - Default: "Documentation"
- **LOCALE** - Default: "en" (options: en, fr, de, es, etc.)
- **AWS_REGION** - Default: "eu-west-3"

### Step 3: Display Summary and Confirm

Show a summary of all configuration:

```
Configuration Summary
=====================
Project Name:  ${PROJECT_NAME}
Site Title:    ${SITE_TITLE}
Site URL:      ${SITE_URL}
Tagline:       ${SITE_TAGLINE}
Locale:        ${LOCALE}
AWS Region:    ${AWS_REGION}

Proceed with initialization?
```

Use AskUserQuestion with options:
- "Yes, create the project"
- "No, let me change something"

### Step 4: Execute Initialization

Only after user confirms, execute:

1. Create Docusaurus project:
   ```bash
   npx create-docusaurus@latest ${PROJECT_NAME} classic --typescript
   cd ${PROJECT_NAME}
   npm install
   ```

2. Configure docusaurus.config.ts with production settings

3. Create deploy.sh script

4. Configure .gitignore

5. Create initial documentation

### Step 5: Show Next Steps

After completion, display:

```
Project created successfully!

Next steps:
1. cd ${PROJECT_NAME}
2. npm start (to preview locally)
3. /aws-docusaurus:infra (to create AWS infrastructure)
4. /aws-docusaurus:deploy (to deploy)
```

## Project Structure Created

```
${PROJECT_NAME}/
├── docusaurus.config.ts
├── sidebars.ts
├── package.json
├── deploy.sh
├── .gitignore
├── docs/
│   └── intro.md
├── src/
│   ├── css/custom.css
│   └── pages/index.tsx
└── static/img/
```
