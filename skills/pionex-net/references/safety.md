# Pionex.Net Safety

Read this file before generating Pionex account, trading, cancel-all, private websocket, tracker, user-client-provider, credential, retry, or lifecycle code.

## Capability Boundary

- Install `Pionex.Net`.
- Current Pionex.Net exposes Spot only.
- Do not generate futures, perpetual, margin, options, bot-management, or derivatives methods.
- Do not invent direct ticker or kline websocket methods.
- Do not invent a listen-key workflow.
- Only `PionexEnvironment.Live` is built in; do not invent testnet.

## Credentials

- Use `PionexCredentials(key, secret)`.
- Never request, log, commit, or echo real secrets.
- Prefer injected configuration, environment variables, or a secrets provider.
- Give credentials only the exchange permissions needed.
- Do not add a passphrase parameter.

## Results and Retries

- REST returns `HttpResult<T>` or `HttpResult`.
- Websocket subscriptions return `WebSocketResult<UpdateSubscription>`.
- Check `Success` before `Data`.
- Retry only transient errors with bounded backoff, jitter, cancellation, and an attempt limit.
- Do not blindly retry authentication failures, validation failures, insufficient balance, or rejected orders.
- A transport failure can leave order outcome uncertain. Reconcile with a unique client order ID or order queries before retrying placement.

## Symbols and Validation

- Native symbols use `BASE_QUOTE`, such as `BTC_USDT`.
- Validate symbol existence and `Enable`.
- Validate quantity and quote precision, minimum and maximum quantities, current price bounds, and balance.
- Do not treat a deliberately distant limit price as a safety mechanism.
- Use Pionex enums instead of free-form strings.

## Live Trading

- Pionex.Net has no test-order endpoint.
- `PlaceOrderAsync` submits a live order.
- `CancelOrderAsync` and `CancelAllOrdersAsync` mutate live account state.
- Mark live endpoint examples clearly.
- Avoid executing live examples during validation.
- Market buys use `quoteQuantity`; market sells use `quantity`.
- Confirm order side, type, quantity semantics, symbol, and price before submission.
- Prefer read-only account or order-query examples when live mutation is unnecessary.

## Websockets

- Configure credentials directly on `PionexSocketClient` for private streams.
- Keep handlers fast; offload heavy work to a queue or channel.
- Check each subscription result independently.
- Store successful `UpdateSubscription` values.
- Unsubscribe with `UnsubscribeAsync` or `UnsubscribeAllAsync` during shutdown.
- Use order-book depth at or below the documented maximum of 100.

## Clients, Order Books, and Trackers

- Reuse clients or use dependency injection.
- Do not instantiate a new client per request.
- Create managed local books through `IPionexOrderBookFactory`.
- Start and stop local books according to their lifecycle and handle failed starts.
- Keep long-lived order books and trackers owned by application services.
- Clear cached user clients when credentials change.
- Treat user identifiers as application data; validate them and do not expose secrets through identifiers or logs.

## SharedApis

- Use native Pionex APIs for Pionex-specific metadata and models.
- Use SharedApis for portable exchange-agnostic workflows.
- Do not mix native `PionexTicker.ClosePrice` with shared `SharedSpotTicker.LastPrice`.
- Use `Discover()` before assuming optional shared features.
- Do not claim shared futures, positions, funding, withdrawal, or deposit support.
