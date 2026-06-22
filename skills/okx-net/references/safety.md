# OKX.Net Safety

Use this file before generating OKX credential, account, transfer, withdrawal, subaccount, copy trading, derivatives, leverage, position mode, account mode, trading, tracker, private websocket, or socket order request code.

## Credentials

- Install `JK.OKX.Net`, not a guessed package id.
- OKX credentials require key, secret, and passphrase: `new OKXCredentials(key, secret, pass)`.
- Prefer environment variables, secret stores, or dependency-injected options.
- Never hardcode real API keys, secrets, passphrases, wallet addresses, memos, withdrawal ids, subaccount names, or client order ids.
- Use `OKXEnvironment.Demo` only when the user explicitly wants demo/sandbox behavior.

## Result Handling

- REST methods return `HttpResult<T>` or `HttpResult`.
- Websocket subscriptions return `WebSocketResult<UpdateSubscription>`.
- Websocket order request methods return `QueryResult<T>` or `QueryResult<CallResult<T>[]>`.
- Shared non-I/O helpers can return `ExchangeCallResult<T>`.
- Always check `Success` before using `Data`.
- Use `Error.IsTransient` for retry decisions. Do not retry validation, permission, passphrase, insufficient balance, leverage, margin, withdrawal, account mode, or order rejection errors blindly.

## Symbols

- Spot symbols use names such as `ETH-USDT`.
- Perpetual swap symbols use names such as `ETH-USDT-SWAP`.
- Delivery futures symbols use names such as `ETH-USDT-260327`.
- Do not generate Binance-style `ETHUSDT`, MEXC-style `ETH_USDT`, or KuCoin-style futures symbols for native OKX methods.
- Fetch instrument metadata with `UnifiedApi.ExchangeData.GetSymbolsAsync(...)` before trading.
- Websocket order request methods use `OKXInstrument.SymbolCode` from instrument metadata, not the display symbol string.
- Pay attention to `InstrumentType`, `instrumentFamily`, `underlying`, `TradeMode`, `MarginMode`, and `PositionSide` parameters.

## Spot And Margin Trading

- Prefer `CheckOrderAsync` for examples that demonstrate order syntax or risk checks.
- Use `PlaceOrderAsync` only when the user explicitly wants a live order.
- Include cancellation cleanup when demonstrating live limit order placement.
- Use `TradeMode.Cash` for spot cash examples.
- Use margin trade modes only when the user asks for margin or leverage on spot/margin.
- Batch methods can return nested per-item failures even when the outer result succeeds.

## Derivatives Trading

- Swap, futures, and options orders can create leveraged exposure. Say so in generated examples when relevant.
- Use `InstrumentType.Swap`, `InstrumentType.Futures`, or `InstrumentType.Option` for instrument queries and private stream filters.
- `SetLeverageAsync`, `SetPositionModeAsync`, `SetAccountModeAsync`, `SetMarginAmountAsync`, `ClosePositionAsync`, and `PlaceOrderAsync` change live account state.
- Read positions with `UnifiedApi.Account.GetPositionsAsync(...)` before close-position examples.
- `PlaceAlgoOrderAsync`, TP/SL-style algo orders, and websocket order requests are live trading operations.
- Be explicit about `TradeMode.Cross`, `TradeMode.Isolated`, `MarginMode.Cross`, `MarginMode.Isolated`, and `PositionSide`.

## Transfers, Withdrawals, Subaccounts, And Copy Trading

- Withdrawal code should be generated only when explicitly requested.
- Validate asset, network, destination address, fee, amount, and destination type before sending withdrawals.
- Transfers move funds between account types or subaccounts; make source, destination, asset, and amount explicit.
- Subaccount creation, API key creation/deletion/reset, and transfer-out permissions are administrative actions. Generate them only when requested.
- Copy-trading stop/close/amend methods affect live lead/copy positions. Generate them only when requested.
- Prefer read-only deposit/withdrawal status, balances, transfer history, subaccount list, and copy-trading configuration examples unless the user asks for live movement.

## Websockets

- Keep handlers fast. Offload heavier work to a queue, channel, or background service.
- Check subscription `Success` before assuming a stream is live.
- Always unsubscribe on shutdown with `UnsubscribeAsync` or `UnsubscribeAllAsync`.
- Private streams require credentials with the correct permissions.
- Websocket order request methods return `QueryResult<T>`, take numeric symbol codes, and are live trading requests.
- For long-running user data services, consider `IOKXTrackerFactory` instead of ad hoc subscription management.

## SharedApis

- Use SharedApis when the user needs exchange-agnostic code.
- Use native OKX APIs for account mode, subaccounts, copy trading, detailed options data, websocket order requests, and OKX-specific algo order controls.
- Do not mix native OKX request/model types with SharedApis request/model types.
- For Europe environment futures formatting, account for `SharedApiEuropeUseXPerps` and `OKXExchange.FormatSymbolEurope(...)` behavior.
