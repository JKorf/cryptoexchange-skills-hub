# Mexc.Net API Surfaces

Use this file when choosing native Mexc client roots, endpoint groups, shared interfaces, result types, or symbol formats.

## Package And Client Types

- NuGet package: `JK.Mexc.Net`
- REST client: `MexcRestClient`
- Socket client: `MexcSocketClient`
- REST interface: `IMexcRestClient`
- Socket interface: `IMexcSocketClient`
- Credentials: `MexcCredentials`
- DI extension: `services.AddMexc(...)`
- Tracker factory: `IMexcTrackerFactory`

## Client Roots

REST roots:

- `client.SpotApi`
- `client.FuturesApi`

Socket roots:

- `socket.SpotApi`
- `socket.FuturesApi`

Do not use exchange roots such as `UsdFuturesApi`, `CoinFuturesApi`, `UnifiedApi`, `V5Api`, or `ExchangeApi`.

## Spot REST Endpoint Groups

`client.SpotApi.ExchangeData`:

- `PingAsync()`
- `GetServerTimeAsync()`
- `GetApiSymbolsAsync()`
- `GetExchangeInfoAsync(symbols)`
- `GetOrderBookAsync(symbol, limit)`
- `GetRecentTradesAsync(symbol, limit)`
- `GetAggregatedTradeHistoryAsync(symbol, startTime, endTime, limit)`
- `GetKlinesAsync(symbol, interval, startTime, endTime, limit)`
- `GetAveragePriceAsync(symbol)`
- `GetTickerAsync(symbol)`
- `GetTickersAsync()`
- `GetPricesAsync(symbols)`
- `GetBookPricesAsync(symbol)`
- `GetBookPricesAsync()`
- `GetOfflineSymbolsAsync()`

`client.SpotApi.Account`:

- `GetAccountInfoAsync()`
- `GetKycStatusAsync()`
- `GetUserAssetsAsync()`
- `WithdrawAsync(asset, address, quantity, ...)`
- `CancelWithdrawAsync(withdrawId)`
- `GetDepositHistoryAsync(...)`
- `GetWithdrawHistoryAsync(...)`
- `GenerateDepositAddressAsync(asset, network)`
- `GetDepositAddressesAsync(asset, network)`
- `GetWithdrawAddressesAsync(asset, page, pageSize)`
- `TransferAsync(asset, fromAccountType, toAccountType, quantity)`
- `GetTransferHistoryAsync(...)`
- `GetTransferAsync(transferId)`
- `GetAssetsForDustTransferAsync()`
- `DustTransferAsync(assets)`
- `GetDustLogAsync(...)`
- `TransferInternalAsync(...)`
- `GetInternalTransferHistoryAsync(...)`
- `SetMxDeductionAsync(enabled)`
- `GetMxDeductionStatusAsync()`
- `GetTradeFeeAsync(symbol)`
- `StartUserStreamAsync()`
- `KeepAliveUserStreamAsync(listenKey)`
- `StopUserStreamAsync(listenKey)`
- `GetRebateHistoryAsync(...)`
- `GetRebateDetailsAsync(...)`
- `GetRebateKickbackAsync(...)`
- `GetAffiliateCommissionAsync(...)`

`client.SpotApi.Trading`:

- `PlaceTestOrderAsync(symbol, side, type, quantity, quoteQuantity, price, clientOrderId)`
- `PlaceOrderAsync(symbol, side, type, quantity, quoteQuantity, price, clientOrderId)`
- `PlaceMultipleOrdersAsync(requests)`
- `CancelOrderAsync(symbol, orderId, clientOrderId, newClientOrderId)`
- `CancelAllOrdersAsync(symbol)`
- `CancelAllOrdersAsync(symbols)`
- `GetOrderAsync(symbol, orderId, clientOrderId)`
- `GetOpenOrdersAsync(symbol)`
- `GetOrdersAsync(symbol, startTime, endTime, limit)`
- `GetUserTradesAsync(symbol, orderId, startTime, endTime, limit)`

`client.SpotApi.SubAccount`:

- `GetSubUserAccountsAsync(name, isFreeze, page, limit)`
- `GetSubUserAccountApiDetailsAsync(subAccount)`
- `UniversalTransferAsync(asset, amount, fromAccountType, toAccountType, fromAccount, toAccount)`
- `GetUniversalTransfersAsync(...)`
- `CreateSubAccountAsync(name, note)`
- `CreateSubAccountApiKeyAsync(subAccount, note, permissions, ipAddresses)`
- `DeleteSubAccountApiKeyAsync(subAccount, apiKey)`
- `GetSubAccountBalancesAsync(subAccount, accountType)`

## Futures REST Endpoint Groups

`client.FuturesApi.ExchangeData`:

- `GetServerTimeAsync()`
- `GetSymbolAsync(symbol)`
- `GetSymbolsAsync()`
- `GetTransferableAssetsAsync()`
- `GetOrderBookAsync(symbol, limit)`
- `GetIndexPriceAsync(symbol)`
- `GetMarkPriceAsync(symbol)`
- `GetFundingRateAsync(symbol)`
- `GetFundingRatesAsync()`
- `GetKlinesAsync(symbol, interval, startTime, endTime)`
- `GetIndexPriceKlinesAsync(symbol, interval, startTime, endTime)`
- `GetMarkPriceKlinesAsync(symbol, interval, startTime, endTime)`
- `GetRecentTradesAsync(symbol, limit)`
- `GetTickerAsync(symbol)`
- `GetTickersAsync()`
- `GetRiskFundBalancesAsync()`
- `GetRiskFundBalanceHistoryAsync(symbol, page, pageSize)`
- `GetFundingRateHistoryAsync(symbol, page, pageSize)`

`client.FuturesApi.Account`:

- `GetBalanceAsync(asset)`
- `GetBalancesAsync()`
- `GetTransferHistoryAsync(asset, status, direction, page, pageSize)`
- `GetFundingHistoryAsync(symbol, positionId, page, pageSize)`
- `GetTradingFeesAsync(symbol)`
- `ChangeMarginAsync(positionId, quantity, changeType)`
- `GetLeverageAsync(symbol)`
- `SetLeverageAsync(leverage, positionId, marginType, symbol, positionSide)`
- `GetPositionModeAsync()`
- `SetPositionModeAsync(positionMode)`
- `GetProfitRateAsync(period)`
- `GetDeductionConfigAsync()`
- `GetDiscountTypesAsync()`
- `GetZeroFeeSymbolsAsync(symbol)`
- `ToggleAutoAddMarginAsync(positionId, enabled)`

`client.FuturesApi.Trading`:

- `GetOpenOrdersAsync(page, pageSize)`
- `GetOrderHistoryAsync(...)`
- `GetOrderByClientOrderIdAsync(symbol, clientOrderId)`
- `GetOrderAsync(orderId)`
- `GetOrdersByIdAsync(orderIds)`
- `GetOrderTradesAsync(orderId)`
- `GetUserTradesAsync(symbol, startTime, endTime, page, pageSize)`
- `GetTriggerOrdersAsync(...)`
- `GetTpSlOrdersAsync(...)`
- `GetRiskLimitsAsync(symbol)`
- `GetPositionHistoryAsync(...)`
- `GetPositionsAsync(symbol)`
- `PlaceOrderAsync(symbol, side, type, quantity, price, leverage, marginType, ...)`
- `PlaceMultipleOrdersAsync(requests)`
- `CancelOrdersAsync(orderIds)`
- `ChaseOrderAsync(orderId)`
- `EditOrderAsync(orderId, ...)`
- `CancelOrdersByClientOrderIdsAsync(orders)`
- `CancelAllOrdersAsync(symbol)`
- `ReversePositionAsync(...)`
- `CloseAllPositionsAsync()`
- `GetOpenOrderCountsAsync()`
- `PlacePlanOrderAsync(...)`
- `EditPlanOrderAsync(...)`
- `CancelPlanOrdersAsync(orders)`
- `CancelAllPlannedOrdersAsync(symbol)`
- `PlaceTpSlOrderAsync(...)`
- `CancelTpSlOrdersAsync(orders)`
- `CancelAllTpSlOrdersAsync(...)`
- `EditLimitOrderTpSlAsync(...)`
- `EditTpSlOrderAsync(...)`
- `GetOpenTpSlOrdersAsync(symbol)`
- `PlaceTrailingOrderAsync(...)`
- `CancelTrailingOrderAsync(...)`
- `EditTrailingOrderAsync(...)`
- `GetTrailingOrdersAsync(...)`

Futures order `side` uses `FuturesOrderSide`, for example `OpenLong`, `OpenShort`, `CloseLong`, or `CloseShort`.

## Socket Subscriptions

`socket.SpotApi` public streams:

- `SubscribeToTradeUpdatesAsync(symbols, updateInterval, handler)`
- `SubscribeToKlineUpdatesAsync(symbols, interval, handler)`
- `SubscribeToOrderBookUpdatesAsync(symbols, updateInterval, handler)`
- `SubscribeToPartialOrderBookUpdatesAsync(symbols, depth, handler)`
- `SubscribeToBookTickerUpdatesAsync(symbols, handler)`
- `SubscribeToMiniTickerUpdatesAsync(symbols, timeZone, handler)`
- `SubscribeToAllMiniTickerUpdatesAsync(timeZone, handler)`

`socket.SpotApi` private streams:

- `SubscribeToAccountUpdatesAsync(handler)`
- `SubscribeToAccountUpdatesAsync(listenKey, handler)`
- `SubscribeToOrderUpdatesAsync(handler)`
- `SubscribeToOrderUpdatesAsync(listenKey, handler)`
- `SubscribeToUserTradeUpdatesAsync(handler)`
- `SubscribeToUserTradeUpdatesAsync(listenKey, handler)`

Credential-based overloads automatically obtain and renew a listen key. Listen-key overloads require `SpotApi.Account.StartUserStreamAsync(...)` and cleanup with `StopUserStreamAsync(...)`.

`socket.FuturesApi` streams:

- `SubscribeToTickersUpdatesAsync(handler)`
- `SubscribeToTickerUpdatesAsync(symbol, handler)`
- `SubscribeToTradeUpdatesAsync(symbol, handler)`
- `SubscribeToKlineUpdatesAsync(symbol, interval, handler)`
- `SubscribeToOrderBookUpdatesAsync(symbol, handler)`
- `SubscribeToPartialOrderBookUpdatesAsync(symbol, limit, handler)`
- `SubscribeToFundingRateUpdatesAsync(symbol, handler)`
- `SubscribeToIndexPriceUpdatesAsync(symbol, handler)`
- `SubscribeToMarkPriceUpdatesAsync(symbol, handler)`
- `SubscribeToSymbolUpdatesAsync(handler)`
- `SubscribeToUserDataUpdatesAsync(...)`

## SharedApis Interfaces

REST shared clients:

- `client.SpotApi.SharedClient`
- `client.FuturesApi.SharedClient`

Implemented spot REST shared interfaces:

- `IAssetsRestClient`
- `IBalanceRestClient`
- `IDepositRestClient`
- `IKlineRestClient`
- `IOrderBookRestClient`
- `IRecentTradeRestClient`
- `ISpotOrderRestClient`
- `ISpotSymbolRestClient`
- `ISpotTickerRestClient`
- `IWithdrawalRestClient`
- `IWithdrawRestClient`
- `IFeeRestClient`
- `IBookTickerRestClient`
- `ITransferRestClient`

Implemented futures REST shared interfaces:

- `IKlineRestClient`
- `IOrderBookRestClient`
- `IRecentTradeRestClient`
- `IFundingRateRestClient`
- `IFuturesSymbolRestClient`
- `IFuturesTickerRestClient`
- `IBalanceRestClient`
- `ILeverageRestClient`
- `IPositionModeRestClient`
- `IPositionHistoryRestClient`
- `IFeeRestClient`
- `IFuturesOrderRestClient`
- `IFuturesTriggerOrderRestClient`
- `IFuturesOrderClientIdRestClient`

Socket shared clients:

- `socket.SpotApi.SharedClient`
- `socket.FuturesApi.SharedClient`

Implemented spot socket shared interfaces:

- `ITradeSocketClient`
- `ITickerSocketClient`
- `ITickersSocketClient`
- `IBookTickerSocketClient`
- `IKlineSocketClient`
- `IOrderBookSocketClient`
- `IBalanceSocketClient`
- `ISpotOrderSocketClient`
- `IUserTradeSocketClient`

Implemented futures socket shared interfaces:

- `IKlineSocketClient`
- `IOrderBookSocketClient`
- `ITickerSocketClient`
- `ITickersSocketClient`
- `ITradeSocketClient`
- `IBalanceSocketClient`
- `IFuturesOrderSocketClient`
- `IPositionSocketClient`
- `IUserTradeSocketClient`

Call `SharedClient.Discover()` before relying on optional shared features.

## Symbols

- Spot symbols use names such as `BTCUSDT` and `ETHUSDT`.
- Futures symbols use names such as `BTC_USDT` and `ETH_USDT`.
- `MexcExchange.FormatSymbol("BTC", "USDT", TradingMode.Spot)` returns `BTCUSDT`.
- `MexcExchange.FormatSymbol("ETH", "USDT", TradingMode.PerpetualLinear)` returns `ETH_USDT`.
- SharedApis uses `SharedSymbol`, for example `new SharedSymbol(TradingMode.Spot, "BTC", "USDT")`.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Websocket subscriptions: `WebSocketResult<UpdateSubscription>`
- Shared non-I/O helpers: `ExchangeCallResult<T>`

Always check `Success` before using `Data`. Batch order methods can return nested per-item `CallResult<T>` values.
