# BloFin.Net Safety

Apply these guardrails when generating BloFin account, futures, trading, transfer, withdrawal, or credential code.

## Credentials

- Use `BloFinCredentials("API_KEY", "API_SECRET", "PASSPHRASE")` placeholders or environment variables.
- Do not print API keys, secrets, passphrases, signatures, or raw authenticated request payloads.
- Do not store credentials in sample source files, markdown, logs, screenshots, or test snapshots.

## Read-Only First

Prefer read-only calls for examples and diagnostics:

- `FuturesApi.ExchangeData.GetSymbolsAsync`
- `FuturesApi.ExchangeData.GetTickersAsync`
- `FuturesApi.ExchangeData.GetOrderBookAsync`
- `AccountApi.GetAccountBalancesAsync`
- `FuturesApi.Account.GetBalancesAsync`
- `FuturesApi.Trading.GetPositionsAsync`
- `FuturesApi.Account.GetLeverageAsync`

## Trading And Futures

- Treat futures orders as real exposure.
- Use conservative limit-order examples unless the user explicitly asks for market orders.
- Do not choose leverage, margin mode, position mode, position side, quantity, or price silently for production workflows.
- Explain that `SetLeverageAsync`, `SetMarginModeAsync`, `SetPositionModeAsync`, `ClosePositionAsync`, and collateral or transfer operations can change account risk.
- Prefer examples that cancel test orders after placement.
- Check `Success` and `Error` on every result before reading `Data`.
- For batch operations returning `CallResult<T>[]`, inspect each item result after the outer `HttpResult` succeeds.

## Transfers, Deposits, Withdrawals

- Treat `TransferAsync` and withdrawal-related workflows as fund-moving operations.
- Require explicit user intent before generating write operations that move funds.
- Keep withdrawal examples read-only unless the user specifically asks for withdrawal execution and provides the intended asset, network, address, and amount.

## Websocket Subscriptions

- Keep handlers fast and non-blocking.
- Include `UnsubscribeAsync` cleanup in examples.
- Private socket subscriptions require credentials and may reveal balances, positions, and orders; do not log sensitive account details unless requested.
