# LTP RapidX Skills

RapidX agent skills for configuring and using the RapidX CLI and MCP server.

## Skills

- `ltp-rapidx-config`: install/configure RapidX CLI, configure `rapidx mcp serve`, set credentials, discover tools, and run read-only integration review.
- `ltp-rapidx-trading`: use RapidX MCP or CLI for market/account reads, preview-first writes, order management, positions, algo orders, and explicit live verification.

## Install

You can install RapidX skills in two ways:

- Run the install commands yourself.
- Send the install instructions to your agent and let the agent install them into its supported skill location.

### Option A: run commands yourself

Use this option when you are comfortable running terminal commands in the target agent workspace.

Start by identifying the agent host, then use the matching install path.

| Agent | Recommended install path | Skill locations or discovery path |
|-------|--------------------------|-----------------------------------|
| Codex | `$skill-installer` in Codex, or `npx skills add ... -a codex`. | `$skill-installer` installs into `$CODEX_HOME/skills`, default `~/.codex/skills`. Workspace skills may use `.agents/skills/`; admin installs may use `/etc/codex/skills`. |
| Claude Code | Copy to `.claude/skills/<name>/SKILL.md` or `~/.claude/skills/<name>/SKILL.md`, or use `npx skills add ... -a claude-code`. | `.claude/skills/`, `~/.claude/skills/`. |
| Cursor | `npx skills add ... -a cursor`, or copy to `.agents/skills/`, `.cursor/skills/`, or `~/.cursor/skills/`. | `.agents/skills/`, `.cursor/skills/`, `~/.cursor/skills/`. |
| Gemini CLI | Use `npx skills add ... -a gemini-cli` only if your environment supports the general skills installer. Current verified Gemini CLI builds expose `gemini extensions`, not `gemini skills`. | If native skills are unavailable, use RapidX CLI or MCP directly. |
| OpenCode | Copy to `.opencode/skills/`, `~/.config/opencode/skills/`, `.claude/skills/`, or `.agents/skills/`; or use `npx skills add ... -a opencode`. | Native, Claude-compatible, and agent-compatible paths. |
| OpenClaw | Install the published ClawHub skills by slug with `openclaw skills install <slug>`. | Workspace: `<workspace>/skills`; project-agent: `<workspace>/.agents/skills`; user: `~/.agents/skills`; managed: `~/.openclaw/skills`. |
| Hermes Agent | Install each skill by GitHub path with `hermes skills install LiquidityTech/ltp-rapidx-skill/skills/<name>`. | Default: `~/.hermes/skills/`; external skill dirs are also supported. |

## Codex

```bash
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config \
  --skill ltp-rapidx-trading \
  -a codex -y
```

Codex also supports installing from another repo through `$skill-installer`. Ask Codex to install both skills from `https://github.com/LiquidityTech/ltp-rapidx-skill`.

## Claude Code

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

## Cursor

```bash
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config \
  --skill ltp-rapidx-trading \
  -a cursor -y
```

## Gemini CLI

```bash
npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config \
  --skill ltp-rapidx-trading \
  -a gemini-cli -y
```

Use this only when your Gemini CLI environment supports Agent Skills through the general skills installer. The verified local Gemini CLI command surface exposes `gemini extensions`, not `gemini skills`.

## OpenCode

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

## OpenClaw

```bash
openclaw skills install ltp-rapidx-config
openclaw skills install ltp-rapidx-trading
```

The recommended OpenClaw path is ClawHub slug installation. For local development, clone this repository and install `./skills/ltp-rapidx-config` and `./skills/ltp-rapidx-trading` as local skills.

## Hermes Agent

```bash
hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-config
hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-trading
```

## Manual Installation

Clone the repository:

```bash
git clone https://github.com/LiquidityTech/ltp-rapidx-skill.git
```

Copy both directories into the target agent's skill directory:

```text
skills/ltp-rapidx-config
skills/ltp-rapidx-trading
```

Reload or restart the agent after copying the skills.

Use the SSH URL only when the repository is private or the agent host already has a configured SSH deploy key.

### Option B: send this to your agent

Use this option when you want the agent to install RapidX skills itself. Choose the section for your agent host and send only that block to the agent.

Codex:

```text
Install RapidX skills with Codex skill-installer from:
https://github.com/LiquidityTech/ltp-rapidx-skill

Install these two paths:
- skills/ltp-rapidx-config
- skills/ltp-rapidx-trading

After installation, restart Codex if needed, then load and follow the installed `ltp-rapidx-config` skill. It is not a shell command. Use the agent host's user-provided secret mechanism for API keys when available. Do not print or store full keys in public files, logs, screenshots, or normal chat output.
```

Claude Code:

```text
Install RapidX skills for Claude Code:

npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config --skill ltp-rapidx-trading -a claude-code -y

If the installer is unavailable, install both skill folders under .claude/skills/. After installation, load and follow the installed `ltp-rapidx-config` skill. It is not a shell command.
```

Cursor:

```text
Install RapidX skills for Cursor:

npx skills add https://github.com/LiquidityTech/ltp-rapidx-skill.git --skill ltp-rapidx-config --skill ltp-rapidx-trading -a cursor -y

After installation, load and follow the installed `ltp-rapidx-config` skill. It is not a shell command.
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

After installation, load and follow the installed `ltp-rapidx-config` skill. It is not a shell command.
```

OpenClaw:

```text
Install RapidX skills from ClawHub:

openclaw skills install ltp-rapidx-config
openclaw skills install ltp-rapidx-trading

After installation, load and follow the installed `ltp-rapidx-config` skill. It is not a shell command.
```

Hermes Agent:

```text
Install RapidX skills for Hermes:

hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-config
hermes skills install LiquidityTech/ltp-rapidx-skill/skills/ltp-rapidx-trading

After installation, load and follow the installed `ltp-rapidx-config` skill. It is not a shell command.
```

## Expected Runtime

The skills guide the agent to install the npm CLI package and configure MCP with:

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

MCP is provided by the CLI command `rapidx mcp serve`; this repository contains skills only.

For existing installations, run `rapidx update check --json` or call `rapidx/update/check` through MCP to detect CLI, MCP schema, and skills update advice. Reinstall these GitHub-distributed skills when the update result sets `skillsUpdateRecommended=true`.

## Repository Layout

```text
skills/
  ltp-rapidx-config/
    SKILL.md
  ltp-rapidx-trading/
    SKILL.md
skills-manifest.json
```

This top-level `skills/` layout is intentional so `npx skills add` can discover the skills without requiring users to know internal package paths.

## License

MIT
