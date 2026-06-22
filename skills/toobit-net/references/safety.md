# Toobit.Net Safety

Use this file before generating Toobit credential, account, transfer, withdrawal, futures, leverage, margin type, trading, tracker, private websocket, or user stream code.

## Credentials

- Install `Toobit.Net`, not a guessed package id.
- Private endpoints require `new ToobitCredentials(key, secret)`.
- Prefer environment variables, secret stores, or dependency-injected options.
- Never hardcode real API keys, secrets, wallet addresses, withdrawal ids, subaccount identifiers, listen keys, or client order ids.

## Result Handling

- REST methods return `HttpResult<T>` or `HttpResult`.
- Websocket subscriptions return `WebSocketResult<UpdateSubscription>`.
- Batch order methods return nested per-item `CallResult<T>` values.
- Shared helper methods can return `ExchangeCallResult<T>`.
- Always check `Success` before using `Data`.
- Use `Error.IsTransient` for retry decisions. Do not retry validation, bad credentials, insufficient balance, leverage, margin, withdrawal, or order rejection errors blindly.

## Symbols

- Spot symbols use names such as `BTCUSDT`.
- USDT futures symbols use names such as `ETH-SWAP-USDT`.
- Futures index-price methods can use index symbols such as `ETHUSDT`.
- Do not generate OKX-style `ETH-USDT`, MEXC-style `ETH_USDT`, or Binance futures root names for Toobit futures methods.
- Fetch exchange info before production trading and validate filters, quantity, price, and status.

## Spot Trading

- Prefer `PlaceTestOrderAsync` for examples that demonstrate order syntax or risk checks.
- Use `PlaceOrderAsync` only when the user explicitly wants a live order.
- Include cancellation cleanup when demonstrating live limit order placement.
- Market buy quantity is in quote asset for spot market buy orders.
- Batch methods can return nested per-item failures even when the outer result succeeds.
- `CancelAllOrdersAsync` can affect many orders; generate it only when explicitly requested.

## Futures Trading

- USDT futures orders can create leveraged exposure. Say so in generated examples when relevant.
- Use `FuturesOrderSide.BuyOpen` and `SellOpen` to open exposure; use `SellClose` and `BuyClose` to close exposure.
- `SetLeverageAsync`, `SetMarginTypeAsync`, `SetPositionMarginAsync`, `SetTradingStopAsync`, and futures `PlaceOrderAsync` change live account state.
- Read positions with `UsdtFuturesApi.Trading.GetPositionsAsync(...)` before close-position examples.
- Be explicit about `FuturesNewOrderType`, `FuturesOrderType`, `PriceType`, `PositionSide`, `MarginType`, `TriggerType`, `quantity`, and `price`.

## Transfers, Withdrawals, And Subaccounts

- Withdrawal code should be generated only when explicitly requested.
- Validate asset, network, address, tag/memo, amount, VASP data, and client order id before sending withdrawals.
- Transfers move funds between accounts or subaccounts; make source, destination, asset, and amount explicit.
- Prefer read-only deposit/withdrawal status, balances, transaction history, and subaccount list examples unless the user asks for live movement.

## Websockets And User Streams

- Keep handlers fast. Offload heavier work to a queue, channel, or background service.
- Check subscription `Success` before assuming a stream is live.
- Always unsubscribe on shutdown with `UnsubscribeAsync` or `UnsubscribeAllAsync`.
- User streams require credentials or a listen key created by `StartUserStreamAsync`.
- If manually using listen keys, call `KeepAliveUserStreamAsync` for long-running streams and `StopUserStreamAsync` on shutdown.
- Private stream callbacks can contain account, balance, order, position, and trade data; avoid logging sensitive details.

## SharedApis

- Use SharedApis when the user needs exchange-agnostic code.
- Use native Toobit APIs for listen-key lifecycle, Toobit-specific futures contract symbols, leverage, margin type, trading stops, withdrawals, and detailed account models.
- Do not mix native Toobit request/model types with SharedApis request/model types.
