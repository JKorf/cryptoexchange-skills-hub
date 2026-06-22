# CoinW.Net Safety

Use this file before generating CoinW account, trading, transfer, withdrawal, futures, leverage, margin, or credential code.

## Credentials

- Never hardcode real API keys or secrets.
- Use placeholders, environment variables, user secrets, or the consuming project's configuration pattern.
- Public market data examples should omit credentials.
- Private REST and private websocket examples need `CoinWCredentials`.

## Read-Only First

Prefer read-only calls before write calls:

- Spot market data: `SpotApi.ExchangeData.GetTickersAsync`, `GetSymbolsAsync`, `GetOrderBookAsync`
- Spot account: `SpotApi.Account.GetBalancesDetailsAsync`
- Futures market data: `FuturesApi.ExchangeData.GetTickerAsync`, `GetSymbolsAsync`, `GetOrderBookAsync`
- Futures positions: `FuturesApi.Trading.GetPositionsAsync`

## Trading

- Always check `Success` before using order ids or returned data.
- Prefer examples that place conservative limit orders and then cancel them.
- Do not generate market orders unless the user explicitly asks for immediate execution.
- Validate symbol rules, minimum quantity, price precision, quantity unit, fees, and account balances before production orders.
- CoinW spot symbols commonly use `BTC_USDT`; do not silently substitute compact symbols like `BTCUSDT`.

## Futures

Futures calls can create leveraged exposure or change account-level settings.

- Make leverage, margin type, and quantity unit explicit.
- `SetMarginModeAsync` changes account futures configuration.
- `PlaceOrderAsync` opens positions; use `ClosePositionAsync`, `CloseAllPositionsAsync`, or `ClosePositionsByClientOrderIdAsync` to close positions.
- Confirm whether the user wants long or short exposure before generating order placement code.
- Confirm whether quantity is contracts, base asset, or quote asset before using `QuantityUnit`.
- Treat take-profit, stop-loss, trailing TP/SL, reverse position, and adjust margin calls as trading writes.

## Transfers And Withdrawals

- `TransferAsync` moves funds between account types.
- `WithdrawAsync` moves funds off exchange or internally depending on parameters.
- Require explicit user intent, asset, network, address, memo/tag rules, and amount before generating withdrawal code.
- Prefer read-only deposit address and history examples unless the user specifically asks to move funds.

## Websockets

- Keep handlers fast; offload heavy work to a queue or channel.
- Always check the `WebSocketResult<UpdateSubscription>` before assuming a subscription exists.
- Unsubscribe during shutdown with `UnsubscribeAsync` or `UnsubscribeAllAsync`.
- Private streams require credentials and can expose sensitive balances, orders, and positions.

## Error Handling

- REST errors are returned in `HttpResult.Error`; do not rely on exceptions for API failures.
- Retry only when `result.Error?.IsTransient == true`.
- Do not retry bad credentials, permission errors, invalid symbols, insufficient balances, invalid quantity, invalid leverage, or rejected orders without changing the request.
- For batch futures orders, check the outer `HttpResult` and then inspect each inner `CallResult<CoinWBatchResult>`.
