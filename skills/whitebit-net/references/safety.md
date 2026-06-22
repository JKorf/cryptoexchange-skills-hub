# WhiteBit.Net Safety

Use this file before generating credentials, trading, collateral exposure, leverage, hedge mode, withdrawal, transfer, convert, code, subaccount, kill-switch, or private stream code.

## Credentials And Results

- Install `WhiteBit.Net`.
- Private endpoints use `WhiteBitCredentials(key, secret)`; do not add a passphrase.
- Never hardcode real credentials, addresses, codes, subaccount ids, or order ids.
- Check `HttpResult`, `QueryResult`, or `WebSocketResult` success before using data.
- Retry only transient errors.

## Symbols And Validation

- Native spot symbols use underscores, such as `ETH_USDT`.
- Perpetual collateral symbols use names such as `ETH_PERP`.
- Do not generate `ETHUSDT`, `ETH-USDT`, or `ETH-SWAP-USDT` for native calls.
- Validate trading status, minimum quantity/value, product type, fees, and collateral configuration.

## Spot Trading

- `PlaceSpotOrderAsync` places a live order.
- Include cancellation cleanup for limit-order examples.
- Market buys can use quote quantity where supported.
- Stop orders require trigger prices.
- Cancel-all and kill-switch methods can affect many orders.

## Collateral Trading

- Collateral/perpetual and margin orders create leveraged exposure.
- Leverage and hedge-mode changes affect the whole account configuration.
- Close exposure using an opposite-side order with `reduceOnly: true`.
- Make `PositionSide` explicit in hedge mode.
- OCO, conditional, OTO, TP/SL, and liquidation-sensitive workflows are live actions.

## Funds And Administration

- Withdrawals, transfers, fiat deposits, address creation, code creation/application, conversion confirmation, and subaccount changes move funds or alter administration.
- Generate these only when explicitly requested.
- Validate asset, network, address, memo, unique id, fee behavior, beneficiary data, amount, and account direction.
- Prefer history, balance, settings, estimate, and status examples.

## Websockets And Environment

- Private streams and socket balance/order queries require credentials.
- Keep handlers fast and unsubscribe on shutdown.
- Socket request methods return `QueryResult<T>`; subscriptions return `WebSocketResult<UpdateSubscription>`.
- Current source exposes live and custom environments, not a built-in testnet.

## SharedApis

- Use SharedApis for portable spot/futures workflows.
- Use native APIs for convert, codes, subaccounts, kill switch, hedge mode, OCO/OTO, and detailed WhiteBit models.
- Do not mix native and SharedApis models.
