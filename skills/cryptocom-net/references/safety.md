# CryptoCom.Net Safety

Use this file before generating Crypto.com account, trading, margin, derivatives, staking, withdrawal, transfer, leverage, or credential code.

## Credentials

- Never hardcode real API keys or secrets.
- Use placeholders, environment variables, user secrets, or the consuming project's configuration pattern.
- Public market data examples should omit credentials.
- Private REST calls, private socket streams, and socket request methods need `CryptoComCredentials`.

## Read-Only First

Prefer read-only calls before write calls:

- Public market data: `ExchangeApi.ExchangeData.GetTickersAsync`, `GetSymbolsAsync`, `GetOrderBookAsync`
- Account inspection: `ExchangeApi.Account.GetBalancesAsync`, `GetAccountSettingsAsync`, `GetFeeRatesAsync`
- Trading inspection: `ExchangeApi.Trading.GetOpenOrdersAsync`, `GetPositionsAsync`, `GetUserTradesAsync`
- Funding inspection: `GetDepositHistoryAsync`, `GetWithdrawalHistoryAsync`, `GetDepositAddressesAsync`
- Staking inspection: `ExchangeApi.Staking.GetStakingSymbolsAsync`, `GetStakingPositionsAsync`, `GetStakingRewardHistoryAsync`

## Trading

- Always check `Success` before using order ids or returned data.
- Prefer examples that place conservative limit orders and then cancel them.
- Do not generate market orders unless the user explicitly asks for immediate execution.
- Validate instrument names, tick sizes, quantity tick sizes, price decimals, fees, and account balances before production orders.
- Crypto.com spot symbols commonly use `BTC_USDT`; do not silently substitute compact symbols like `BTCUSDT`.
- Trigger, OCO, margin, isolated margin, leverage, and reduce-only parameters all change order semantics; make them explicit.

## Derivatives And Margin

Derivative and margin calls can create leveraged exposure.

- CryptoCom.Net uses `ExchangeApi.Trading` for derivatives; there is no separate futures client root.
- Confirm the exact derivative instrument name from `GetSymbolsAsync()` before trading.
- Confirm whether an order should open, increase, reduce, or close exposure.
- Use `ClosePositionAsync` when the user asks to close a derivative position.
- Treat `SetAccountLeverageAsync`, `SetAccountSettingsAsync`, `CreateIsolatedMarginTransferAsync`, and `SetIsolatedMarginLeverageAsync` as risk-changing writes.

## Staking And Conversion

- `StakeAsync`, `UnstakeAsync`, and `ConvertAsync` can lock, unlock, convert, or move funds.
- Prefer read-only staking position and reward examples unless the user explicitly asks for staking writes.
- Require explicit symbol, quantity, expected conversion rate, and slippage tolerance for conversion code.

## Transfers And Withdrawals

- `WithdrawAsync` moves funds off exchange.
- Require explicit user intent, asset, network, address, memo/tag rules, and amount before generating withdrawal code.
- Prefer read-only deposit address and history examples unless the user specifically asks to move funds.

## Websockets

- Keep handlers fast; offload heavy work to a queue or channel.
- Always check the `WebSocketResult<UpdateSubscription>` before assuming a subscription exists.
- Socket API request methods return `QueryResult<T>` or `QueryResult`; check `Success` before using `Data`.
- Unsubscribe during shutdown with `UnsubscribeAsync` or `UnsubscribeAllAsync`.
- Private streams expose sensitive balances, orders, trades, and positions.

## Error Handling

- REST errors are returned in `HttpResult.Error`; socket request errors are returned in `QueryResult.Error`.
- Retry only when `result.Error?.IsTransient == true`.
- Do not retry bad credentials, permission errors, invalid symbols, invalid precision, insufficient balances, invalid leverage, unknown order, or rejected orders without changing the request.
- For batch order methods, check the outer result and then inspect each inner `CallResult<T>`.
