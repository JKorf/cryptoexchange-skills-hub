# XT.Net Safety

Use this file before generating credentials, trading, leverage, position, withdrawal, transfer, or private-stream code.

## Credentials And Results

- Private endpoints use `XTCredentials(key, secret)`; do not add a passphrase.
- Never hardcode real credentials, withdrawal addresses, order ids, or listen keys.
- Check `HttpResult` or `WebSocketResult` success before using data.
- Retry only transient failures, with bounded backoff and idempotency awareness.

## Symbols And Validation

- Spot native symbols are lowercase underscore values such as `eth_usdt`.
- Futures native symbols are uppercase underscore values such as `ETH_USDT`.
- Validate product, trading status, precision, minimum quantity/value, fees, and available balance.
- Avoid silently normalizing symbols because spot and futures casing differs.

## Spot Trading

- `PlaceOrderAsync` places a live order and requires `TimeInForce` plus `BusinessType`.
- Use `quoteQuantity` for spot market buys and `quantity` for market sells.
- Include cancellation cleanup in limit-order examples.
- Batch cancellation and cancel-all can affect multiple live orders.
- `BusinessType.Leverage` can create margin exposure.

## Futures Trading

- Futures orders and leverage changes create leveraged exposure.
- Make `PositionSide` explicit and verify account position mode.
- Use the correct REST product root: `UsdtFuturesApi` or `CoinFuturesApi`.
- Trigger and TP/SL orders need explicit trigger semantics and current-position validation.
- `CloseAllPositionsAsync()` closes all futures positions in scope; generate it only when explicitly requested and clearly label the impact.

## Funds

- Withdrawals and transfers move funds. Generate them only when explicitly requested.
- Validate asset, network, address, memo/tag, amount, fee, and source/destination account.
- Prefer balance, address, and history examples over fund movement.

## Websockets And Environment

- Public spot streams use lowercase symbols; futures streams use uppercase symbols.
- Spot private streams require a websocket token; futures private streams require a listen key.
- Credentialed overloads can manage token acquisition, but still require private API permissions.
- Keep handlers fast, observe subscription failures, and unsubscribe during shutdown.
- Current source exposes `XTEnvironment.Live` and custom environments, not a built-in testnet.

## SharedApis

- Use SharedApis for portable spot/futures workflows.
- Use native APIs for XT-specific account, token/listen-key, trigger, and detailed model behavior.
- Do not mix native XT models with SharedApis models.
