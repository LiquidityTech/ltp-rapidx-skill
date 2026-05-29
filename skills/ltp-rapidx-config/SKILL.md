---
name: ltp-rapidx-config
description: Use when an agent needs to install or configure RapidX CLI/MCP access, set production LTP credentials, locate the agent workspace MCP config, review integration, discover tools, or run read-only self-checks.
---

# RapidX Config

Use this skill for setup and integration review only. Use `ltp-rapidx-trading` for account, market, order, position, algo, and live trading workflows.

## Scope

- Configure the published RapidX CLI package as the single runtime entrypoint.
- Configure MCP by launching `rapidx mcp serve` from the agent's own workspace MCP config.
- Classify whether this agent host is `MCP_READY`, `CLI_ONLY_READY`, or `NOT_VERIFIED`.
- Verify real tool availability with read-only calls.
- Produce an integration review with masked credentials and actual evidence.

Do not describe how to install this skill inside the skill itself. Assume the skill has already been installed by the agent host.

## Workspace First

Before changing MCP config, identify the agent host workspace that will run RapidX:

1. State the active agent workspace path from session/runtime context.
2. Identify the workspace-local MCP config file read by this same agent host.
3. If the MCP config path is ambiguous, inspect the confirmed workspace for existing MCP settings.
4. If still ambiguous, ask the user which MCP config file this agent should edit.
5. Add or update `mcpServers.rapidx` only after the workspace and config path are known.

Never assume the source repository root, filesystem root, or a global home config is the right target.

## Credential Intake

Ask whether the user wants to provide credentials as a user-provided chat secret. This is the default path for non-programmers, but state the risk first: even protected chat-secret flows are controlled by the agent host and may be subject to that host's retention, access, or collaboration settings.

If the agent host has a dedicated chat-secret UI, ask the user to create three secrets with the exact names `LTP_ACCESS_KEY`, `LTP_SECRET_KEY`, and `LTP_API_HOST`. If the host has no chat-secret UI, ask whether the user wants the agent to write masked-reference placeholders into MCP config or whether they prefer to set local environment variables manually.

Offer alternatives when the user wants stronger isolation:

- Local shell environment variables.
- Enterprise or OS secret manager referenced by the MCP host.

Rules:

- Required variables are `LTP_ACCESS_KEY`, `LTP_SECRET_KEY`, and `LTP_API_HOST`.
- Default production host is `https://api.liquiditytech.com`.
- Do not use `LTP_BASE_URL`, `RAPIDX_BASE_URL`, or `RAPIDX_PORTFOLIO_*`.
- Do not ask for or echo complete keys in normal chat text when a chat-secret mechanism is available.
- Never print full keys in logs, config review, test output, or evidence. Use masked values only.

## CLI Install

RapidX is published as an npm CLI package. Configure the GitHub Packages registry, then install the CLI:

```bash
npm config set @liquiditytech:registry https://npm.pkg.github.com
npm install -g @liquiditytech/rapidx-cli
```

If global install is not allowed, install in the confirmed agent workspace and use the workspace-local executable path in MCP config:

```bash
npm install @liquiditytech/rapidx-cli
./node_modules/.bin/rapidx --version
```

Verify the installed CLI:

```bash
rapidx --version
rapidx schema --json
rapidx self-check --read-only --json
```

For CLI-only agents, use direct `rapidx ... --json` commands. Do not create temporary bridge scripts, directory-changing shell chains, or shell command chaining for MCP access.

## Runtime Path Selection

Classify the agent after CLI install and, when available, MCP configuration. Do not classify from agent product name or config file existence alone.

Before install, only choose a candidate path:

- If the host exposes workspace MCP config plus native tool discovery/call surfaces, attempt the MCP path.
- If the host only exposes shell, exec, or terminal commands, use the CLI-only path.
- If uncertain, install CLI first, run CLI self-check, and keep MCP as `NOT_VERIFIED` until MCP tools are actually callable.

After install/config, set one status:

- `CLI_READY`: `rapidx --version` and `rapidx schema --json` pass.
- `MCP_READY`: `CLI_READY`, `initialize` returns `serverInfo.name=rapidx`, `tools/list` shows 33 `rapidx/...` tools, and `rapidx/tools` plus `rapidx/self-check` can be called as MCP tools.
- `CLI_ONLY_READY`: `CLI_READY`, but the host cannot configure, discover, or call MCP tools.
- `NOT_VERIFIED`: no real invocation evidence, or only a config file was edited.

Writing `mcpServers.rapidx` is only an attempted MCP setup. It is not proof of MCP readiness. If MCP is not `MCP_READY`, mark MCP as `NOT_VERIFIED` and use direct `rapidx ... --json` commands.

## MCP Config

MCP is started by the CLI. Add this server to the agent workspace MCP config:

```json
{
  "mcpServers": {
    "rapidx": {
      "command": "rapidx",
      "args": ["mcp", "serve"],
      "env": {
        "LTP_ACCESS_KEY": "<user-provided-secret-or-env-reference>",
        "LTP_SECRET_KEY": "<user-provided-secret-or-env-reference>",
        "LTP_API_HOST": "https://api.liquiditytech.com"
      }
    }
  }
}
```

The MCP server command should be `rapidx` with args `["mcp", "serve"]` when `rapidx` is on the MCP host PATH. If PATH is not guaranteed, use the absolute path to the installed `rapidx` executable as `command` and keep args as `["mcp", "serve"]`. Do not point MCP tools at one-off CLI commands and do not add shell script wrappers.

## Expected MCP Tools

Healthy MCP discovery exposes 33 tools:

```text
Discovery: rapidx/tools, rapidx/self-check
Market:    rapidx/market/get-ticker, rapidx/market/get-orderbook, rapidx/market/get-klines,
           rapidx/market/get-funding-rate, rapidx/market/get-mark-price,
           rapidx/market/get-symbol-info, rapidx/market/get-open-interest
Account:   rapidx/account/overview, rapidx/account/balance, rapidx/account/set-position-mode
Trade:     rapidx/trade/preview, rapidx/trade/verify-live
Order:     rapidx/order/preview, rapidx/order/place-preview,
           rapidx/order/amend-preview, rapidx/order/cancel-preview,
           rapidx/order/place, rapidx/order/amend, rapidx/order/cancel,
           rapidx/order/get, rapidx/order/list, rapidx/order/history
Position:  rapidx/position/list, rapidx/position/history,
           rapidx/position/close, rapidx/position/set-leverage
Algo:      rapidx/algo/place, rapidx/algo/amend, rapidx/algo/cancel, rapidx/algo/list
Compat:    rapidx/trading-verification
```

Legacy snake_case names such as `get_ticker`, `place_order`, or `list_positions` indicate a stale integration and should not be used.

## Read-Only Self-Check

The self-check proves the configured runtime is real. Do not simulate results, invent balances, or claim success from documentation alone.

Run the quick check:

1. Confirm `CLI_READY` with `rapidx --version` and `rapidx schema --json`.
2. If attempting MCP, discover tools through the MCP host and confirm the 33-tool inventory.
3. Call `rapidx/self-check` with read-only scope when the host supports tool invocation.
4. Call one public market route, preferably `rapidx/market/get-ticker` for `BINANCE_PERP_BTC_USDT`.
5. Call read routes for account overview, portfolio balance, open orders, positions, and algo orders.

If the host cannot invoke MCP tools yet, run equivalent CLI read-only checks and mark MCP tool invocation as `NOT_VERIFIED`; do not convert CLI success into MCP success.

Run the deeper review when asked for integration review or self-validation:

```text
1. rapidx/tools
2. rapidx/self-check
3. rapidx/market/get-ticker
4. rapidx/market/get-orderbook
5. rapidx/market/get-klines
6. rapidx/market/get-funding-rate
7. rapidx/market/get-mark-price
8. rapidx/market/get-symbol-info
9. rapidx/market/get-open-interest
10. rapidx/account/overview
11. rapidx/account/balance with mode="portfolio"
12. rapidx/account/balance with mode="account" only to classify key scope
13. rapidx/order/list
14. rapidx/order/history
15. rapidx/order/get with a deliberately nonexistent self-check order id
16. rapidx/position/list
17. rapidx/position/history
18. rapidx/algo/list
```

`mode="account"` may return a real permission or key-scope error for portfolio-scoped credentials. Treat that as `EXPECTED_ERROR`, not as a failed portfolio integration.

## Result Classes

- `PASS`: actual tool or command returned a successful real response.
- `EXPECTED_ERROR`: route is live and returned a real business, permission, unsupported-mode, or deliberate not-found error.
- `FAIL`: tool is missing, startup/auth/network failed, response is malformed, or a required call timed out.
- `NOT_VERIFIED`: the agent could not invoke the tool or the user declined credentials.

Every row must include `toolOrCommandEvidence` or equivalent observed code/message evidence. Empty order, position, or history lists are `PASS` if the response is real and well formed.

## Integration Review Output

Return this structure when asked to review setup:

```markdown
# RapidX Integration Review

## Verdict
- status: PASS / PARTIAL / FAIL / NOT_VERIFIED
- runtime path: MCP_READY / CLI_ONLY_READY / NOT_VERIFIED
- main issues:

## Workspace And Config
- agent workspace:
- MCP config path:
- MCP command: rapidx mcp serve
- CLI package:
- host:
- credentials: configured and masked / missing / not verified

## Tool Discovery
- expected MCP tools: 33
- actual MCP tools:
- missing tools:
- legacy tools found:

## Read-Only Checks
| check | result | evidence |
| --- | --- | --- |

## Required Fixes
- ...
```

Switch to `ltp-rapidx-trading` for any write verification or live trading test.
