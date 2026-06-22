# Polymarket.Net Safety

Use this file before generating Polymarket credential, allowance, trading, user stream, order heartbeat, local order book, or per-user client code.

## Credentials

- Install `Polymarket.Net`, not a guessed package id.
- Private endpoints require L1 wallet credentials: `PolymarketL1Credential`.
- Trading and private user websocket endpoints generally require L2 credentials: `HMACPassCredential`.
- Prefer environment variables, secret stores, or dependency-injected options.
- Never hardcode real private keys, API keys, API secrets, passphrases, wallet addresses, order ids, token ids tied to private strategy, or user identifiers.
- If deriving L2 credentials with `GetOrCreateApiCredentialsAsync`, call `UpdateL2Credentials(...)` before trading or subscribing to user streams.

## Result Handling

- REST methods return `HttpResult<T>` or `HttpResult`.
- Websocket subscriptions return `WebSocketResult<UpdateSubscription>`.
- Batch order placement returns `HttpResult<CallResult<PolymarketOrderResult>[]>`.
- Always check `Success` before using `Data`.
- For order placement, also check `PolymarketOrderResult.Success` and `PolymarketOrderResult.Error`.
- Use `Error.IsTransient` for retry decisions. Do not retry invalid token ids, auth errors, geo restrictions, insufficient allowance, closed markets, order validation failures, or rejected order payloads blindly.

## Tokens, Markets, And Prices

- CLOB methods use token ids, not symbols.
- Market/condition ids are not token ids.
- Do not generate Binance-style `BTCUSDT`, OKX-style `ETH-USDT`, or futures symbols for Polymarket CLOB methods.
- Use Gamma data or CLOB market lists to discover token ids before trading.
- Prices are probabilities between `0` and `1`; `0.42` means 42 cents.
- Validate tick size, fee rate, negative-risk status, market status, and allowance before production order placement.

## Trading

- Prefer read-only examples unless the user explicitly asks for live trading.
- `PlaceOrderAsync` and `PlaceMultipleOrdersAsync` place live CLOB orders.
- `CancelAllOrdersAsync` and `CancelOrdersOnMarketAsync` can affect many orders; generate them only when explicitly requested.
- Include cancellation cleanup when demonstrating live limit order placement.
- `UpdateBalanceAllowanceAsync` changes allowance/approval state; generate it only when requested.
- `PostOrderHeartbeatAsync` is for live order lifecycle behavior and should be sent repeatedly only in services intentionally using heartbeat-protected orders.
- Be explicit about `OrderSide`, `OrderType`, `TimeInForce`, `QuantityType`, `quantity`, and `price`.

## Websockets

- Keep handlers fast. Offload heavier work to a queue, channel, or background service.
- Check subscription `Success` before assuming a stream is live.
- Always unsubscribe on shutdown with `UnsubscribeAsync` or `UnsubscribeAllAsync`.
- Public token, platform, and sports streams do not require credentials.
- Private user streams require credentials with the correct L1/L2 setup.

## Local Order Books

- Local order books use CLOB token ids.
- Start/stop order books explicitly and handle failed `StartAsync` results.
- Do not create one order book per request in web applications; use long-lived managed instances.

## User Client Provider

- Use `IPolymarketUserClientProvider` when an application handles multiple user credential sets.
- Avoid logging user identifiers alongside private keys or API secrets.
- Make credential ownership and environment explicit when initializing user clients.

## SharedApis

- Polymarket.Net does not expose exchange-agnostic trading SharedApis surfaces like spot/futures exchanges.
- Use native Polymarket APIs for CLOB tokens, Gamma metadata, user streams, local order books, and trading.
