# Mexc.Net Safety

Use this file before generating Mexc credential, account, transfer, withdrawal, subaccount, futures, leverage, position mode, trading, tracker, private websocket, or listen-key code.

## Credentials

- Install `JK.Mexc.Net`, not a guessed package id.
- Use `new MexcCredentials(key, secret)` for standard HMAC credentials.
- `MexcCredentials` also supports `HMACCredential`, `RSAXmlCredential`, and `RSAPemCredential`; use RSA only when the target project or user asks for it.
- Prefer environment variables, secret stores, or dependency-injected options.
- Never hardcode real API keys, secrets, private keys, wallet addresses, memos, withdrawal ids, listen keys, or client order ids.

## Result Handling

- REST methods return `HttpResult<T>` or `HttpResult`.
- Websocket subscriptions return `WebSocketResult<UpdateSubscription>`.
- Shared non-I/O helpers can return `ExchangeCallResult<T>`.
- Always check `Success` before using `Data`.
- Use `Error.IsTransient` for retry decisions. Do not retry validation, unknown symbol, permission, insufficient balance, leverage, margin, withdrawal, or order rejection errors blindly.

## Symbols

- Spot symbols use names such as `BTCUSDT` and `ETHUSDT`.
- Futures symbols use names such as `BTC_USDT` and `ETH_USDT`.
- Do not generate KuCoin-style `BTC-USDT` or Binance futures names for native Mexc futures endpoints.
- Fetch spot metadata with `SpotApi.ExchangeData.GetExchangeInfoAsync(...)` before trading.
- Fetch futures contract metadata with `FuturesApi.ExchangeData.GetSymbolAsync(...)` or `GetSymbolsAsync()` before trading.

## Spot Trading

- Prefer `PlaceTestOrderAsync` for spot order examples that demonstrate order syntax.
- Use `PlaceOrderAsync` only when the user explicitly wants a live order.
- Include cancellation cleanup when demonstrating live limit order placement.
- Market buys can use `quoteQuantity`; be explicit whether sizing is base quantity or quote value.
- Batch order methods can return nested per-item failures even when the outer result succeeds.

## Futures Trading

- Futures orders create leveraged exposure. Say so in generated examples when relevant.
- `FuturesApi.Trading.PlaceOrderAsync` places live futures orders.
- `FuturesOrderSide` encodes both direction and intent: use `OpenLong`/`OpenShort` to open and `CloseLong`/`CloseShort` to close.
- Read positions with `FuturesApi.Trading.GetPositionsAsync(symbol)` before close-position examples.
- `FuturesApi.Account.SetLeverageAsync`, `SetPositionModeAsync`, `ChangeMarginAsync`, and `ToggleAutoAddMarginAsync` change live account state.
- `CloseAllPositionsAsync` and `ReversePositionAsync` are broad live actions; generate only when explicitly requested.
- Plan orders, TP/SL orders, and trailing orders are live conditional orders. Use clear trigger, execution, price, and side language.

## Transfers, Withdrawals, And Subaccounts

- Withdrawal code should be generated only when explicitly requested.
- Validate asset, network, destination address, memo/tag, amount, and client order id before sending withdrawals.
- Transfers and universal transfers move funds between account types or subaccounts; make source, destination, asset, amount, and subaccount identifiers explicit.
- Subaccount creation and API key creation/deletion are administrative actions. Generate them only when requested.
- Prefer read-only deposit/withdrawal status, balances, transfer history, and subaccount list examples unless the user asks for live movement.

## Websockets And Listen Keys

- Keep handlers fast. Offload heavier work to a queue, channel, or background service.
- Check subscription `Success` before assuming a stream is live.
- Always unsubscribe on shutdown with `UnsubscribeAsync` or `UnsubscribeAllAsync`.
- Spot private streams can auto-manage a listen key when credentials are configured.
- If manually using `StartUserStreamAsync()`, keep the listen key alive, handle `ListenkeyRenewed` when applicable, and stop it with `StopUserStreamAsync(...)`.
- Futures private streams use authenticated socket subscriptions directly.
- For long-running user data services, consider `IMexcTrackerFactory` instead of ad hoc subscription management.

## SharedApis

- Use SharedApis when the user needs exchange-agnostic code.
- Use native Mexc APIs for subaccounts, spot listen-key handling, detailed futures leverage/position mode, plan orders, TP/SL, trailing orders, close-all, reverse-position, transfer, and withdrawal details.
- Do not mix native Mexc request/model types with SharedApis request/model types.
