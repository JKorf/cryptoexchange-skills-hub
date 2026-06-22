# CoinW.Net API Surfaces

Use this file when choosing native CoinW client roots, endpoint groups, shared interfaces, or symbol formats.

## Package And Client Types

- NuGet package: `CoinW.Net`
- REST client: `CoinWRestClient`
- Socket client: `CoinWSocketClient`
- REST interface: `ICoinWRestClient`
- Socket interface: `ICoinWSocketClient`
- Credentials: `CoinWCredentials`
- DI extension: `services.AddCoinW(...)`

## Client Roots

REST roots:

- `client.SpotApi`
- `client.FuturesApi`

Socket roots:

- `socket.SpotApi`
- `socket.FuturesApi`

Do not use Binance-style roots such as `UsdFuturesApi`, `CoinFuturesApi`, `V5Api`, `ExchangeApi`, or `UnifiedApi`.

## Spot REST

`client.SpotApi.ExchangeData`:

- `GetTickersAsync()`
- `GetAssetsAsync()`
- `GetSymbolsAsync()`
- `GetOrderBookAsync(symbol, limit)`
- `GetRecentTradesAsync(symbol, startTime, endTime)`
- `GetKlinesAsync(symbol, interval, startTime, endTime)`

`client.SpotApi.Account`:

- `GetBalancesAsync()`
- `GetBalancesDetailsAsync()`
- `GetDepositWithdrawalHistoryAsync(asset, id)`
- `GetDepositWithdrawalHistoryAsync(assets, id)`
- `GetDepositAddressesAsync(asset, network)`
- `WithdrawAsync(asset, quantity, address, network, memo, type, internalWithdrawType)`
- `CancelWithdrawalAsync(withdrawalId)`
- `TransferAsync(fromAccount, toAccount, asset, quantity)`

`client.SpotApi.Trading`:

- `PlaceOrderAsync(symbol, side, type, quantity, quoteQuantity, price, clientOrderId)`
- `CancelOrderAsync(orderId)`
- `CancelAllOrdersAsync(symbol)`
- `GetOpenOrdersAsync(symbol, startTime, endTime)`
- `GetOrderAsync(orderId)`
- `GetOrderTransactionHistoryAsync(symbol, startTime, endTime)`
- `GetUserTradesAsync(symbol, fromId, toId, startTime, endTime, limit)`

## Futures REST

`client.FuturesApi.ExchangeData`:

- `GetSymbolsAsync(symbol)`
- `GetTickerAsync(symbol)`
- `GetTickersAsync()`
- `GetKlinesAsync(symbol, interval, startTime, endTime, limit)`
- `GetLastFundingRateAsync(symbol)`
- `GetOrderBookAsync(symbol)`
- `GetRecentTradesAsync(symbol)`
- `GetMarginRequirementsAsync()`
- `GetTradeHistoryAsync(symbol, page, pageSize)`

`client.FuturesApi.Account`:

- `GetLeverageAsync(positionId, orderId)`
- `GetMarginRateAsync(positionId)`
- `GetMaxTradeSizeAsync(symbol, leverage, marginType, orderPrice)`
- `GetMaxTransferableAsync()`
- `GetBalancesAsync()`
- `GetFeesAsync()`
- `GetMarginModeAsync()`
- `SetMarginModeAsync(marginType, positionCombineType)`
- `ToggleMegaCouponAsync(enabled)`
- `GetMaxPositionSizeAsync(symbol)`

`client.FuturesApi.Trading`:

- `PlaceOrderAsync(symbol, side, orderType, quantity, leverage, price, quantityUnit, marginType, stopLossPrice, takeProfitPrice, triggerPrice, triggerOrderType, goldenId, clientOrderId, useMegaCoupon)`
- `PlaceMultipleOrdersAsync(requests)`
- `ClosePositionAsync(positionId, orderType, quantityToClose, factorToClose, price)`
- `ClosePositionsByClientOrderIdAsync(clientOrderIds)`
- `CloseAllPositionsAsync(symbol)`
- `ReversePositionAsync(positionId)`
- `AdjustMarginAsync(positionId, addMargin, reduceMargin)`
- `SetTpSlAsync(orderOrPositionId, symbol, takeProfitPrice, takeProfitOrderPrice, takeProfitRate, stopLossPrice, stopLossOrderPrice, stopLossRate)`
- `SetTrailingTpSlAsync(positionId, callbackRate, triggerPrice, quantity, quantityType)`
- `EditOrderAsync(orderId, symbol, side, orderType, quantity, leverage, price, quantityUnit, marginType, stopLossPrice, takeProfitPrice, triggerPrice, triggerOrderType, goldenId, clientOrderId, useMegaCoupon)`
- `CancelOrderAsync(orderId)`
- `CancelOrdersAsync(orderIds)`
- `GetOpenOrdersAsync(...)`
- `GetOpenOrderCountAsync()`
- `GetTpSlAsync(orderId, positionId, planOrderId, symbol)`
- `GetTrailingTpSlAsync()`
- `GetOrderHistory7DaysAsync(symbol, orderType, page, pageSize)`
- `GetOrderHistory3MonthsAsync(symbol, orderType, page, pageSize)`
- `GetPositionsAsync(symbol)`
- `GetPositionsAsync()`
- `GetPositionHistoryAsync(symbol, marginType)`
- `GetTransactionHistory3DaysAsync(symbol, orderType, marginType, page, pageSize)`
- `GetTransactionHistory3MonthsAsync(symbol, orderType, marginType, page, pageSize)`

Futures order placement opens positions. To close existing positions, use `ClosePositionAsync`, `CloseAllPositionsAsync`, or the close-by-client-order-id helper; do not place an opposite order unless the user explicitly wants that strategy.

## Spot Websocket

`socket.SpotApi`:

- `SubscribeToTickerUpdatesAsync(symbol, handler)`
- `SubscribeToAllTickerUpdatesAsync(handler)`
- `SubscribeToOrderBookUpdatesAsync(symbol, handler)`
- `SubscribeToPartialOrderBookUpdatesAsync(symbol, handler)`
- `SubscribeToKlineUpdatesAsync(symbol, interval, handler)`
- `SubscribeToTradeUpdatesAsync(symbol, handler)`
- `SubscribeToBalanceUpdatesAsync(handler)`
- `SubscribeToOrderUpdatesAsync(handler)`

Private spot streams require credentials on the socket client.

## Futures Websocket

`socket.FuturesApi`:

- `SubscribeToTickerUpdatesAsync(symbol, handler)`
- `SubscribeToOrderBookUpdatesAsync(symbol, handler)`
- `SubscribeToTradeUpdatesAsync(symbol, handler)`
- `SubscribeToKlineUpdatesAsync(symbol, interval, handler)`
- `SubscribeToIndexPriceUpdatesAsync(symbol, handler)`
- `SubscribeToMarkPriceUpdatesAsync(symbol, handler)`
- `SubscribeToFundingRateUpdatesAsync(symbol, handler)`
- `SubscribeToOrderUpdatesAsync(handler)`
- `SubscribeToPositionUpdatesAsync(handler)`
- `SubscribeToPositionDetailUpdatesAsync(handler)`
- `SubscribeToBalanceUpdatesAsync(handler)`
- `SubscribeToMarginConfigUpdatesAsync(handler)`

Private futures streams require credentials on the socket client.

## SharedApis Interfaces

Spot REST shared client: `client.SpotApi.SharedClient`

Implemented spot REST interfaces:

- `IAssetsRestClient`
- `IBalanceRestClient`
- `IDepositRestClient`
- `IKlineRestClient`
- `IOrderBookRestClient`
- `IRecentTradeRestClient`
- `IWithdrawalRestClient`
- `IWithdrawRestClient`
- `ISpotTickerRestClient`
- `ISpotSymbolRestClient`
- `ISpotOrderRestClient`
- `ITransferRestClient`

Spot socket shared client: `socket.SpotApi.SharedClient`

Implemented spot socket interfaces:

- `IBalanceSocketClient`
- `IKlineSocketClient`
- `IOrderBookSocketClient`
- `ITickerSocketClient`
- `ITickersSocketClient`
- `ITradeSocketClient`
- `ISpotOrderSocketClient`

Futures REST shared client: `client.FuturesApi.SharedClient`

Implemented futures REST interfaces:

- `IFeeRestClient`
- `IKlineRestClient`
- `IOrderBookRestClient`
- `IRecentTradeRestClient`
- `IFuturesSymbolRestClient`
- `IFuturesTickerRestClient`
- `IFuturesOrderRestClient`
- `IFuturesTpSlRestClient`
- `IBalanceRestClient`

Futures socket shared client: `socket.FuturesApi.SharedClient`

Implemented futures socket interfaces:

- `IBalanceSocketClient`
- `IKlineSocketClient`
- `IOrderBookSocketClient`
- `ITickerSocketClient`
- `ITradeSocketClient`
- `IFuturesOrderSocketClient`
- `IPositionSocketClient`

Call `SharedClient.Discover()` before relying on optional shared features.

## Symbols

- Native spot examples use `BTC_USDT` and `ETH_USDT`.
- Native futures examples use `BTC` and `ETH`.
- SharedApis uses `new SharedSymbol(TradingMode.Spot, "BTC", "USDT")` or futures trading modes such as `TradingMode.PerpetualLinear`.
- `CoinWExchange.FormatSymbol(baseAsset, quoteAsset, tradingMode)` can format CoinW-native symbols when the code is not using SharedApis.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Websocket subscriptions: `WebSocketResult<UpdateSubscription>`
- Shared non-I/O helpers: `ExchangeCallResult<T>`
- Batch futures order placement: `HttpResult<CallResult<CoinWBatchResult>[]>`

Always check the outer result first. For batch methods, inspect each inner item after the outer result succeeds.
