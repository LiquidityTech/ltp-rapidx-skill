# LTP RapidX Skills

Agent skills for installing, configuring, and using RapidX CLI and RapidX MCP.

Current release:

| Component | Version |
|---|---:|
| RapidX CLI / MCP | `1.0.40` |
| RapidX Skills | `1.0.16` |

## What This Repo Provides

- `ltp-rapidx-config`: guides an agent through RapidX CLI installation, credential setup, MCP configuration, tool discovery, upgrade checks, and read-only self-check.
- `ltp-rapidx-trading`: guides an agent through RapidX reads, preview-first writes, automation sessions, order lifecycle, positions, algo orders, and explicit live-trade verification.

The npm package is `@liquiditytech/rapidx-cli`. MCP is started by the CLI command `rapidx mcp serve`. This repository contains skills only.

After installing the skills, ask your agent to load and follow `ltp-rapidx-config`.

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

| Agent | Install method |
|---|---|
| Codex | Use `$skill-installer`, or use the full Codex command below |
| Claude Code | Use the full Claude Code command below, or copy to `.claude/skills/` |
| Cursor | Use the full Cursor command below, or copy to `.agents/skills/` / `.cursor/skills/` |
| Gemini CLI | Use the full Gemini CLI command below only when your environment supports the general skills installer |
| OpenCode | Use the full OpenCode command below, or copy to OpenCode/Claude-compatible skill paths |
| OpenClaw | Use the full OpenClaw commands below |
| Hermes Agent | Use the full Hermes commands below |

#### Codex

```bash
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config ltp-rapidx-trading \
  -a codex -y
```

Codex also supports installing from another repo through `$skill-installer`. Ask Codex to install both skills from `https://github.com/LiquidityTech/ltp-rapidx-skill`.

#### Claude Code

```bash
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config ltp-rapidx-trading \
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
  --skill ltp-rapidx-config ltp-rapidx-trading \
  -a cursor -y
```

#### Gemini CLI

```bash
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config ltp-rapidx-trading \
  -a gemini-cli -y
```

Use this only when your Gemini CLI environment supports Agent Skills through the general skills installer. If Agent Skills are unavailable, use RapidX CLI or MCP directly.

#### OpenCode

```bash
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config ltp-rapidx-trading \
  -a opencode -y
```

Manual install paths include:

```text
.opencode/skills/ltp-rapidx-config/SKILL.md
.opencode/skills/ltp-rapidx-trading/SKILL.md
~/.config/opencode/skills/ltp-rapidx-config/SKILL.md
~/.config/opencode/skills/ltp-rapidx-trading/SKILL.md
.agents/skills/ltp-rapidx-config/SKILL.md
.agents/skills/ltp-rapidx-trading/SKILL.md
```

#### OpenClaw

```bash
openclaw skills install ltp-rapidx-config
openclaw skills install ltp-rapidx-trading
```

The recommended OpenClaw path is ClawHub slug installation. Current ClawHub skill version: `1.0.16`.

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

For every agent host, use the host's user-provided secret mechanism for API keys when available. Do not print or store full keys in public files, logs, screenshots, or normal chat output.

Codex:

```text
Install RapidX skills for this Codex workspace.

Run:
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config ltp-rapidx-trading -a codex -y

After installation, follow ltp-rapidx-config first to install or upgrade @liquiditytech/rapidx-cli, configure credentials, configure MCP if supported, and run self-check.
After RapidX is verified, follow ltp-rapidx-trading for market, portfolio, order, position, algo, and automation workflows.
Use the agent host's user-provided secret mechanism for API keys when available. Do not print or store full keys in public files, logs, screenshots, or normal chat output.
```

Claude Code:

```text
Install RapidX skills for this Claude Code workspace.

Run:
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config ltp-rapidx-trading -a claude-code -y

If the installer is unavailable, install both skill folders under .claude/skills/.
After installation, load ltp-rapidx-config first, complete CLI/MCP configuration and self-check, then use ltp-rapidx-trading for RapidX operations.
Use the agent host's user-provided secret mechanism for API keys when available. Do not print or store full keys in public files, logs, screenshots, or normal chat output.
```

Cursor:

```text
Install RapidX skills for this Cursor workspace.

Run:
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config ltp-rapidx-trading -a cursor -y

Use ltp-rapidx-config first to set up RapidX CLI/MCP and verify access, then use ltp-rapidx-trading for trading workflows.
Use the agent host's user-provided secret mechanism for API keys when available. Do not print or store full keys in public files, logs, screenshots, or normal chat output.
```

Gemini CLI:

```text
Install RapidX skills for Gemini CLI only if this environment supports Agent Skills through the general skills installer:

Run:
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config ltp-rapidx-trading -a gemini-cli -y

If Agent Skills are unavailable, use @liquiditytech/rapidx-cli directly and run rapidx self-check --json before any trading workflow.
Use the agent host's user-provided secret mechanism for API keys when available. Do not print or store full keys in public files, logs, screenshots, or normal chat output.
```

OpenCode:

```text
Install RapidX skills for this OpenCode workspace.

Run:
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config ltp-rapidx-trading -a opencode -y

Follow ltp-rapidx-config before any RapidX query or trading action.
Use the agent host's user-provided secret mechanism for API keys when available. Do not print or store full keys in public files, logs, screenshots, or normal chat output.
```

OpenClaw:

```text
Install RapidX skills from ClawHub:

openclaw skills install ltp-rapidx-config
openclaw skills install ltp-rapidx-trading

After installation, follow ltp-rapidx-config first. Configure CLI/MCP access, run self-check, and only then use ltp-rapidx-trading.
Use the agent host's user-provided secret mechanism for API keys when available. Do not print or store full keys in public files, logs, screenshots, or normal chat output.
```

Hermes Agent:

```text
Install RapidX skills for Hermes:

hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-config
hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-trading

After installation, follow ltp-rapidx-config first, then ltp-rapidx-trading.
Use the agent host's user-provided secret mechanism for API keys when available. Do not print or store full keys in public files, logs, screenshots, or normal chat output.
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

Use the API host provided by the environment or workspace owner.

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

Upgrade order:

1. Upgrade or reinstall RapidX skills.
2. Upgrade RapidX CLI.
3. Restart or reload the agent/MCP host.
4. Run CLI self-check with `rapidx self-check --json`.
5. If MCP is enabled, run MCP self-check with `rapidx/self-check`.

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
rapidx trade verify-live --input '{
  "symbol": "BINANCE_PERP_BTC_USDT",
  "side": "BUY",
  "maxNotional": "100",
  "clientOrderId": "verify-live-001",
  "explicitUserConsent": true,
  "acceptedRiskText": "I authorize a real verification order for BINANCE_PERP_BTC_USDT BUY maxNotional 100 with cancel cleanup."
}' --json
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
