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

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.SpotApi.SharedApi`
- `restClient.FuturesApi.SharedApi`
- `socketClient.SpotApi.SharedApi`
- `socketClient.FuturesApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `ICoinWSharedApiClient`. It exposes:

- `SpotRest` as `ICoinWRestClientSpotSharedApi`
- `FuturesRest` as `ICoinWRestClientFuturesSharedApi`
- `SpotSocket` as `ICoinWSocketClientSpotSharedApi`
- `FuturesSocket` as `ICoinWSocketClientFuturesSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `ICoinWSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.

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
