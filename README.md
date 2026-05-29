# LTP RapidX Skills

RapidX agent skills for configuring and using the RapidX CLI and MCP server.

## Skills

- `ltp-rapidx-config`: install/configure RapidX CLI, configure `rapidx mcp serve`, set credentials, discover tools, and run read-only integration review.
- `ltp-rapidx-trading`: use RapidX MCP or CLI for market/account reads, preview-first writes, order management, positions, algo orders, and explicit live verification.

## Install With npx skills

If the repository is public, install both skills into the current Codex workspace:

```bash
npx skills add LiquidityTech/ltp-rapidx-skill \
  --skill ltp-rapidx-config \
  --skill ltp-rapidx-trading \
  -a codex -y
```

If the repository is private, use the SSH URL so Git can use your SSH key:

```bash
npx skills add git@github.com:LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config \
  --skill ltp-rapidx-trading \
  -a codex -y
```

Install globally for Codex:

```bash
npx skills add git@github.com:LiquidityTech/ltp-rapidx-skill.git \
  --skill ltp-rapidx-config \
  --skill ltp-rapidx-trading \
  -a codex -g -y
```

Install for another supported agent by changing `-a codex` to that agent name.

## Expected Runtime

The skills guide the agent to install the npm CLI package and configure MCP with:

```json
{
  "mcpServers": {
    "ltp-rapidx": {
      "command": "rapidx",
      "args": ["mcp", "serve"],
      "env": {
        "LTP_ACCESS_KEY": "<secret>",
        "LTP_SECRET_KEY": "<secret>",
        "LTP_API_HOST": "https://api.liquiditytech.com"
      }
    }
  }
}
```

MCP is provided by the CLI command `rapidx mcp serve`; this repository contains skills only.

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
