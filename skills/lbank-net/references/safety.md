# LBank.Net Safety

Read this file before generating LBank account, order, cancellation, withdrawal, private websocket, tracker, user-client-provider, credential, retry, or lifecycle code.

## Capability Boundary

- Install `LBank.Net`.
- Current LBank.Net exposes Spot only.
- Do not generate futures, derivatives, margin, options, or unified trading roots.
- Public socket streams use v3 by default; select v2 only when specifically required.
- Only `LBankEnvironment.Live` is built in; do not invent testnet.

## Credentials

- Use `LBankCredentials(key, secret)` for HMAC.
- Use `WithRSAXml(...)` or compatible-target `WithRSAPem(...)` only for RSA authentication.
- Never request, log, commit, or echo real secrets or RSA private keys.
- Prefer injected configuration, environment variables, or a secrets provider.
- Give credentials only the exchange permissions needed.
- Do not add a passphrase parameter.

## Results and Retries

- REST returns `HttpResult<T>` or `HttpResult`.
- Websocket subscriptions return `WebSocketResult<UpdateSubscription>`.
- Check `Success` before `Data`.
- Retry only transient errors with bounded backoff, jitter, cancellation, and an attempt limit.
- Do not blindly retry authentication, validation, insufficient-balance, withdrawal, or rejected-order failures.
- A transport failure can leave an order or withdrawal outcome uncertain. Reconcile using a unique client ID and read-only queries before retrying a mutation.

## Symbols and Validation

- Default native symbols to lowercase `base_quote`, such as `eth_usdt`.
- Follow explicit per-method interface documentation where an upstream endpoint uses a different representation.
- Query current symbol metadata and validate quantity and price precision, minimum quantity, balance, and order type.
- Use `KlineInterval` for REST and `StreamKlineInterval` for sockets.
- Supply required kline `limit` and `afterTime` arguments.
- Use LBank's combined `OrderType`; do not invent separate side or time-in-force arguments.

## Live Trading

- `PlaceOrderAsync` submits a live order.
- `CancelOrderAsync` and `CancelAllOrdersAsync` mutate live account state.
- Mark live endpoint examples clearly.
- Avoid executing live examples during validation.
- Market-buy `quantity` is in the quote asset.
- Confirm order type, quantity semantics, symbol, price, and balance before submission.
- The upstream order API may not be enabled for every account; preserve all error handling.
- Prefer read-only order queries when mutation is unnecessary.

## Withdrawals and Wallets

- `WithdrawAsync` moves real funds.
- Preserve the requested address, asset, network, memo/tag, quantity, fee, notes, name, client ID, and internal-transfer setting.
- Validate the address and network combination from current exchange metadata.
- Treat memo/tag omission as potentially fund-losing for assets that require one.
- Require explicit user intent before generating or executing a live withdrawal workflow.
- Never retry an ambiguous withdrawal without reconciling withdrawal history by client ID.

## Websockets

- Private order and balance streams require an existing listen key or credentials when passing `null`.
- Let the library manage the listen-key lifecycle when using `null`; do not create a duplicate refresh loop.
- Use order-book depth `10`, `50`, or `100`.
- Keep handlers fast; offload heavy work to a queue or channel.
- Check each subscription result independently.
- Store successful `UpdateSubscription` values.
- Unsubscribe with `UnsubscribeAsync` or `UnsubscribeAllAsync` during shutdown.

## Clients, Order Books, and Trackers

- Reuse clients or use dependency injection.
- Do not instantiate a new client per request.
- Create managed local books through `ILBankOrderBookFactory`.
- Start and stop local books according to their lifecycle and handle failed starts.
- Keep long-lived order books and trackers owned by application services.
- Clear cached user clients when credentials change.
- Treat user identifiers as application data; validate them and do not expose secrets through identifiers or logs.

## Shared API V2

- Use native LBank APIs for exchange-specific metadata, wallet operations, and models.
- Use SharedApis for portable exchange-agnostic workflows.
- Do not mix native ticker wrappers with shared ticker models.
- Use `Discover()` before assuming optional shared features.
- Do not call the shared all-assets operation; LBank requires a specified asset.
- Do not claim shared futures, positions, funding, margin, or options support.
