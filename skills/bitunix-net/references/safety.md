# Bitunix.Net Safety

Read this reference before generating Bitunix credential, account, withdrawal, transfer, trading, futures, leverage, margin/position mode, TP/SL, close-position, private socket, tracker, or multi-user client code.

## Credentials

- Private operations use `BitunixCredentials(key, secret)`; there is no passphrase.
- Prefer injected configuration, environment variables, or a secret store.
- Never hardcode or log real API keys, secrets, wallet addresses, memos/tags, transaction hashes, withdrawal IDs, client order IDs, or position IDs.
- Futures private sockets authenticate from configured credentials. Do not invent or expose a listen key.

## Result Handling And Retries

- REST returns `HttpResult<T>` or `HttpResult`.
- Futures subscriptions return `WebSocketResult<UpdateSubscription>`.
- Spot socket order queries return `QueryResult<T>` or `QueryResult`.
- Always check `Success` before reading `Data`.
- Retry only transient failures with bounded backoff, jitter, cancellation, and an attempt limit.
- Do not blindly retry validation, authentication, insufficient-balance, leverage, margin-mode, withdrawal, or order-rejection errors.
- Reconcile an uncertain order outcome by client order ID or order query before retrying placement, where the chosen API path supports that identifier.

## Symbols And Filters

- Native symbols use compact names such as `ETHUSDT`.
- Fetch Spot/Futures symbols before production trading and validate status, precision, minimum quantity/notional, and any price restrictions.
- Do not substitute OKX-style `ETH-USDT`, MEXC-style `ETH_USDT`, or Toobit futures `ETH-SWAP-USDT` examples.
- `KlineInterval`, `FuturesKlineInterval`, and `FuturesSocketKlineInterval` are distinct types.

## Spot Trading

- Bitunix.Net exposes no test-order endpoint; valid `PlaceOrderAsync` calls can create live orders.
- Direct Spot placement accepts symbol, side, order type, quantity, and optional price. Do not invent time-in-force, quote-quantity, or client-order-ID arguments.
- Cancellation is also a live mutation. Confirm the symbol and order ID.
- Batch cancellation may affect multiple orders and should be used only when explicitly requested.
- Check balances and current open orders before examples that place or replace live orders.

## Futures Trading

- Futures orders create leveraged exposure. Make live impact explicit.
- `SetLeverageAsync`, `SetMarginModeAsync`, `SetPositionModeAsync`, `AdjustMarginAsync`, order placement/edit/cancel, TP/SL, and close-position calls mutate the live account.
- Futures balance and leverage calls require a margin asset such as `USDT`.
- In hedge mode, use `TradeSide.Open` or `TradeSide.Close` deliberately. In one-way mode, avoid implying hedge semantics.
- Use `reduceOnly` deliberately when reducing exposure; do not assume it applies to every close workflow.
- Read current position mode, leverage/margin settings, open orders, and positions before changing or closing exposure.
- `CloseAllPositionsAsync` and `CancelAllOrdersAsync` have broad impact. Generate them only when explicitly requested and preserve any requested symbol scope.
- TP/SL methods have many optional parameters. Verify exact signatures and trigger/price types in the source interface.

## Transfers And Withdrawals

- Generate withdrawal or transfer code only when explicitly requested.
- Withdrawal destinations must satisfy Bitunix quick-withdrawal restrictions.
- Validate asset, network, address, quantity, and memo/tag before sending a withdrawal.
- Make Spot/Futures transfer direction, asset, and quantity explicit through `TransferType`.
- Prefer read-only balance, deposit-address, deposit-history, and withdrawal-history examples unless live movement is requested.
- Cancellation of a pending withdrawal is also an account mutation.

## Sockets

- Keep callbacks fast; queue expensive work to a channel or background service.
- Check subscription `Success` before assuming a Futures stream is live.
- Store and unsubscribe successful Futures `UpdateSubscription` values on shutdown.
- Spot socket order calls are queries. Do not call `UnsubscribeAsync` on `QueryResult` data.
- Private update callbacks can contain balances, orders, positions, and TP/SL details; avoid logging sensitive data.
- Reconnection is managed by the library, but application code must still handle duplicate/state-refresh concerns appropriately.

## Shared API V2

- Use SharedApis when portability is required and native APIs for Bitunix-specific features.
- Spot REST has no shared ticker capability. Do not cast it to `IGetTickerRest`; use native `GetLastPriceAsync`.
- Spot socket SharedApi has order query capabilities only, not subscriptions.
- Use a strict capability only from a surface that implements it.
- Do not mix Bitunix-native models or enums with SharedApis request/model types.

## Client Lifetime And Multi-User Code

- Reuse clients or use dependency injection; do not create clients per request.
- Use `IBitunixUserClientProvider` for multiple credential sets rather than scattering ad-hoc credentialed clients.
- Avoid cross-account credential leakage in logs, caches, callbacks, and DI scopes.
- Use the local Futures order-book implementation or factory instead of manually merging depth messages unless the user specifically needs custom order-book logic.

## Environments

- `BitunixEnvironment.Live` is the only built-in environment.
- Do not claim a built-in testnet.
- Custom environments require four explicit addresses. Treat custom endpoints as security-sensitive configuration and validate their origin.
