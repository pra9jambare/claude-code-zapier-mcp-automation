# 🤖 AI-Powered Cross-Platform Automation with Claude Code + OpenRouter + Zapier MCP

> **End-to-End Guide:** From a fresh terminal to posting on LinkedIn — all through natural language.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Step 1: Install Claude Code](#step-1-install-claude-code)
- [Step 2: Set Up OpenRouter](#step-2-set-up-openrouter)
- [Step 3: Configure Claude Code Gateway (OpenRouter)](#step-3-configure-claude-code-gateway-openrouter)
- [Step 4: Install & Authenticate Zapier MCP](#step-4-install--authenticate-zapier-mcp)
- [Step 5: Configure MCP Server in Claude Code](#step-5-configure-mcp-server-in-claude-code)
- [Step 6: Connect Gmail & LinkedIn via Zapier](#step-6-connect-gmail--linkedin-via-zapier)
- [Step 7: Test Gmail Integration](#step-7-test-gmail-integration)
- [Step 8: Test LinkedIn Integration](#step-8-test-linkedin-integration)
- [Step 9: Create & Post to LinkedIn](#step-9-create--post-to-linkedin)
- [What We Achieved](#what-we-achieved)
- [Real-World Use Cases](#real-world-use-cases)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

---

## Overview

This guide walks through setting up a **local-first, model-agnostic automation stack** that connects:

- **Claude Code** — AI terminal assistant by Anthropic
- **OpenRouter** — Unified API gateway to 200+ LLMs
- **DeepSeek V4 Flash** — The routed model powering the agent
- **Zapier MCP** — Bridge connecting 7,000+ apps via the Model Context Protocol
- **Gmail & LinkedIn** — The apps we integrated end-to-end

**The result:** A single terminal session where you can read emails, send messages, verify profiles, and post to LinkedIn — all through natural language prompts.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Your Local Machine                         │
│                                                              │
│  ┌─────────────┐    ┌──────────────┐    ┌─────────────────┐ │
│  │ Claude Code │───▶│  OpenRouter  │───▶│ DeepSeek V4     │ │
│  │    (CLI)    │    │   Gateway    │    │   Flash (Model) │ │
│  └──────┬──────┘    └──────────────┘    └─────────────────┘ │
│         │                                                     │
│         │  MCP Protocol                                       │
│         ▼                                                     │
│  ┌──────────────┐                                             │
│  │ Zapier MCP   │                                             │
│  │   Server     │                                             │
│  └──────┬───────┘                                             │
│         │                                                     │
│         ├──────────────────┬──────────────────┐               │
│         ▼                  ▼                  ▼               │
│  ┌───────────┐    ┌───────────��──┐    ┌──────────────┐      │
│  │ Gmail API │    │ LinkedIn API │    │ 7000+ more   │      │
│  │ (OAuth2)  │    │ (OAuth2)     │    │   Apps       │      │
│  └───────────┘    └──────────────┘    └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

**Data flow:**
1. User types a natural language prompt in Claude Code
2. Claude Code routes the request through OpenRouter to DeepSeek V4 Flash
3. The model interprets intent and calls MCP tools
4. Zapier MCP server handles OAuth authentication with the target app
5. API call executes against Gmail / LinkedIn / etc.
6. Result flows back through the same chain

---

## Prerequisites

Before starting, ensure you have:

| Requirement | Details |
|------------|---------|
| **OS** | macOS (≥12), Linux (Ubuntu 20.04+, Debian 11+, Fedora 38+), or Windows 10/11 |
| **Node.js** | v18 or later |
| **npm** | v9 or later |
| **Internet** | Required for API calls to OpenRouter, Zapier, Gmail, LinkedIn |
| **Accounts** | GitHub (for Claude Code auth), [OpenRouter](https://openrouter.ai) account, [Zapier](https://zapier.com) account, Gmail account, LinkedIn account |

---

## Step 1: Install Claude Code

Claude Code is Anthropic's official CLI agent. It runs in your terminal and can execute commands, read/write files, and interact with MCP servers.

### macOS

```bash
# Using npm (recommended)
npm install -g @anthropic-ai/claude-code

# Or using Homebrew
brew install claude-code
```

### Linux

```bash
# Using npm (recommended)
npm install -g @anthropic-ai/claude-code

# Using apt (Ubuntu/Debian)
# First install Node.js if not present
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo npm install -g @anthropic-ai/claude-code
```

### Windows

```powershell
# Using npm (recommended)
npm install -g @anthropic-ai/claude-code

# Or using winget
winget install Claude.Code
```

### Verify Installation

```bash
claude --version
# Should output something like: claude-code v0.x.x
```

### Authenticate Claude Code

```bash
claude
# On first run, it will prompt you to authenticate via GitHub
# Follow the URL displayed in the terminal to complete OAuth
```

---

## Step 2: Set Up OpenRouter

OpenRouter provides a unified API to access 200+ LLMs including DeepSeek V4 Flash, Claude, GPT, Gemini, and more.

### 2.1 Create an OpenRouter Account

1. Go to [openrouter.ai](https://openrouter.ai)
2. Sign up using GitHub, Google, or email
3. Navigate to **Keys** → **Create API Key**
4. Copy your API key (starts with `sk-or-...`)

### 2.2 Fund Your Account (Optional)

- OpenRouter works on a pay-as-you-go model
- Add credits via **Billing** → **Add Credits**
- DeepSeek V4 Flash is very cost-effective (~$0.10–$0.50/M tokens)

### 2.3 Note Your Model ID

The model we used:
```
deepseek/deepseek-v4-flash
```

Find available models at: [openrouter.ai/models](https://openrouter.ai/models)

---

## Step 3: Configure Claude Code Gateway (OpenRouter)

Claude Code supports custom model routing through its configuration system.

### 3.1 Locate Claude Code Config

```bash
# Claude Code config file location
# macOS/Linux:
~/.claude/settings.json

# Windows:
%USERPROFILE%\.claude\settings.json

# Project-level config (overrides global):
.claude/settings.local.json
```

### 3.2 Configure the Model Gateway

Create or edit `~/.claude/settings.json`:

```json
{
  "model": "deepseek/deepseek-v4-flash",
  "provider": {
    "id": "openrouter",
    "api_key": "YOUR_OPENROUTER_API_KEY",  # Get yours at https://openrouter.ai/keys
    "base_url": "https://openrouter.ai/api/v1"
  }
}
```

> **⚠️ Security:** Never commit your API key to git. Use environment variables instead:
> ```bash
> export ANTHROPIC_API_KEY="YOUR_API_KEY"  # For direct Claude API
> export OPENROUTER_API_KEY="YOUR_API_KEY" # Alternative: via env
> ```
> Or reference it in settings as `"api_key": "$OPENROUTER_API_KEY"`

### 3.3 Verify the Gateway Works

```bash
claude -p "Say hello and tell me which model you are running on"
# Should respond confirming DeepSeek V4 Flash via OpenRouter
```

---

## Step 4: Install & Authenticate Zapier MCP

Zapier MCP (Model Context Protocol) is the bridge that connects your AI agent to 7,000+ apps.

### 4.1 Install the Zapier MCP Integration

Claude Code discovers MCP servers from your configuration. Add the Zapier MCP server:

Edit `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "zapier": {
      "type": "url",
      "url": "https://mcp.zapier.com"
    }
  }
}
```

> **Alternative:** You can also install via the `/mcp` command inside Claude Code.

### 4.2 Authenticate Zapier MCP

Inside a Claude Code session:

```bash
# Run the MCP setup command
/mcp
```

1. Select **Zapier-MCP** from the list of available MCP servers
2. A browser window will open asking you to authorize Zapier
3. Sign in to your Zapier account
4. Grant the requested permissions

> **If running headless (no browser):**
> - Claude Code will provide an OAuth URL
> - Open it manually in any browser
> - If the redirect fails, copy the callback URL from the address bar and paste it into the terminal

---

## Step 5: Configure MCP Server in Claude Code

### 5.1 Verify MCP Connection

Inside Claude Code:

```
/inspect mcp
```

Or simply ask:
```
"List my connected MCP servers"
```

### 5.2 Check Available Tools

Once connected, the Zapier MCP tools become available:

| Tool | Purpose |
|------|---------|
| `inspect_zapier_actions` | List connected apps and their available actions |
| `discover_zapier_actions` | Search 7,000+ apps to find new actions |
| `enable_zapier_action` | Enable a specific app/action |
| `execute_zapier_read_action` | Read/search data from connected apps |
| `execute_zapier_write_action` | Create/update data in connected apps |
| `list_zapier_connections` | View authenticated accounts per app |
| `manage_zapier_connections` | Switch accounts or reconnect |
| `list_zapier_skills` | View saved workflow skills |
| `search_triggers` | Check if an app has event triggers |

### 5.3 Load the Quickstart Skill

Zapier provides a built-in onboarding skill:

```
"Load the Zapier quickstart skill"
```

This guides you through connecting your first apps.

---

## Step 6: Connect Gmail & LinkedIn via Zapier

### 6.1 Discover Available Apps

```bash
# In Claude Code, ask:
"Check what apps I can connect through Zapier"
```

Claude Code will call `inspect_zapier_actions` to list enabled apps.

### 6.2 Enable Gmail Actions

```bash
# Ask Claude:
"Enable Gmail actions through Zapier"
```

Claude will call `enable_zapier_action` with `selected_api: "GoogleMailV2CLIAPI"`.

### 6.3 Enable LinkedIn Actions

```bash
# Ask Claude:
"Enable LinkedIn actions through Zapier"
```

Claude will call `enable_zapier_action` with `selected_api: "LinkedInCLIAPI"`.

### 6.4 Authenticate Each App

After enabling, you'll receive OAuth sign-in links:

```
**Sign in to Gmail and LinkedIn**
- Gmail: https://mcp.zapier.com/api/v1/connect-auth/GoogleMailV2CLIAPI?accountId=...
- LinkedIn: https://mcp.zapier.com/api/v1/connect-auth/LinkedInCLIAPI?accountId=...
```

Open each link in your browser and authorize the connection.

### 6.5 Verify Connections

```bash
# Ask Claude:
"Check my connected accounts"
```

Claude will verify via `list_zapier_connections` and `inspect_zapier_actions`.

Expected output:
```
Gmail: Connected as your.email@gmail.com
LinkedIn: Connected as Your Name (profile)
```

---

## Step 7: Test Gmail Integration

### 7.1 Read Inbox

```bash
# Ask Claude:
"Show me my recent inbox emails from the last 7 days"
```

Claude will call `execute_zapier_read_action` using the `gmail_find_email` tool with a query like:
```
in:inbox newer_than:7d -category:promotions
```

### 7.2 Send an Email

```bash
# Ask Claude:
"Send an email to friend@email.com with subject 'Testing Zapier MCP' and body 'This was sent from Claude Code through Zapier!'"
```

Claude will call `execute_zapier_write_action` using the `gmail_send_email` tool.

### 7.3 Send with CC/BCC

```bash
# Ask Claude:
"Send the same email but CC another@email.com"
```

### 7.4 Search for Specific Emails

```bash
# Ask Claude:
"Find emails from LinkedIn or about job applications"
```

---

## Step 8: Test LinkedIn Integration

### 8.1 Verify Profile

```bash
# Ask Claude:
"Show me my LinkedIn profile"
```

Claude will call the LinkedIn API via `execute_zapier_read_action` using the `linkedin_make_api_get_request` tool.

### 8.2 Available LinkedIn Actions

LinkedIn via Zapier supports:
- **Create Share Update** — Post text updates to your feed
- **Create Company Update** — Post to a Company Page
- **Make API GET Request** — Read LinkedIn API data
- **Make API Mutating Request** — POST/PUT/DELETE to LinkedIn API

---

## Step 9: Create & Post to LinkedIn

### 9.1 Draft the Post

```bash
# Ask Claude:
"Draft a LinkedIn post about our automation stack"
```

Claude will generate a post. Review it and request changes:

```
"Make it more technical — add the architecture details"
"Add hashtags: #DevOps #MCP #LearnWithPranav"
```

### 9.2 Post to LinkedIn

```bash
# Once satisfied:
"Post it to LinkedIn"
```

Claude calls `execute_zapier_write_action` with `action: "share"` on the LinkedIn API.

You'll receive a confirmation with the post URL:
```
https://www.linkedin.com/feed/update/urn:li:share:XXXXXXXXXX/
```

---

## What We Achieved

### ✅ Connected Stack
| Layer | Technology | Status |
|-------|-----------|--------|
| Terminal | Claude Code CLI | ✅ |
| Model Router | OpenRouter | ✅ |
| AI Model | DeepSeek V4 Flash | ✅ |
| API Bridge | Zapier MCP Server | ✅ |
| Email | Gmail (OAuth 2.0) | ✅ |
| Professional Network | LinkedIn (OAuth 2.0) | ✅ |

### ✅ Actions Performed
1. ✅ **Gmail Read** — Scanned 21 inbox emails from last 7 days
2. ✅ **Gmail Write** — Sent email with subject, body, and CC support
3. ✅ **LinkedIn Read** — Verified profile via REST API
4. ✅ **LinkedIn Write** — Published a post to LinkedIn feed
5. ✅ **Cross-app workflow** — Read from Gmail, composed context-aware content, posted to LinkedIn

### ✅ Key Learnings
- MCP (Model Context Protocol) enables universal API orchestration
- No custom glue code required — just describe what you want
- Model-agnostic: swap providers through OpenRouter without changing architecture
- Local-first: runs on your machine, not a cloud IDE
- 7,000+ apps accessible through the same interface

---

## Real-World Use Cases

### 🚨 Incident Response Automation
```
"Check PagerDuty alerts, pull related CloudWatch logs,
summarize in a Slack thread, and create a Jira ticket"
```

### ☁️ Infrastructure-as-Chat
```
"Scale down all dev EKS clusters after 7 PM IST,
take a snapshot of prod RDS, and email the report"
```

### 📋 Daily Ops Briefing
```
"Read my Gmail for urgent emails, check GitHub PRs needing review,
check PagerDuty on-call schedule, and summarize everything in Slack"
```

### 🔄 Cross-Tool Workflows
```
"When a new lead comes in via HubSpot, add them to Google Sheets,
send a welcome email, and create a Slack notification"
```

### 📊 Monitoring Copilot
```
"Find the last 5 deployments, check if any rolled back,
compare with the incident timeline, and send findings to the team chat"
```

---

## Troubleshooting

### Claude Code won't start
```bash
# Reinstall
npm uninstall -g @anthropic-ai/claude-code
npm install -g @anthropic-ai/claude-code

# Check Node version
node --version  # Must be v18+
```

### OpenRouter connection fails
```bash
# Verify API key
curl -H "Authorization: Bearer $OPENROUTER_API_KEY" \
     https://openrouter.ai/api/v1/auth/key

# Test model access
curl -X POST https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "deepseek/deepseek-v4-flash", "messages": [{"role": "user", "content": "Hello"}]}'
```

### Zapier MCP auth fails
- Run `/mcp` inside Claude Code to restart the auth flow
- If you see "Invalid state parameter," close the tab and try again
- On headless servers, use the callback URL method:
  1. Open the auth URL in a browser
  2. When the redirect fails, copy the URL from the address bar
  3. Paste it back into the terminal

### Gmail actions return "connection not found"
```bash
# List connections to find the correct connection ID
"List my Gmail connections"
```
Then use the returned `connection_id` in subsequent calls.

### LinkedIn API version error
```bash
# Include a version header
"Try with LinkedIn-Version: 202611"
```

### Permission errors
If Claude Code asks for permission repeatedly:
```bash
# Check settings.json for tool permissions
# Add allow rules for common tools (curl, npm, git, etc.)
```

---

## Resources

| Resource | Link |
|----------|------|
| Claude Code Docs | https://docs.anthropic.com/en/docs/claude-code/overview |
| OpenRouter | https://openrouter.ai |
| Zapier MCP | https://zapier.com/mcp |
| MCP Specification | https://modelcontextprotocol.io |
| DeepSeek Models | https://openrouter.ai/models/deepseek/deepseek-v4-flash |
| Gmail API | https://developers.google.com/gmail/api |
| LinkedIn API | https://learn.microsoft.com/en-us/linkedin/ |

---

## 🏁 Final Words

This stack represents a fundamental shift in how we interact with SaaS tools:

> **From:** Write code → Configure auth → Handle errors → Test → Deploy
>
> **To:** Describe what you want → It happens

The MCP protocol, combined with universal API bridges like Zapier and model-agnostic routers like OpenRouter, turns your terminal into a control plane for every tool you use.

**What will you automate first?**

---

*Built with ❤️ using Claude Code + OpenRouter + DeepSeek V4 Flash + Zapier MCP*

*📅 Created: November 22, 2026*

*🔗 Follow: [#LearnWithPranav](https://linkedin.com/in/pranav-jambare)*