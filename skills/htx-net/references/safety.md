# HTX.Net Safety

Use this file before generating HTX credential, account, transfer, withdrawal, API-key, margin, futures, leverage, trading, tracker, or websocket code.

## Credentials

- Install `JKorf.HTX.Net`, not `HTX.Net`.
- Use `HTXCredentials("API_KEY", "API_SECRET")` for HMAC credentials.
- Use `new HTXCredentials().WithEd25519(key, privateKey)` only when Ed25519 is explicitly needed.
- Prefer environment variables, secret stores, or dependency-injected options.
- Never hardcode real API keys, secrets, private keys, wallet addresses, or address tags.
- Public market data examples should avoid credentials unless private endpoints are needed.

## Result Handling

- REST methods return `HttpResult<T>` or `HttpResult`.
- Websocket subscriptions return `WebSocketResult<UpdateSubscription>`.
- Spot socket query/order methods return `QueryResult<T>` or `QueryResult`.
- Shared non-I/O helpers can return `ExchangeCallResult<T>`.
- Always check `Success` before using `Data`.
- Use `Error.IsTransient` for retry decisions. Do not retry validation, signature, insufficient balance, margin-mode, leverage, or sizing errors blindly.

## Symbols And Account Ids

- Native spot symbols use no separator, for example `ETHUSDT`.
- Native USDT futures contract codes use hyphens, for example `ETH-USDT`.
- Do not generate Gate.io-style `ETH_USDT` or OKX-style spot symbols for HTX native spot methods.
- Spot balances and spot order placement require an account id. Fetch it with `SpotApi.Account.GetAccountsAsync()` and handle the no-account case.
- SharedApis can format symbols, but native methods expect HTX-native symbols or contract codes.

## Spot Trading

- Prefer read-only examples unless the user explicitly asks for trading.
- Prefer limit-order examples with conservative prices for examples that place real spot orders.
- Include cancellation cleanup when demonstrating live order placement.
- Use `PlaceConditionalOrderAsync` for conditional-order workflows; do not pretend normal orders and conditional orders are the same surface.
- Margin spot orders and normal spot orders use different methods and parameters.

## Margin

- Spot margin methods can borrow funds, repay loans, and move collateral.
- Cross and isolated margin have separate transfer, loan, and balance concepts.
- Do not generate borrow, repay, or margin transfer actions unless the user asks for them or they are required by the task.
- When generating margin examples, name the account type, symbol or margin account, asset, and quantity explicitly.

## USDT Futures

- Futures orders create leveraged exposure. Say so in generated examples when relevant.
- USDT futures order `quantity` is contract quantity.
- Cross-margin and isolated-margin futures have separate method families. Choose one deliberately.
- Setting leverage persists until changed; do not hide this effect.
- Use `Offset.Open` for opening examples and `Offset.Close`, `reduceOnly: true`, or the close-position endpoints for closing examples.
- Prefer `CloseCrossMarginPositionAsync` or `CloseIsolatedMarginPositionAsync` for close-position examples when that matches the user request.
- V5 futures is a native `UsdtFuturesV5Api` surface; use `UsdtFuturesApi.SharedClient` for cross-exchange futures code.

## Withdrawals And Transfers

- Withdrawal code should be generated only when explicitly requested.
- Validate network, address, address tag, amount, and fee assumptions before sending withdrawals.
- Prefer deposit-address, withdrawal-quota, and withdrawal-history examples over live withdrawals.
- Transfer examples can move funds between accounts and affect collateral. Make source, destination, asset, quantity, and margin account explicit.

## Websockets And Trackers

- Keep handlers fast. Offload heavier work to a queue, channel, or background service.
- Check subscription `Success` before assuming a stream is live.
- Always unsubscribe on shutdown with `UnsubscribeAsync` or `UnsubscribeAllAsync`.
- Spot socket query/order methods return `QueryResult<T>` and can place/cancel live orders. Treat them like trading endpoints.
- For long-running user data services, consider `IHTXTrackerFactory` instead of ad hoc subscription management.

## SharedApis

- Use SharedApis when the user needs exchange-agnostic code.
- Use native HTX APIs for HTX-specific details such as spot account ids, margin loans, V5 futures, detailed trigger/TP-SL orders, and HTX-specific websocket streams.
- Do not mix native HTX request/model types with SharedApis request/model types.
