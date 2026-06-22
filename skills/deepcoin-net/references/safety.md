# DeepCoin.Net Safety

Use this file before generating DeepCoin account, trading, swap/futures, leverage, withdrawal, listen-key, or credential code.

## Credentials

- Never hardcode real API keys, secrets, or passphrases.
- Use placeholders, environment variables, user secrets, or the consuming project's configuration pattern.
- Public market data examples should omit credentials.
- Private REST calls and private websocket user streams need `DeepCoinCredentials`.

## Read-Only First

Prefer read-only calls before write calls:

- Market data: `ExchangeApi.ExchangeData.GetTickersAsync`, `GetSymbolsAsync`, `GetOrderBookAsync`
- Account inspection: `ExchangeApi.Account.GetBalancesAsync`, `GetBillsAsync`
- Trading inspection: `ExchangeApi.Trading.GetOpenOrdersAsync`, `GetClosedOrdersAsync`, `GetPositionsAsync`
- Funding inspection: `GetDepositHistoryAsync`, `GetWithdrawHistoryAsync`

## Trading

- Always check `Success` before using order ids or returned data.
- Prefer examples that place conservative limit orders and then cancel them.
- Do not generate market orders unless the user explicitly asks for immediate execution.
- Validate symbol, market type, order type, margin mode, position side, quantity, price, and balance before production orders.
- DeepCoin spot symbols commonly use `ETH-USDT`; do not silently substitute compact symbols like `ETHUSDT`.
- DeepCoin swap symbols commonly use `ETH-USDT-SWAP`.
- `EditOrderAsync` is not supported for spot according to the source comment.

## Swaps And Leverage

Swap/futures calls can create leveraged exposure.

- Make `TradeMode`, `PositionType`, and `PositionSide` explicit.
- `SetLeverageAsync` changes risk configuration.
- Confirm whether the user wants long or short exposure before generating order placement code.
- For close examples, prefer the position-id and `reduceOnly: true` pattern shown in `references/usage.md`.
- Treat TP/SL trigger prices as trading writes.

## Withdrawals And Transfers

- Withdrawal history is supported through `GetWithdrawHistoryAsync`.
- Source transfer methods are commented out and marked not usable; do not generate transfer calls.
- Prefer read-only deposit/withdrawal history examples unless the user explicitly asks for fund movement.

## User Streams

- `StartUserStreamAsync` returns a listen key for private user streams.
- Use `KeepAliveUserStreamAsync` when managing a listen key manually.
- The socket `SubscribeToUserDataUpdatesAsync()` overload without a listen key can obtain and renew a listen key automatically.
- Private streams expose sensitive balances, orders, trades, account updates, trigger orders, and positions.

## Websockets

- Keep handlers fast; offload heavy work to a queue or channel.
- Always check the `WebSocketResult<UpdateSubscription>` before assuming a subscription exists.
- Unsubscribe during shutdown with `UnsubscribeAsync` or `UnsubscribeAllAsync`.

## Error Handling

- REST errors are returned in `HttpResult.Error`; websocket subscription errors are returned in `WebSocketResult.Error`.
- Retry only when `result.Error?.IsTransient == true`.
- Do not retry bad credentials, bad passphrase, permission errors, invalid symbols, invalid order modes, insufficient balances, invalid leverage, or rejected orders without changing the request.
