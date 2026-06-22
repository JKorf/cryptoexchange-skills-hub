# DeepCoin.Net API Surfaces

Use this file when choosing native DeepCoin client roots, endpoint groups, shared interfaces, result types, or symbol formats.

## Package And Client Types

- NuGet package: `DeepCoin.Net`
- REST client: `DeepCoinRestClient`
- Socket client: `DeepCoinSocketClient`
- REST interface: `IDeepCoinRestClient`
- Socket interface: `IDeepCoinSocketClient`
- Credentials: `DeepCoinCredentials`
- DI extension: `services.AddDeepCoin(...)`

## Client Roots

REST root:

- `client.ExchangeApi`

Socket root:

- `socket.ExchangeApi`

Do not use exchange roots such as `SpotApi`, `FuturesApi`, `UsdFuturesApi`, `CoinFuturesApi`, `UnifiedApi`, or `V5Api`.

## REST Endpoint Groups

`client.ExchangeApi.ExchangeData`:

- `GetTickersAsync(symbolType, underlying)`
- `GetSymbolsAsync(type, underlying, symbol)`
- `GetKlinesAsync(symbol, interval, endTime, limit)`
- `GetOrderBookAsync(symbol, depth)`
- `GetFundingRateAsync(type, symbol)`

`client.ExchangeApi.Account`:

- `GetBalancesAsync(accountType, asset)`
- `GetBillsAsync(accountType, asset, billType, startTime, endTime, limit)`
- `SetLeverageAsync(symbol, leverage, tradeMode, positionType)`
- `GetDepositHistoryAsync(asset, transactionHash, startTime, endTime, page, pageSize)`
- `GetWithdrawHistoryAsync(asset, transactionHash, startTime, endTime, page, pageSize)`
- `StartUserStreamAsync()`
- `KeepAliveUserStreamAsync(listenKey)`

Transfer endpoints are present in comments in the source and marked unusable; do not document or generate them as available methods.

`client.ExchangeApi.Trading`:

- `GetPositionsAsync(symbolType, symbol)`
- `PlaceOrderAsync(symbol, side, orderType, quantity, price, tradeMode, asset, clientOrderId, quantityType, positionSide, positionType, closePosId, reduceOnly, tpTriggerPrice, slTriggerPrice)`
- `EditOrderAsync(orderId, price, quantity)`
- `CancelOrderAsync(symbol, orderId)`
- `CancelOrdersAsync(orderIds)`
- `CancelAllOrdersAsync(symbol, productGroup, marginMode, positionType)`
- `GetUserTradesAsync(symbolType, symbol, orderId, afterId, beforeId, startTime, endTime, limit)`
- `GetOpenOrderAsync(symbol, orderId)`
- `GetClosedOrderAsync(symbol, orderId)`
- `GetClosedOrdersAsync(symbolType, symbol, orderId, orderType, status, afterId, beforeId, limit)`
- `GetOpenOrdersAsync(symbol, page, pageSize, orderId)`
- `SetTpSlAsync(orderId, takeProfitTriggerPrice, stopLossTriggerPrice)`

## Socket Subscriptions

`socket.ExchangeApi` public streams:

- `SubscribeToSymbolUpdatesAsync(symbol, handler)`
- `SubscribeToTradeUpdatesAsync(symbol, handler)`
- `SubscribeToKlineUpdatesAsync(symbol, handler)`
- `SubscribeToOrderBookUpdatesAsync(symbol, handler)`

Native kline subscriptions support one-minute updates.

`socket.ExchangeApi` private streams:

- `SubscribeToUserDataUpdatesAsync(...)`
- `SubscribeToUserDataUpdatesAsync(listenKey, ...)`

Private stream callbacks include order updates, balance updates, position updates, user trade updates, account updates, and trigger order updates. The overload without a listen key obtains and renews a listen key automatically. The overload with a listen key uses `Account.StartUserStreamAsync()` output.

## SharedApis Interfaces

REST shared client: `client.ExchangeApi.SharedClient`

Implemented REST shared interfaces:

- `IBalanceRestClient`
- `IDepositRestClient`
- `IKlineRestClient`
- `IOrderBookRestClient`
- `IWithdrawalRestClient`
- `ISpotTickerRestClient`
- `ISpotSymbolRestClient`
- `ISpotOrderRestClient`
- `ILeverageRestClient`
- `IFuturesTickerRestClient`
- `IFuturesSymbolRestClient`
- `IFuturesOrderRestClient`
- `IBookTickerRestClient`

Socket shared client: `socket.ExchangeApi.SharedClient`

Implemented socket shared interfaces:

- `IKlineSocketClient`
- `ITickerSocketClient`
- `ITradeSocketClient`
- `IBalanceSocketClient`
- `ISpotOrderSocketClient`
- `IFuturesOrderSocketClient`
- `IUserTradeSocketClient`
- `IPositionSocketClient`

Supported shared trading modes include `TradingMode.Spot`, `TradingMode.PerpetualLinear`, and `TradingMode.PerpetualInverse`. Call `SharedClient.Discover()` before relying on optional shared features.

## Symbols

- Native spot symbols use hyphenated pairs: `ETH-USDT`.
- Native swap/futures symbols use `-SWAP`: `ETH-USDT-SWAP`.
- `DeepCoinExchange.FormatSymbol("ETH", "USDT", TradingMode.Spot)` returns `ETH-USDT`.
- `DeepCoinExchange.FormatSymbol("ETH", "USDT", TradingMode.PerpetualLinear)` returns `ETH-USDT-SWAP`.
- Native socket methods accept normal native symbols such as `ETH-USDT` or `ETH-USDT-SWAP`; the exchange helper also has `FormatWebsocketSymbol` for websocket-specific formatting inside shared implementations.
- SharedApis uses `SharedSymbol`, for example `new SharedSymbol(TradingMode.Spot, "ETH", "USDT")`.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Websocket subscriptions: `WebSocketResult<UpdateSubscription>`
- Shared non-I/O helpers: `ExchangeCallResult<T>`

Always check `Success` before using `Data`. Cancellation and batch cancel methods return result objects that still need inspection after outer success.
