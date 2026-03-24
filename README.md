# gas-deploy

Deploy any HTML file to Google Apps Script as a web app — in one command.

## What is this?

`gas-deploy` wraps Google's [clasp](https://github.com/google/clasp) CLI to provide a simple two-step workflow:

1. `gas-deploy init my-app.html` → Creates an Apps Script project, deploys your HTML as a web app, gives you a URL
2. `gas-deploy push my-app.html` → Updates the deployed web app with your latest HTML

No need to touch the Apps Script editor. No need to manage deployment IDs. Just push HTML and get a URL.

## Prerequisites

- **macOS or Linux** (Windows WSL should also work)
- **Node.js** (auto-installed via Homebrew if missing on macOS)
- **Google account** with Apps Script API enabled

## Quick Start

```bash
# Install
git clone https://github.com/<your-user>/gas-deploy.git ~/source/gas-deploy
mkdir -p ~/bin && ln -s ~/source/gas-deploy/gas-deploy ~/bin/gas-deploy

# Make sure ~/bin is in your PATH (add to .zshrc if not already there)
export PATH="$HOME/bin:$PATH"

# Initialize (first time only — creates project + deploys)
gas-deploy init my-dashboard.html --title "My Dashboard"

# Update (anytime you change the HTML)
gas-deploy push my-dashboard.html
```

On first run, `gas-deploy` will:
- Install `clasp` if not found
- Open a browser for Google OAuth login
- Guide you to enable the Apps Script API if needed

## How it works

```
my-app.html
    │
    ▼
gas-deploy init
    │
    ├── clasp create (Apps Script project)
    ├── Generate Code.gs (doGet → HtmlService)
    ├── Generate appsscript.json (webapp config)
    ├── clasp push (upload files)
    ├── clasp deploy (create web app)
    └── Save config to .gas-deploy.json
    │
    ▼
https://script.google.com/macros/s/.../exec  ← Your web app URL (fixed!)
```

On subsequent `gas-deploy push`, it updates the **same deployment ID**, so the URL never changes.

## Commands

| Command | Description |
|---------|-------------|
| `gas-deploy init <file> --title "Name"` | Create project + first deploy |
| `gas-deploy push <file>` | Update existing deployment |
| `gas-deploy list` | Show all configured apps |
| `gas-deploy open <file>` | Open the deployed URL in browser |

### Options for `init`

| Option | Default | Description |
|--------|---------|-------------|
| `--title "Name"` | filename | Title shown in browser tab |
| `--access DOMAIN` | DOMAIN | `DOMAIN` = org only, `ANYONE_ANONYMOUS` = public |

## Configuration

`gas-deploy` stores config in `.gas-deploy.json` in your current directory:

```json
{
  "apps": {
    "my-app.html": {
      "script_id": "1vNzsvh...",
      "deploy_id": "AKfycbx...",
      "title": "My App",
      "access": "DOMAIN",
      "gas_dir": "/tmp/gas-deploy-my-app"
    }
  }
}
```

You can manage multiple HTML files in the same directory — each gets its own Apps Script project.

## Usage with Claude Code

Combine `gas-deploy` with a build/sync script for automated deployments:

```bash
#!/bin/bash
# sync_my_app.sh

# Step 1: Your custom build/transform step
python3 build_data.py > data.json
python3 inject_data.py template.html data.json > my-app.html

# Step 2: Deploy
gas-deploy push my-app.html
```

In your `CLAUDE.md`:
```markdown
## Deploy
After modifying my-app.html, run:
\`\`\`bash
bash sync_my_app.sh
\`\`\`
```

## Access Control

- **`DOMAIN`** (default): Only users in your Google Workspace organization can access
- **`ANYONE_ANONYMOUS`**: Anyone with the URL can access (no login required)

To change access after init, update the deployment in the Apps Script editor:
1. Open `https://script.google.com/d/<script_id>/edit`
2. Deploy → Manage deployments → Edit → Access

## Troubleshooting

### "User has not enabled the Apps Script API"
→ Visit https://script.google.com/home/usersettings and toggle the API on

### "clasp login" opens browser but nothing happens
→ Make sure you're logged into the correct Google account in the browser

### "Deployed but can't access"
→ The first deployment may need manual access configuration in the Apps Script editor (Deploy → Manage deployments → Edit access)

### Large HTML files (>50MB)
→ Apps Script has a 50MB limit per file. Split your app or optimize assets.

## License

MIT
