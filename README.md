# LTP RapidX Skills

Agent skills for installing, configuring, and using RapidX CLI and RapidX MCP.

Current release:

| Component | Version |
|---|---:|
| RapidX CLI / MCP | `1.0.38` |
| RapidX Skills | `1.0.13` |

## What This Repo Provides

- `ltp-rapidx-config`: guides an agent through RapidX CLI installation, credential setup, MCP configuration, tool discovery, upgrade checks, and read-only self-check.
- `ltp-rapidx-trading`: guides an agent through RapidX reads, preview-first writes, automation sessions, order lifecycle, positions, algo orders, and explicit live-trade verification.

The npm package is `@liquiditytech/rapidx-cli`. MCP is started by the CLI command `rapidx mcp serve`. This repository contains skills only.

After installing the skills, ask your agent to load and follow `ltp-rapidx-config`. `ltp-rapidx-config` is a skill name, not a shell command.

## Runtime Model

```text
Agent
  -> RapidX Skills
       -> install/configure @liquiditytech/rapidx-cli
       -> choose MCP-capable mode or CLI-only mode
       -> run self-check
  -> RapidX CLI
       -> one-shot commands
       -> rapidx mcp serve
  -> RapidX MCP tools
       -> structured tool calls for MCP-capable agents
```

## Install

You can install RapidX skills in two ways:

- Run the install commands yourself.
- Send the install instructions to your agent and let the agent install them into its supported skill location.

### Install Commands

Use the method that matches your agent host.

| Agent | Recommended install method |
|---|---|
| Codex | `$skill-installer`, or `npx skills add ... -a codex` |
| Claude Code | `npx skills add ... -a claude-code`, or copy to `.claude/skills/` |
| Cursor | `npx skills add ... -a cursor`, or copy to `.agents/skills/` / `.cursor/skills/` |
| Gemini CLI | `npx skills add ... -a gemini-cli` only when your environment supports the general skills installer |
| OpenCode | `npx skills add ... -a opencode`, or copy to OpenCode/Claude-compatible skill paths |
| OpenClaw | `openclaw skills install <slug>` from ClawHub |
| Hermes Agent | `hermes skills install LiquidityTech/ltp-rapidx-skill/skills/<name>` |

#### Codex

```bash
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config \
  --skill ltp-rapidx-trading \
  -a codex -y
```

Codex also supports installing from another repo through `$skill-installer`. Ask Codex to install both skills from `https://github.com/LiquidityTech/ltp-rapidx-skill`.

#### Claude Code

```bash
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config \
  --skill ltp-rapidx-trading \
  -a claude-code -y
```

Manual install paths:

```text
.claude/skills/ltp-rapidx-config/SKILL.md
.claude/skills/ltp-rapidx-trading/SKILL.md
~/.claude/skills/ltp-rapidx-config/SKILL.md
~/.claude/skills/ltp-rapidx-trading/SKILL.md
```

#### Cursor

```bash
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config \
  --skill ltp-rapidx-trading \
  -a cursor -y
```

#### Gemini CLI

```bash
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config \
  --skill ltp-rapidx-trading \
  -a gemini-cli -y
```

Use this only when your Gemini CLI environment supports Agent Skills through the general skills installer. If Agent Skills are unavailable, use RapidX CLI or MCP directly.

#### OpenCode

```bash
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config \
  --skill ltp-rapidx-trading \
  -a opencode -y
```

Manual install paths include:

```text
.opencode/skills/<name>/SKILL.md
~/.config/opencode/skills/<name>/SKILL.md
.claude/skills/<name>/SKILL.md
~/.claude/skills/<name>/SKILL.md
.agents/skills/<name>/SKILL.md
~/.agents/skills/<name>/SKILL.md
```

#### OpenClaw

```bash
openclaw skills install ltp-rapidx-config
openclaw skills install ltp-rapidx-trading
```

The recommended OpenClaw path is ClawHub slug installation. Current ClawHub skill version: `1.0.13`.

#### Hermes Agent

```bash
hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-config
hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-trading
```

#### Manual Installation

Clone the repository:

```bash
git clone https://github.com/LiquidityTech/ltp-rapidx-skill.git
```

Copy both directories into the target agent's supported skill directory:

```text
skills/ltp-rapidx-config
skills/ltp-rapidx-trading
```

Reload or restart the agent after copying the skills.

Use the SSH URL only when the repository is private or the agent host already has a configured SSH deploy key.

### Send This To Your Agent

Use this when you want the agent to install RapidX skills itself. Send only the block that matches your agent host.

Codex:

```text
Install RapidX skills with Codex skill-installer from:
https://github.com/LiquidityTech/ltp-rapidx-skill

Install these two paths:
- skills/ltp-rapidx-config
- skills/ltp-rapidx-trading

After installation, restart Codex if needed, then load and follow the installed `ltp-rapidx-config` skill. `ltp-rapidx-config` is not a shell command.
Use the agent host's user-provided secret mechanism for API keys when available. Do not print or store full keys in public files, logs, screenshots, or normal chat output.
```

Claude Code:

```text
Install RapidX skills for Claude Code:

npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config --skill ltp-rapidx-trading -a claude-code -y

If the installer is unavailable, install both skill folders under .claude/skills/. After installation, load and follow the installed `ltp-rapidx-config` skill. `ltp-rapidx-config` is not a shell command.
```

Cursor:

```text
Install RapidX skills for Cursor:

npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config --skill ltp-rapidx-trading -a cursor -y

After installation, load and follow the installed `ltp-rapidx-config` skill. `ltp-rapidx-config` is not a shell command.
```

Gemini CLI:

```text
Install RapidX skills for Gemini CLI only if this environment supports Agent Skills through the general skills installer:

npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config --skill ltp-rapidx-trading -a gemini-cli -y

If Agent Skills are unavailable, use RapidX CLI or MCP directly.
```

OpenCode:

```text
Install RapidX skills for OpenCode:

npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config --skill ltp-rapidx-trading -a opencode -y

After installation, load and follow the installed `ltp-rapidx-config` skill. `ltp-rapidx-config` is not a shell command.
```

OpenClaw:

```text
Install RapidX skills from ClawHub:

openclaw skills install ltp-rapidx-config
openclaw skills install ltp-rapidx-trading

After installation, load and follow the installed `ltp-rapidx-config` skill. `ltp-rapidx-config` is not a shell command.
```

Hermes Agent:

```text
Install RapidX skills for Hermes:

hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-config
hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-trading

After installation, load and follow the installed `ltp-rapidx-config` skill. `ltp-rapidx-config` is not a shell command.
```

## Expected Runtime

The config skill guides the agent to install the npm CLI package:

```bash
npm install -g @liquiditytech/rapidx-cli@latest
```

For MCP-capable agents, configure MCP with:

```json
{
  "mcpServers": {
    "rapidx": {
      "command": "rapidx",
      "args": ["mcp", "serve"],
      "env": {
        "LTP_ACCESS_KEY": "<secret>",
        "LTP_SECRET_KEY": "<secret>",
        "LTP_API_HOST": "<provided-api-host>"
      }
    }
  }
}
```

Required credentials:

```text
LTP_ACCESS_KEY
LTP_SECRET_KEY
LTP_API_HOST
```

`LTP_API_HOST` has no default. Use the API host provided by the environment or workspace owner.

## Check and Upgrade

For existing installations, run:

```bash
rapidx update check --json
```

Or call the MCP tool:

```text
rapidx/update/check
```

Upgrade when the result reports a newer CLI or skills version.

Recommended order:

1. Upgrade or reinstall RapidX skills.
2. Upgrade RapidX CLI.
3. Restart or reload the agent/MCP host.
4. Run `rapidx self-check --json`.

For CLI upgrade:

```bash
npm install -g @liquiditytech/rapidx-cli@latest
```

For skills upgrade, rerun the install command for your agent host or reinstall from ClawHub/GitHub.

## Self-Check

After installation, the agent should run:

```bash
rapidx self-check --json
```

For MCP-capable agents, the agent should also discover tools and run the MCP self-check tool:

```text
rapidx/tools
rapidx/self-check
```

Expected MCP tool count for this release: `46`.

For small live-trade verification, use:

```bash
rapidx trade verify-live --json
```

This submits a real order and requires explicit user consent. Do not use it as a routine health check.

## Repository Layout

```text
skills/
  ltp-rapidx-config/
    SKILL.md
    references/
  ltp-rapidx-trading/
    SKILL.md
    references/
releases/
  stable.json
skills-manifest.json
```

This top-level `skills/` layout is intentional so `npx skills add` can discover the skills without requiring users to know internal package paths.

## License

MIT
