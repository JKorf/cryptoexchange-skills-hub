# GateIo.Net Safety

Use this file before generating Gate.io credential, account, transfer, withdrawal, trading, margin, unified account, futures, leverage, or websocket code.

## Credentials

- Use `GateIoCredentials("API_KEY", "API_SECRET")`.
- Prefer environment variables, secret stores, or dependency-injected options.
- Never hardcode real API keys, secrets, wallet addresses, or memo/tag values.
- Public market data examples should avoid credentials unless private endpoints are needed.
- Private account, trading, withdrawal, transfer, and authenticated websocket calls require credentials with the correct Gate.io permissions.

## Result Handling

- REST methods return `HttpResult<T>` or `HttpResult`.
- Websocket subscriptions return `WebSocketResult<UpdateSubscription>`.
- Websocket request/order methods return `QueryResult<T>` or `QueryResult`.
- Shared non-I/O helpers can return `ExchangeCallResult<T>`.
- Always check `Success` before using `Data`.
- Use `Error.IsTransient` for retry decisions. Do not retry validation, insufficient balance, permission, leverage, symbol, or sizing errors blindly.

## Symbols And Settlement Assets

- Native Gate.io symbols use underscores, for example `ETH_USDT`.
- Do not generate Binance-style `ETHUSDT` or OKX-style `ETH-USDT` for native GateIo methods.
- Futures methods usually require `settlementAsset`, for example `"usdt"`, before `contract`.
- Validate contract metadata with `PerpetualFuturesApi.ExchangeData.GetContractAsync(settlementAsset, contract)` before sizing futures orders.

## Spot Trading

- Prefer read-only examples unless the user explicitly asks for trading.
- Prefer limit-order examples with prices away from the current market for examples that place real orders.
- Include cancellation cleanup when demonstrating live order placement.
- For market spot buys, Gate.io uses quote-asset quantity. State this explicitly.
- Do not assume every optional spot parameter applies to every account type.
- Trigger orders have separate trigger account types and trigger parameters; do not simplify them into normal spot orders.

## Margin And Unified Account

- Margin and unified account operations can borrow funds, repay loans, transfer assets, and change risk.
- Be explicit when using margin account types or unified account borrow/repay methods.
- Do not generate borrow, repay, leverage, or transfer actions unless the user asks for them or they are required by the task.
- When generating transfer examples, name the source account, target account, asset, quantity, and settlement asset where relevant.

## Perpetual Futures

- Futures orders create leveraged exposure. Say so in generated examples when relevant.
- Futures order `quantity` is an integer number of contracts, not a decimal base-asset quantity.
- Read contract metadata for multiplier, min/max size, and precision before order sizing.
- `price: 0m` with `TimeInForce.ImmediateOrCancel` is the Gate.io market-style futures pattern.
- Use `reduceOnly: true` in close-position examples to avoid flipping position direction.
- Setting leverage and margin mode persists until changed; do not hide this effect.
- Dual position mode and margin mode are account/contract state changes. Generate them only when requested.

## Withdrawals And Transfers

- Withdrawal code should be generated only when explicitly requested.
- Validate network, address, memo/tag, amount, and fee assumptions before sending withdrawals.
- Prefer showing deposit-address lookup and withdrawal-history examples over live withdrawals.
- Transfer examples can move funds between accounts and affect available collateral; make account types and settlement assets explicit.

## Websockets

- Keep handlers fast. Offload heavier work to a queue, channel, or background service.
- Check subscription `Success` before assuming a stream is live.
- Always unsubscribe on shutdown with `UnsubscribeAsync` or `UnsubscribeAllAsync`.
- Authenticated futures user streams often require `userId` plus settlement asset.
- Socket order request methods return `QueryResult<T>` and can place/cancel live orders. Treat them like trading endpoints.

## SharedApis

- Use SharedApis when the user needs exchange-agnostic code.
- Use native GateIo APIs for Gate.io-specific details such as `RebateApi`, `AlphaApi`, unified account operations, settlement-asset routing, and detailed trigger-order parameters.
- Do not mix native GateIo request/model types with SharedApis request/model types.
