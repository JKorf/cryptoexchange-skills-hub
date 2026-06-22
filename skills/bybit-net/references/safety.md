# Bybit.Net Safety

Apply these guardrails when generating Bybit account, trading, derivatives, transfer, withdrawal, earn, loan, or credential code.

## Credentials

- Use `BybitCredentials("API_KEY", "API_SECRET")` placeholders or environment variables.
- Do not print API keys, secrets, signatures, or raw authenticated request payloads.
- Do not store credentials in sample source files, markdown, logs, screenshots, or test snapshots.

## Read-Only First

Prefer read-only calls for examples and diagnostics:

- `V5Api.ExchangeData.GetSpotTickersAsync`
- `V5Api.ExchangeData.GetLinearInverseTickersAsync`
- `V5Api.ExchangeData.GetOrderbookAsync`
- `V5Api.ExchangeData.GetKlinesAsync`
- `V5Api.Account.GetBalancesAsync`
- `V5Api.Trading.GetOrdersAsync`
- `V5Api.Trading.GetPositionsAsync`
- `V5Api.Account.GetFeeRateAsync`

## Trading And Derivatives

- Treat Bybit order placement as live unless explicitly dry-run gated.
- Keep examples dry-run by default; mirror `BYBIT_EXAMPLE_PLACE_ORDER=true` for sample order placement.
- Use conservative limit-order examples unless the user explicitly asks for market orders.
- Do not choose category, leverage, margin mode, position mode, position index, quantity, or price silently for production workflows.
- Explain that `SetLeverageAsync`, `SetMarginModeAsync`, `SwitchCrossIsolatedMarginAsync`, `SwitchPositionModeAsync`, `SetRiskLimitAsync`, `SetTradingStopAsync`, and `AddOrReduceMarginAsync` can change account risk.
- Prefer examples that cancel test orders after placement.
- Check `Success` and `Error` on every result before reading `Data`.
- For batch operations returning `CallResult<T>[]` or `BybitBatchResult<T>[]`, inspect each item result after the outer `HttpResult` succeeds.

## Transfers, Deposits, Withdrawals

- Treat `CreateInternalTransferAsync`, `CreateUniversalTransferAsync`, `WithdrawAsync`, convert confirmation, earn order placement, borrow, repay, and collateral adjustment as fund-moving operations.
- Require explicit user intent before generating write operations that move funds.
- Keep withdrawal examples read-only unless the user specifically asks for withdrawal execution and provides the intended asset, network, address, account type, and amount.

## Websocket Subscriptions

- Keep handlers fast and non-blocking.
- Include `UnsubscribeAsync` or `UnsubscribeAllAsync` cleanup in examples.
- Private socket subscriptions require credentials and may reveal balances, orders, positions, and user trades; do not log sensitive account details unless requested.
