# Kraken.Net Safety

Use this file before generating Kraken credential, account, transfer, withdrawal, Earn allocation, futures, leverage, trading, websocket order, tracker, or websocket code.

## Credentials

- Install `KrakenExchange.Net`, not a guessed package id.
- Use separate spot and futures HMAC credential slots with `new KrakenCredentials(spotCredential, futuresCredential)`.
- Use `new HMACCredential(key, secret)` from `CryptoExchange.Net.Authentication`.
- Avoid the obsolete `KrakenCredentials(string apiKey, string secret)` constructor.
- Prefer environment variables, secret stores, or dependency-injected options.
- Never hardcode real API keys, secrets, wallet keys, withdrawal keys, two-factor codes, or client order ids.

## Result Handling

- REST methods return `HttpResult<T>` or `HttpResult`.
- Websocket subscriptions return `WebSocketResult<UpdateSubscription>`.
- Spot websocket request/order methods return `QueryResult<T>` or `QueryResult`.
- Shared non-I/O helpers can return `ExchangeCallResult<T>`.
- Always check `Success` before using `Data`.
- Use `Error.IsTransient` for retry decisions. Do not retry validation, permission, 2FA, insufficient balance, leverage, or order rejection errors blindly.

## Symbols

- REST spot symbols use names such as `ETHUSDT`.
- Spot websocket symbols use names such as `ETH/USDT`.
- Fetch `KrakenSymbol.WebsocketName` from `SpotApi.ExchangeData.GetSymbolsAsync(...)` when converting.
- Futures symbols use names such as `PF_ETHUSD`.
- Do not generate `ETH_USDT`, `ETH-USDT`, or Binance futures symbols for Kraken native methods.

## Spot Trading

- Prefer `validateOnly: true` for spot order examples that demonstrate order placement syntax.
- Remove `validateOnly` only when the user explicitly wants a live order.
- Include cancellation cleanup when demonstrating live order placement.
- Kraken spot order methods have optional 2FA parameters. Do not invent 2FA codes; pass only when supplied.
- Spot websocket v2 order methods can place live orders and return `QueryResult<T>`. Treat them like trading endpoints.
- Batch methods can return nested per-item failures even when the outer result succeeds.

## Futures Trading

- Futures orders create leveraged exposure. Say so in generated examples when relevant.
- `SetLeverageAsync` changes account/symbol leverage state.
- Futures REST order placement has no `validateOnly` parameter in the inspected interface. Do not claim it can dry-run futures orders.
- Use `reduceOnly` when generating close-position examples.
- Read positions with `FuturesApi.Trading.GetOpenPositionsAsync()` before close-position examples.

## Earn, Transfers, And Withdrawals

- `Earn.AllocateEarnFundsAsync` and `Earn.DeallocateEarnFundsAsync` move funds into or out of Earn strategies. Generate them only when explicitly requested.
- Withdrawal code should be generated only when explicitly requested.
- Validate withdrawal asset, method/key, address, amount, and 2FA requirements before sending.
- Wallet `TransferAsync` moves funds between Kraken wallets; make source wallet, destination wallet, asset, and quantity explicit.
- Prefer read-only deposit/withdrawal status and method examples unless the user asks for live movement.

## Websockets And Trackers

- Keep handlers fast. Offload heavier work to a queue, channel, or background service.
- Check subscription `Success` before assuming a stream is live.
- Always unsubscribe on shutdown with `UnsubscribeAsync` or `UnsubscribeAllAsync`.
- Private spot streams and spot websocket order requests require spot credentials and websocket token handling inside the library.
- Private futures streams require futures credentials.
- For long-running user data services, consider `IKrakenTrackerFactory` instead of ad hoc subscription management.

## SharedApis

- Use SharedApis when the user needs exchange-agnostic code.
- Use native Kraken APIs for Earn, wallet transfer, 2FA-specific parameters, Kraken websocket v2 order requests, and detailed futures features.
- Do not mix native Kraken request/model types with SharedApis request/model types.
