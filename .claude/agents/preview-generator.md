# Preview Generator Agent

Generate animated GIF previews of AskUserQuestion interaction flows for documentation.

## Purpose

Create visual documentation showing how users interact with the plugin's AskUserQuestion prompts. These GIFs are used in README and documentation to help users understand the plugin workflow before using it.

## Prerequisites

- `asciinema` installed: `brew install asciinema`
- `agg` installed: `cargo install agg` or `brew install agg`
- Plugin commands with AskUserQuestion flows

## Interactive Flow

### Step 1: Identify Commands with AskUserQuestion

Analyze commands to find AskUserQuestion usage:

```bash
grep -l "AskUserQuestion" commands/*.md
```

Display:
```
Commands with interactive flows found:
- env.md : 5 AskUserQuestion prompts
- init.md : 3 AskUserQuestion prompts
- infra.md : 4 AskUserQuestion prompts
- deploy.md : 2 AskUserQuestion prompts
- destroy-infra.md : 2 AskUserQuestion prompts
```

### Step 2: Select Commands to Generate

Use AskUserQuestion:
"Which command flows do you want to generate GIFs for?"
- "env" - Environment management flow
- "init" - Project initialization flow
- "deploy" - Deployment flow
- "All commands"

### Step 3: Configure Recording

Use AskUserQuestion:
"Recording configuration?"
- "Standard" - 80x24 terminal, 2s per frame
- "Large" - 120x30 terminal, 2s per frame
- "Compact" - 60x20 terminal, 1.5s per frame
- "Custom" - Specify dimensions and timing

### Step 4: Create Simulation Script

For each selected command, create a simulation script that mimics the AskUserQuestion flow:

```bash
# assets/previews/${CMD}-script.sh
#!/bin/bash

# Simulate terminal typing
type_text() {
  echo -n "$1" | while IFS= read -r -n1 char; do
    echo -n "$char"
    sleep 0.05
  done
  echo ""
}

clear
echo "$ /yaccp-aws-docusaurus:${CMD}"
sleep 1

echo ""
echo "? Which environment do you want to use?"
echo "  ○ dev (Development)"
echo "  ● staging (Staging)"
echo "  ○ prod (Production)"
sleep 2

echo ""
echo "✓ Selected: staging"
sleep 1

# Continue with the flow...
```

### Step 5: Record with Asciinema

```bash
asciinema rec assets/previews/${CMD}-flow.cast \
  --command "bash assets/previews/${CMD}-script.sh" \
  --cols 80 \
  --rows 24 \
  --idle-time-limit 3
```

### Step 6: Convert to GIF

```bash
agg assets/previews/${CMD}-flow.cast assets/previews/${CMD}-flow.gif \
  --theme monokai \
  --speed 1 \
  --font-size 14
```

### Step 7: Optimize GIF (Optional)

```bash
# Reduce file size
gifsicle -O3 --colors 256 \
  assets/previews/${CMD}-flow.gif \
  -o assets/previews/${CMD}-flow.gif
```

### Step 8: Clean Up

```bash
rm assets/previews/${CMD}-script.sh
rm assets/previews/${CMD}-flow.cast
```

### Step 9: Summary

```
Preview GIFs generated successfully!

Created:
- assets/previews/env-flow.gif (245 KB)
- assets/previews/init-flow.gif (312 KB)
- assets/previews/deploy-flow.gif (198 KB)

To use in documentation:
![Env Flow](assets/previews/env-flow.gif)
```

## AskUserQuestion Visual Style

When simulating AskUserQuestion prompts, use this visual style:

```
? Question text here?
  ○ Option 1 (description)
  ● Option 2 (selected)
  ○ Option 3

✓ Selected: Option 2
```

## Tips

- Keep GIFs under 500KB for faster loading
- Use 2-3 second pauses to let viewers read
- Highlight the selected option clearly
- Show the result/confirmation after selection
- Include a brief "success" message at the end
