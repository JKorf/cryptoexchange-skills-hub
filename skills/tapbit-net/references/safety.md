# Tapbit.Net Safety

Read this file before generating Tapbit credentials, account, order, batch, user-client-provider, retry, or tracker code.

## Capability Boundary

- Target the anticipated `Tapbit.Net` 1.0.0 package.
- Current Tapbit.Net exposes spot REST only.
- Do not generate `TapbitSocketClient`, websocket subscriptions, futures, margin, positions, leverage, funding, deposits, withdrawals, or transfers.
- Do not generate market-order placement or user-trade endpoints.
- Do not claim kline, public-trade, or managed local order-book factory support.
- Only `TapbitEnvironment.Live` is built in; do not invent testnet.

## Credentials

- Use `TapbitCredentials(key, secret)` without a passphrase.
- Never request, log, echo, or commit real secrets.
- Prefer injected configuration, environment variables, or a secrets provider.
- Use read-only keys unless trading permission is actually required.
- Clear cached multi-user clients after credential rotation.

## Result Handling And Retries

- Check `HttpResult<T>.Success` before reading `Data`.
- For batch calls, check the outer result and every `CallResult<T>` item.
- Retry only transient failures with bounded exponential backoff, jitter, cancellation, and an attempt limit.
- Do not retry authentication, validation, invalid-price, invalid-quantity, insufficient-balance, or unknown-order errors blindly.
- A timeout can leave placement outcome uncertain. Query and reconcile before repeating an order request.

## Symbols And Order Validation

- Native symbols use `BASE/QUOTE`, for example `BTC/USDT`.
- Query `GetSymbolAsync` before production placement.
- Validate price and quantity precision, minimum quantity, minimum notional, price-fluctuation bounds, current market price, side, and available balance.
- Native order IDs are `long`; do not pass symbol arguments to `GetOrderAsync` or `CancelOrderAsync`.
- Do not treat a distant limit price as a safety mechanism.

## Live Trading And Batches

- `PlaceOrderAsync` and `PlaceMultipleOrdersAsync` can create live limit orders.
- Tapbit.Net has no test-order endpoint.
- Cancellation mutates live order state.
- Batch success can be partial. Record successful IDs, report failed items, and do not retry the entire batch without reconciliation.
- Prefer read-only order and balance examples when live mutation is unnecessary.

## Trackers And Multi-User Clients

- Reuse clients or dependency injection rather than constructing a client per request.
- `CanCreateKlineTracker` and `CanCreateTradeTracker` return false; do not call their create methods.
- User spot tracking is REST polling, not websocket streaming.
- User tracker configuration must include `TrackedSymbols` and set `TrackTrades = false`.
- Keep polling intervals bounded and respect rate limits.
- Validate user identifiers and never place secrets in them or in logs.

## SharedApis

- Use native methods for Tapbit-specific fields and batch operations.
- Use SharedApis for portable spot workflows.
- Shared order placement supports limit/GTC only.
- Shared user-trade and order-trade methods are unsupported.
- Call `Discover()` before assuming an optional endpoint.
- Do not mix Tapbit-native models and enums with SharedApis request and response types.
