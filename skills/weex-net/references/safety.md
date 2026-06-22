# Weex.Net Safety

Use this file before generating credentials, account, trading, futures, leverage, margin, fund-history, TP/SL, conditional order, or private stream code.

## Credentials And Results

- Install `Weex.Net`.
- Private endpoints require key, secret, and passphrase through `WeexCredentials`.
- Never hardcode real credentials, wallet addresses, order ids, or user identifiers.
- Check `HttpResult.Success` or `WebSocketResult.Success` before using `Data`.
- Retry only transient failures.

## Symbols And Precision

- Spot and futures commonly use compact symbols such as `ETHUSDT`.
- Fetch exchange info or trading symbols before using user-provided symbols.
- Align spot price and quantity to `TickSize`, `StepSize`, and min/max quantity rules.
- Do not generate OKX-style `ETH-USDT` or Toobit futures-style `ETH-SWAP-USDT`.

## Spot Trading

- `PlaceOrderAsync` places a live order; use only when requested.
- Include cancellation cleanup for live limit-order examples.
- `CancelAllSymbolOrdersAsync` affects multiple orders.
- Check balances and exchange filters before order placement.

## Futures Trading

- Futures orders create leveraged exposure.
- Regular futures orders use `OrderType`; conditional orders use `FuturesOrderType`.
- `SetLeverageAsync`, `SetMarginModeAsync`, isolated-margin methods, TP/SL methods, and `ClosePositionsAsync` change live account state.
- Read current positions before closing exposure.
- Make `PositionSide`, `MarginType`, quantity, price, and trigger working type explicit.

## Fund History

- Current native APIs expose balances, bills, funding bills, and transfer history, not methods that initiate transfers, deposits, or withdrawals.
- SharedApis exposes deposit and withdrawal history from account bills; deposit address retrieval is unavailable.
- Do not invent fund-movement methods, wallet addresses, networks, or destination parameters.

## Websockets

- Private streams require credentials with appropriate permissions.
- Keep handlers fast and avoid logging sensitive account details.
- Check subscription success and unsubscribe on shutdown.

## Environment And SharedApis

- Current source exposes `WeexEnvironment.Live` and custom environments, not a built-in testnet.
- Use SharedApis for portable workflows and native APIs for Weex-specific account, conditional-order, TP/SL, and configuration features.
- Do not mix native and SharedApis models.
