# Coinbase.Net Safety

Apply these guardrails when generating Coinbase account, trading, futures/perpetuals, portfolio transfer, deposit, withdrawal, convert, or credential code.

## Credentials

- Use `CoinbaseCredentials("API_KEY_NAME", "PRIVATE_KEY")` placeholders or environment variables.
- Normalize escaped private-key newlines with `.Replace("\\n", "\n")` when reading keys from environment variables.
- Do not print API key names, private keys, signatures, or raw authenticated request payloads.
- Do not store credentials in sample source files, markdown, logs, screenshots, or test snapshots.

## Read-Only First

Prefer read-only calls for examples and diagnostics:

- `AdvancedTradeApi.ExchangeData.GetSymbolAsync`
- `AdvancedTradeApi.ExchangeData.GetKlinesAsync`
- `AdvancedTradeApi.ExchangeData.GetOrderBookAsync`
- `AdvancedTradeApi.Account.GetAccountsAsync`
- `AdvancedTradeApi.Account.GetPortfoliosAsync`
- `AdvancedTradeApi.Account.GetFeeInfoAsync`
- `AdvancedTradeApi.Trading.GetOrdersAsync`
- `AdvancedTradeApi.Trading.GetUserTradesAsync`
- `AdvancedTradeApi.Trading.GetFuturesPositionsAsync`
- `ExchangeApi.ExchangeData.GetSymbolsAsync`

## Trading And Futures

- Treat Coinbase order placement as live unless explicitly dry-run gated.
- Keep examples dry-run by default; mirror `COINBASE_EXAMPLE_PLACE_ORDER=true` for sample order placement.
- Use conservative limit-order examples unless the user explicitly asks for market orders.
- Check both the outer `HttpResult.Success` and `CoinbaseOrderResult.Success` after `PlaceOrderAsync`.
- Do not choose product id, quantity, price, portfolio id, account id, or private key configuration silently for production workflows.
- Explain that `ClosePositionAsync`, futures/perpetual settings, portfolio allocation, and portfolio transfer operations can change account risk.
- Prefer examples that cancel test orders after placement.

## Transfers, Deposits, Withdrawals

- Treat `TransferPortfolioFundsAsync`, `AllocatePortfolioAsync`, `WithdrawAsync`, `WithdrawCryptoAsync`, `DepositAsync`, `CommitConvertTradeAsync`, and deposit-address creation as fund-moving or account-state-changing operations.
- Require explicit user intent before generating write operations that move funds.
- Keep withdrawal examples read-only unless the user specifically asks for withdrawal execution and provides the intended account id, asset, payment method or crypto address, network/tag if needed, and amount.

## Websocket Subscriptions

- Keep handlers fast and non-blocking.
- Include `UnsubscribeAsync` or `UnsubscribeAllAsync` cleanup in examples.
- Private socket subscriptions require credentials and may reveal balances, positions, orders, and fills; do not log sensitive account details unless requested.
