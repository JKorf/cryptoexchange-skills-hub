# CoinEx.Net Safety

Apply these guardrails when generating CoinEx account, trading, futures, margin, transfer, withdrawal, or credential code.

## Credentials

- Use `CoinExCredentials("API_KEY", "API_SECRET")` placeholders or environment variables.
- Do not print API keys, secrets, signatures, or raw authenticated request payloads.
- Do not store credentials in sample source files, markdown, logs, screenshots, or test snapshots.

## Read-Only First

Prefer read-only calls for examples and diagnostics:

- `SpotApiV2.ExchangeData.GetTickersAsync`
- `SpotApiV2.ExchangeData.GetSymbolsAsync`
- `SpotApiV2.ExchangeData.GetOrderBookAsync`
- `SpotApiV2.Account.GetBalancesAsync`
- `SpotApiV2.Trading.GetOpenOrdersAsync`
- `SpotApiV2.Trading.GetUserTradesAsync`
- `FuturesApi.ExchangeData.GetTickersAsync`
- `FuturesApi.Account.GetBalancesAsync`
- `FuturesApi.Trading.GetPositionsAsync`

## Trading, Margin, And Futures

- Treat spot, margin, and futures order placement as live unless explicitly dry-run gated.
- Use conservative limit-order examples unless the user explicitly asks for market orders.
- Do not choose leverage, margin mode, account type, quantity, price, stop price, or client order id silently for production workflows.
- Query symbol metadata and round with `QuantityPrecision` and `PricePrecision` before placing production orders.
- Explain that `FuturesApi.Account.SetLeverageAsync`, `FuturesApi.Trading.ClosePositionAsync`, TP/SL, margin adjustment, margin borrow/repay, and transfers can change account risk or move funds.
- Prefer examples that cancel test orders after placement.
- Check `Success` and `Error` on every result before reading `Data`.
- For batch operations returning `CallResult<T>[]` or `CoinExBatchResult<T>[]`, inspect each item result after the outer `HttpResult` succeeds.

## Transfers, Deposits, Withdrawals

- Treat `TransferAsync`, `WithdrawAsync`, `CancelWithdrawalAsync`, `MarginBorrowAsync`, and `MarginRepayAsync` as fund-moving or liability-changing operations.
- Require explicit user intent before generating write operations that move funds or create liabilities.
- Keep withdrawal examples read-only unless the user specifically asks for withdrawal execution and provides the intended asset, network/address/memo, method, and amount.

## Websocket Subscriptions

- Keep handlers fast and non-blocking.
- Include `UnsubscribeAsync` or `UnsubscribeAllAsync` cleanup in examples.
- Private socket subscriptions require credentials and may reveal balances, orders, positions, and user trades; do not log sensitive account details unless requested.
