---
name: ltp-rapidx-trading
description: Use when an agent needs to operate RapidX through MCP or CLI for account reads, market reads, order preview, order submit/amend/cancel, position management, algo orders, or explicit live trading verification.
---

# RapidX Trading

Use this skill after `ltp-rapidx-config` has confirmed RapidX CLI/MCP access. Prefer MCP tools when the agent host supports MCP. Use direct CLI commands only when MCP is unavailable.

## Non-Negotiable Rules

- Do not fake query or trading results. Every claim must come from an actual MCP tool or `rapidx ... --json` response, and final summaries must include `toolOrCommandEvidence` or equivalent observed evidence.
- Do not use shell bridge scripts, temporary JavaScript scripts, directory-changing shell chains, or chained shell invocations.
- Treat all trade-write tools as real production actions.
- Never submit a write without preview evidence and explicit user consent for that specific write.
- Use `confirmation.submitToken` from the preview response as the submit `continueConsentId`.
- Keep business parameters unchanged between preview and submit. If symbol, side, quantity, price, order id, leverage, or mode changes, create a new preview.
- If a write times out or the result is uncertain, query state before retrying.
- Never echo secrets.

## Current MCP Surface

Use `ltp-rapidx/tools` for the authoritative runtime schema. Current normal-use tool names are:

```text
Market:   ltp-rapidx/market/get-ticker, ltp-rapidx/market/get-orderbook,
          ltp-rapidx/market/get-klines, ltp-rapidx/market/get-funding-rate,
          ltp-rapidx/market/get-mark-price, ltp-rapidx/market/get-symbol-info,
          ltp-rapidx/market/get-open-interest
Account:  ltp-rapidx/account/overview, ltp-rapidx/account/balance,
          ltp-rapidx/account/set-position-mode
Trade:    ltp-rapidx/trade/preview, ltp-rapidx/trade/verify-live
Order:    ltp-rapidx/order/place-preview, ltp-rapidx/order/amend-preview,
          ltp-rapidx/order/cancel-preview, ltp-rapidx/order/place,
          ltp-rapidx/order/amend, ltp-rapidx/order/cancel,
          ltp-rapidx/order/get, ltp-rapidx/order/list, ltp-rapidx/order/history
Position: ltp-rapidx/position/list, ltp-rapidx/position/history,
          ltp-rapidx/position/close, ltp-rapidx/position/set-leverage
Algo:     ltp-rapidx/algo/place, ltp-rapidx/algo/amend,
          ltp-rapidx/algo/cancel, ltp-rapidx/algo/list
```

`ltp-rapidx/order/preview` and `ltp-rapidx/trading-verification` are compatibility tools. Prefer `ltp-rapidx/order/place-preview` and `ltp-rapidx/trade/verify-live` in new workflows.

## Read Workflow

Before making trading decisions, refresh state:

```text
1. ltp-rapidx/account/overview
2. ltp-rapidx/account/balance with mode="portfolio"
3. ltp-rapidx/order/list
4. ltp-rapidx/position/list
5. ltp-rapidx/algo/list
```

For a symbol, refresh market data:

```text
1. ltp-rapidx/market/get-symbol-info
2. ltp-rapidx/market/get-ticker
3. ltp-rapidx/market/get-orderbook
4. ltp-rapidx/market/get-mark-price
5. ltp-rapidx/market/get-klines
6. ltp-rapidx/market/get-funding-rate      # PERP only
7. ltp-rapidx/market/get-open-interest     # PERP only
```

Use symbol format `{EXCHANGE}_{TYPE}_{BASE}_{QUOTE}`, for example `BINANCE_PERP_BTC_USDT` or `OKX_PERP_BTC_USDT`. For OKX perpetuals, quantity is contract count; inspect symbol info before placing or amending orders.

## Preview Then Submit

All writes use this pattern:

1. Call the write-specific preview tool.
2. Read `previewId` and `confirmation.submitToken`.
3. Show the user the actual preview summary, max notional, order id/client order id, and risk-relevant fields.
4. Ask for explicit consent for this one write.
5. Submit the target write with the same business parameters plus `previewId` and `continueConsentId=<confirmation.submitToken>`.
6. Query resulting state with the relevant read tool.

If the preview response does not include `confirmation.submitToken`, do not submit the write. Re-run preview with the current CLI/MCP runtime or report the integration as stale.

Order placement:

```text
ltp-rapidx/order/place-preview
ltp-rapidx/order/place
ltp-rapidx/order/get or ltp-rapidx/order/list
```

Order amend:

```text
ltp-rapidx/order/amend-preview
ltp-rapidx/order/amend
ltp-rapidx/order/get or ltp-rapidx/order/list
```

Order cancel:

```text
ltp-rapidx/order/cancel-preview
ltp-rapidx/order/cancel
ltp-rapidx/order/list
```

Non-order writes:

```text
ltp-rapidx/trade/preview with targetCapabilityId
target tool, such as ltp-rapidx/position/set-leverage
matching read-back tool
```

Common `targetCapabilityId` values are `position.set-leverage`, `position.close`, `account.set-position-mode`, `algo.place`, `algo.amend`, and `algo.cancel`.

## Order Rules

- LIMIT order: requires quantity and price.
- MARKET order: use only when the user explicitly authorizes market execution.
- SPOT MARKET by quote amount uses quote quantity semantics.
- PERP writes are leverage and margin sensitive.
- Use a stable `clientOrderId` when the schema accepts one so status can be checked after a timeout.
- Do not infer fills from placement. Confirm through `order/get`, `order/list`, `order/history`, or positions.

## Algo Orders

Use preview/submit for `ltp-rapidx/algo/place`, `ltp-rapidx/algo/amend`, and `ltp-rapidx/algo/cancel`.

Before placing TPSL or conditional orders:

- Confirm target symbol, side, quantity, trigger price, stop/take-profit intent, and position side if hedge mode is used.
- For TPSL, require at least one valid take-profit or stop-loss trigger.
- After submit, verify through `ltp-rapidx/algo/list`.

## Position And Account Risk Writes

Use separate explicit consent for each:

- `ltp-rapidx/position/set-leverage` changes future risk for the symbol.
- `ltp-rapidx/account/set-position-mode` changes account position mode and can affect existing workflows.
- `ltp-rapidx/position/close` is a real close-position action. Verify current position first.

Do not test these writes as part of ordinary setup.

## Live Trading Verification

Use `ltp-rapidx/trade/verify-live` only when the user explicitly asks for a small real-trade verification and authorizes symbol, exchange, amount cap, cleanup behavior, and test window.

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

If any step cannot be verified, return `NOT_VERIFIED`, `EXPECTED_ERROR`, or `FAIL` with observed evidence. Do not call it successful without real evidence.

## CLI Fallback

When MCP is unavailable, use direct CLI equivalents with `--json` and the same preview/submit discipline:

```bash
rapidx order place-preview --input '{"symbol":"BINANCE_PERP_BTC_USDT","side":"BUY","orderType":"LIMIT","price":"65000","quantity":"0.001","maxNotional":"100","clientOrderId":"example-001"}' --json
rapidx order place --input '{"symbol":"BINANCE_PERP_BTC_USDT","side":"BUY","orderType":"LIMIT","price":"65000","quantity":"0.001","maxNotional":"100","clientOrderId":"example-001","previewId":"<previewId>","continueConsentId":"<confirmation.submitToken>"}' --json
rapidx trade preview --input '{"targetCapabilityId":"position.set-leverage","params":{"symbol":"BINANCE_PERP_BTC_USDT","leverage":5}}' --json
rapidx trade verify-live --input '{"symbol":"BINANCE_PERP_BTC_USDT","side":"BUY","maxNotional":"100","clientOrderId":"verify-001","explicitUserConsent":true}' --json
```

Avoid shell chaining and wrapper scripts. Run commands from the agent workspace or use absolute paths supported by the host.

## Final Answer

For trading work, state:

- Which real tools or commands were called.
- Which account/order/position facts were verified.
- Whether the final state is open, filled, cancelled, closed, unchanged, or not verified.
- Any remaining action the user must explicitly authorize.
