---
name: ltp-rapidx-trading
description: Use when an agent needs to operate RapidX through MCP or CLI for account reads, market reads, order preview, order submit/amend/cancel, position management, algo orders, or explicit live trading verification.
---

# RapidX Trading

Use this skill after `ltp-rapidx-config` has confirmed the runtime path as `MCP_READY` or `CLI_ONLY_READY`. Prefer MCP tools only when the agent host is `MCP_READY`. Use direct CLI commands only when the confirmed path is `CLI_ONLY_READY`.

## Non-Negotiable Rules

- Do not fake query or trading results. Every claim must come from an actual MCP tool or `rapidx ... --json` response, and final summaries must include `toolOrCommandEvidence` or equivalent observed evidence.
- Do not use shell bridge scripts, temporary JavaScript scripts, directory-changing shell chains, or chained shell invocations.
- Treat all trade-write tools as real production actions.
- Never submit a write without preview evidence and explicit user consent for that specific write, unless the user has explicitly enabled RapidX automation mode for the current scope in chat.
- Use `confirmation.submitToken` from the preview response as the submit `continueConsentId`.
- Keep business parameters unchanged between preview and submit. If symbol, side, positionSide, quantity, price, order id, leverage, or mode changes, create a new preview.
- If a write times out or the result is uncertain, query state before retrying.
- Never echo secrets.

## Invocation Path

Before any trading workflow, read the latest integration review from `ltp-rapidx-config` or run that skill first.

- `MCP_READY`: use `rapidx/...` MCP tools and do not shell out to wrapper scripts.
- `CLI_ONLY_READY`: use direct `rapidx ... --json` commands and do not claim MCP tools were called.
- `NOT_VERIFIED` or only `CLI_READY`: stop and run config self-check before account, market, or trade workflows.

Do not switch paths during a task without new evidence. If an MCP call fails after `MCP_READY`, mark MCP degraded and verify state before retrying or falling back to CLI.

## Version Check

At the start of a trading session or before the first write in a session, check the cached release status once:

- `MCP_READY`: call `rapidx/update/check` or `rapidx/self-check` with `checkUpdates=true`.
- `CLI_ONLY_READY`: run `rapidx update check --json`.

Do not perform a fresh network update check before every trade submit. If the update result is `WRITE_BLOCKED` or `UPGRADE_REQUIRED`, stop all trade-write actions, upgrade the CLI, restart or reload the MCP host when applicable, and rerun self-check. If `skillsUpdateRecommended=true`, tell the user the skills should be reinstalled from GitHub, but do not block read-only work solely for that reason.

## Current MCP Surface

Use `rapidx/tools` for the authoritative runtime schema. It returns the tool list plus concrete `inputSchemas`; read the relevant input schema before constructing write inputs. Current normal-use tool names are:

```text
Market:   rapidx/market/get-ticker, rapidx/market/get-orderbook,
          rapidx/market/get-klines, rapidx/market/get-funding-rate,
          rapidx/market/get-mark-price, rapidx/market/get-symbol-info,
          rapidx/market/get-open-interest
Account:  rapidx/account/overview, rapidx/account/balance,
          rapidx/account/set-position-mode
Update:   rapidx/update/check
Trade:    rapidx/trade/preview, rapidx/trade/verify-live
Order:    rapidx/order/place-preview, rapidx/order/amend-preview,
          rapidx/order/cancel-preview, rapidx/order/place,
          rapidx/order/amend, rapidx/order/cancel,
          rapidx/order/get, rapidx/order/list, rapidx/order/history
Position: rapidx/position/list, rapidx/position/history,
          rapidx/position/close, rapidx/position/set-leverage
Algo:     rapidx/algo/place, rapidx/algo/amend,
          rapidx/algo/cancel, rapidx/algo/list
```

`rapidx/order/preview` and `rapidx/trading-verification` are compatibility tools. Prefer `rapidx/order/place-preview` and `rapidx/trade/verify-live` in new workflows.

## Read Workflow

Before making trading decisions, refresh state:

```text
1. rapidx/account/overview
2. rapidx/account/balance with mode="portfolio"
3. rapidx/order/list
4. rapidx/position/list
5. rapidx/algo/list
```

For a symbol, refresh market data:

```text
1. rapidx/market/get-symbol-info
2. rapidx/market/get-ticker
3. rapidx/market/get-orderbook
4. rapidx/market/get-mark-price
5. rapidx/market/get-klines
6. rapidx/market/get-funding-rate      # PERP only
7. rapidx/market/get-open-interest     # PERP only
```

Use RapidX symbol format `BINANCE_PERP_<BASE>_<QUOTE>`, for example `BINANCE_PERP_BTC_USDT` or `BINANCE_PERP_ETH_USDT`. `OKX_PERP_<BASE>_<QUOTE>` is supported for OKX perpetual instruments. If the user says an OKX swap symbol, `OKX_SWAP_<BASE>_<QUOTE>` is accepted as an input alias and normalizes to `OKX_PERP_<BASE>_<QUOTE>`. Market adapters may return `originalSymbol` for venue-native symbols such as `BTCUSDT` or `BTC-USDT-SWAP`.

Inspect symbol info before placing or amending orders.
For hedge-mode orders, pass `positionSide="LONG"` or `positionSide="SHORT"` in order placement, algo placement, set-leverage, or verify-live inputs when the schema exposes it. Do not call `rapidx/account/set-position-mode` just to choose an order side.

## Preview Then Submit

All writes use this pattern:

1. Call the write-specific preview tool.
2. Read `previewId` and `confirmation.submitToken`.
3. Show the user the actual `requestSummary`, `businessParams`, max notional, order id/client order id, and `riskNotes`.
4. Ask for explicit consent for this one write.
5. Submit the target write with the same business parameters plus `previewId` and `continueConsentId=<confirmation.submitToken>`.
6. Query resulting state with the relevant read tool.

If the preview response does not include `confirmation.submitToken`, do not submit the write. Re-run preview with the current CLI/MCP runtime or report the integration as stale.

Preview ids are runtime-local. Use MCP preview ids only with the same MCP server runtime. Use CLI preview ids only with the same CLI preview store. Do not cross-submit MCP preview ids through CLI, or CLI preview ids through MCP.

Automation mode still requires preview. Only use it when the user explicitly enables RapidX automation mode in chat and states the scope. Add `automationMode=true` and the user's exact `automationConsentText` to the preview input. If the preview returns automation details and `confirmation.submitToken`, the agent may submit that preview without asking for another per-order chat confirmation within the authorized scope. Do not invent `automationConsentText`.

`maxNotional` is a safety upper bound, not the target order amount. Before increasing quantity, amount, or notional to satisfy an exchange rule, check symbol `minNotional` and ask the user to confirm the new amount.

Order placement:

```text
rapidx/order/place-preview
rapidx/order/place
rapidx/order/get or rapidx/order/list
```

Order amend:

```text
rapidx/order/amend-preview
rapidx/order/amend
rapidx/order/get or rapidx/order/list
```

Order cancel:

```text
rapidx/order/cancel-preview
rapidx/order/cancel
rapidx/order/list
```

`rapidx/order/cancel` is asynchronous. If the result has `cancelAccepted=true` and `terminalStateConfirmed=false`, poll `rapidx/order/get` until `CANCELED`, `REJECTED`, `EXPIRED`, or timeout before claiming a final state.

Non-order writes:

```text
rapidx/trade/preview with targetCapabilityId
target tool, such as rapidx/position/set-leverage
matching read-back tool
```

Common `targetCapabilityId` values are `position.set-leverage`, `position.close`, `account.set-position-mode`, `algo.place`, `algo.amend`, and `algo.cancel`.

## Order Rules

- LIMIT order: requires quantity and price.
- MARKET order: allowed after preview and explicit user authorization. Treat it as immediate execution with possible slippage and no guaranteed fill price.
- SPOT MARKET by quote amount uses quote quantity semantics.
- PERP writes are leverage and margin sensitive.
- Hedge-mode order placement uses `positionSide="LONG"` or `positionSide="SHORT"` when needed.
- Use a stable `clientOrderId` when the schema accepts one so status can be checked after a timeout.
- Do not infer fills from placement. Confirm through `order/get`, `order/list`, `order/history`, or positions.
- If a requested order is below the symbol `minNotional`, do not auto-increase to the minimum. Ask the user to approve the revised amount first.
- Do not tell users that RapidX blocks all MARKET orders by default. Do not silently replace a requested MARKET order with a best-bid/best-ask LIMIT order.

## Algo Orders

Use preview/submit for `rapidx/algo/place`, `rapidx/algo/amend`, and `rapidx/algo/cancel`.

Before placing TPSL or conditional orders:

- Confirm target symbol, side, quantity when required, trigger price, stop/take-profit intent, and position side if hedge mode is used.
- For TPSL, require at least one valid take-profit or stop-loss trigger.
- `conditionType="ENTIRE_CLOSE_POSITION"` may use `orderType="MARKET"` without `quantity` or `amount`.
- After submit, verify through `rapidx/algo/list`.

## Position And Account Risk Writes

Use separate explicit consent for each:

- `rapidx/position/set-leverage` changes future risk for the symbol.
- `rapidx/account/set-position-mode` changes account position mode and can affect existing workflows. Use it only when the user explicitly asks to change account position mode.
- `rapidx/position/close` is a real close-position action. Verify current position first.

Do not pass `side` or `quantity` to `position.close`. The close-position API determines BUY or SELL from the current position and closes the target symbol/positionSide. In NET mode, closing a long behaves like SELL and closing a short behaves like BUY. Treat `position.close` as a market close unless the tool schema explicitly exposes another order type, and verify the result with `rapidx/position/list`. Use a reduce-only order flow for partial closes. If `order/get` later shows `reduceOnly=false`, do not treat that alone as a failed close; `position.close` uses the RapidX close-position API and the order readback may not echo the reduce-only intent.

Do not test these writes as part of ordinary setup.

## Live Trading Verification

Use `rapidx/trade/verify-live` only when the user explicitly asks for a small real-trade verification and authorizes symbol, exchange, amount cap, cleanup behavior, and test window. The tool input must include `acceptedRiskText` that names the exact symbol, side, positionSide when provided, maxNotional, real-order risk, and cancel cleanup behavior.

The verification must include:

```text
1. read-only self-check
2. market and symbol rule lookup
3. explicit user consent
4. internal preview
5. post-only or safely far-from-market limit submit
6. order query
7. amend when supported
8. cancel
9. cleanup check for open orders, positions, and algo orders
```

If any step cannot be verified, return `NOT_VERIFIED`, `EXPECTED_ERROR`, `INVALID_INPUT`, `BLOCKED`, `NOT_FOUND`, `PERMISSION_SCOPE_ERROR`, `BUSINESS_ERROR`, or `FAIL` with observed evidence. Do not call it successful without real evidence.

## CLI Fallback

When MCP is unavailable, use direct CLI equivalents with `--json` and the same preview/submit discipline:

```bash
rapidx order place-preview --input '{"symbol":"BINANCE_PERP_BTC_USDT","side":"BUY","orderType":"LIMIT","price":"65000","quantity":"0.001","maxNotional":"100","clientOrderId":"example-001"}' --json
rapidx order place --input '{"symbol":"BINANCE_PERP_BTC_USDT","side":"BUY","orderType":"LIMIT","price":"65000","quantity":"0.001","maxNotional":"100","clientOrderId":"example-001","previewId":"<previewId>","continueConsentId":"<confirmation.submitToken>"}' --json
rapidx trade preview --input '{"targetCapabilityId":"position.set-leverage","symbol":"BINANCE_PERP_BTC_USDT","leverage":5}' --json
rapidx trade verify-live --input '{"symbol":"BINANCE_PERP_BTC_USDT","side":"BUY","maxNotional":"100","clientOrderId":"verify-001","explicitUserConsent":true,"acceptedRiskText":"I authorize a real verification order for BINANCE_PERP_BTC_USDT BUY maxNotional 100 with cancel cleanup."}' --json
```

Avoid shell chaining and wrapper scripts. Run commands from the agent workspace or use absolute paths supported by the host.

## Final Answer

For trading work, state:

- Which real tools or commands were called.
- Which account/order/position facts were verified.
- Whether the final state is open, filled, cancelled, closed, unchanged, or not verified.
- Any remaining action the user must explicitly authorize.
