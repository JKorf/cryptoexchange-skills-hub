# Kraken.Net API Surfaces

Use this file when choosing native Kraken client roots, endpoint groups, shared interfaces, result types, or symbol formats.

## Package And Client Types

- NuGet package: `KrakenExchange.Net`
- REST client: `KrakenRestClient`
- Socket client: `KrakenSocketClient`
- REST interface: `IKrakenRestClient`
- Socket interface: `IKrakenSocketClient`
- Credentials: `KrakenCredentials`
- DI extension: `services.AddKraken(...)`
- Tracker factory: `IKrakenTrackerFactory`

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

- `GetServerTimeAsync()`
- `GetSystemStatusAsync()`
- `GetAssetsAsync(assets, aClass, newAssetNameResponse)`
- `GetSymbolsAsync(symbols, countryCode, aClass, newAssetNameResponse, executionVenue)`
- `GetTickerAsync(symbol)`
- `GetTickersAsync(symbols, assetClass)`
- `GetKlinesAsync(symbol, interval, since, aClass)`
- `GetOrderBookAsync(symbol, limit, aClass)`
- `GetTradeHistoryAsync(symbol, since, limit, aClass)`
- `GetRecentSpreadAsync(symbol, since, aClass)`

`client.SpotApi.Account`:

- `GetBalancesAsync(newAssetNameResponse, twoFactorPassword)`
- `GetAvailableBalancesAsync(twoFactorPassword)`
- `GetTradeBalanceAsync(baseAsset, twoFactorPassword)`
- `GetOpenPositionsAsync(transactionIds, twoFactorPassword)`
- `GetLedgerInfoAsync(...)`
- `GetLedgersEntryAsync(ledgerIds, twoFactorPassword)`
- `GetTradeVolumeAsync(symbols, twoFactorPassword)`
- `GetDepositMethodsAsync(...)`
- `GetDepositAddressesAsync(...)`
- `GetDepositStatusAsync(...)`
- `GetDepositHistoryAsync(...)`
- `GetWithdrawInfoAsync(asset, key, quantity, ...)`
- `WithdrawAsync(asset, key, quantity, address, assetClass, twoFactorPassword)`
- `GetWithdrawAddressesAsync(...)`
- `GetWithdrawMethodsAsync(...)`
- `GetWebsocketTokenAsync()`
- `GetWithdrawalStatusAsync(...)`
- `GetWithdrawalHistoryAsync(...)`
- `CancelWithdrawalAsync(asset, referenceId, twoFactorPassword)`
- `TransferAsync(asset, quantity, fromWallet, toWallet, twoFactorPassword)`
- `GetApiKeyInfoAsync()`

`client.SpotApi.Trading`:

- `GetOpenOrdersAsync(userReference, clientOrderId, twoFactorPassword)`
- `GetClosedOrdersAsync(...)`
- `GetOrderAsync(orderId, clientOrderId, consolidateTaker, trades, twoFactorPassword)`
- `GetOrdersAsync(orderIds, clientOrderId, consolidateTaker, trades, twoFactorPassword)`
- `GetUserTradesAsync(...)`
- `GetUserTradeDetailsAsync(tradeId, twoFactorPassword)`
- `PlaceOrderAsync(symbol, side, type, quantity, price, ..., validateOnly, clientOrderId, twoFactorPassword, ...)`
- `PlaceMultipleOrdersAsync(...)`
- `EditOrderAsync(...)`
- `CancelOrderAsync(orderId, clientOrderId, twoFactorPassword)`
- `CancelAllOrdersAsync(twoFactorPassword)`
- `CancelAllOrdersAfterAsync(cancelAfter, twoFactorPassword)`
- `CancelMultipleOrdersAsync(orderIds, clientOrderIds, twoFactorPassword)`

`client.SpotApi.Earn`:

- `GetStrategiesAsync(asset, lockType, cursor, limit, asc, twoFactorPassword)`
- `GetAllocationsAsync(convertAsset, hideZeroAllocations, asc, twoFactorPassword)`
- `GetAllocationStatusAsync(strategyId, twoFactorPassword)`
- `GetDeallocationStatusAsync(strategyId, twoFactorPassword)`
- `AllocateEarnFundsAsync(strategyId, quantity, twoFactorPassword)`
- `DeallocateEarnFundsAsync(strategyId, quantity, twoFactorPassword)`

## Futures REST Endpoint Groups

`client.FuturesApi.ExchangeData`:

- `GetFeeSchedulesAsync()`
- `GetHistoricalFundingRatesAsync(symbol)`
- `GetKlinesAsync(tickType, symbol, interval, startTime, endTime, limit)`
- `GetOrderBookAsync(symbol)`
- `GetPlatformNotificationsAsync()`
- `GetSymbolsAsync()`
- `GetSymbolStatusAsync()`
- `GetTickerAsync(symbol)`
- `GetTickersAsync()`
- `GetTradesAsync(symbol, startTime)`

`client.FuturesApi.Account`:

- `GetAccountLogAsync(startTime, endTime, fromId, toId, sort, type, limit)`
- `GetBalancesAsync()`
- `GetPnlCurrencyAsync()`
- `SetPnlCurrencyAsync(symbol, pnlCurrency)`
- `TransferAsync(asset, quantity, fromAccount, toAccount)`
- `GetFeeScheduleVolumeAsync()`
- `GetInitialMarginRequirementsAsync(symbol, orderType, side, quantity, price)`
- `GetMaxOrderQuantityAsync(symbol, orderType, price)`

`client.FuturesApi.Trading`:

- `CancelAllOrderAfterAsync(cancelAfter)`
- `CancelAllOrdersAsync(symbol)`
- `CancelOrderAsync(orderId, clientOrderId)`
- `EditOrderAsync(orderId, clientOrderId, quantity, price, stopPrice, trailingStopDeviationUnit, trailingStopMaxDeviation)`
- `GetExecutionEventsAsync(...)`
- `GetLeverageAsync()`
- `GetOpenOrdersAsync()`
- `GetOpenPositionsAsync()`
- `GetOrderAsync(orderId, clientOrderId)`
- `GetOrdersAsync(orderIds, clientOrderIds)`
- `GetSelfTradeStrategyAsync()`
- `GetUserTradesAsync(startTime)`
- `PlaceOrderAsync(symbol, side, type, quantity, price, stopPrice, reduceOnly, trailingStopDeviationUnit, trailingStopMaxDeviation, triggerSignal, clientOrderId)`
- `SetLeverageAsync(symbol, maxLeverage)`
- `SetSelfTradeStrategyAsync(strategy)`
- `GetOrderHistoryAsync(...)`

Futures symbols use Kraken futures names such as `PF_ETHUSD`.

## Socket Subscriptions And Requests

`socket.SpotApi` public streams:

- `SubscribeToSystemStatusUpdatesAsync(handler)`
- `SubscribeToTickerUpdatesAsync(symbols, handler, eventTrigger, snapshot)`
- `SubscribeToKlineUpdatesAsync(symbols, interval, handler, snapshot)`
- `SubscribeToTradeUpdatesAsync(symbols, handler, snapshot)`
- `SubscribeToAggregatedOrderBookUpdatesAsync(symbols, depth, handler, snapshot)`
- `SubscribeToIndividualOrderBookUpdatesAsync(symbols, depth, handler, snapshot)`
- `SubscribeToInstrumentUpdatesAsync(handler, snapshot)`

`socket.SpotApi` private streams:

- `SubscribeToBalanceUpdatesAsync(snapshotHandler, updateHandler, snapshot)`
- `SubscribeToOrderUpdatesAsync(...)`

`socket.SpotApi` websocket v2 order methods return `QueryResult<T>`:

- `PlaceOrderAsync(...)`
- `EditOrderAsync(...)`
- `ReplaceOrderAsync(...)`
- `PlaceMultipleOrdersAsync(...)`
- `CancelOrderAsync(orderId)`
- `CancelOrdersAsync(orderIds)`
- `CancelAllOrdersAsync()`
- `CancelAllOrdersAfterAsync(timeout)`

`socket.FuturesApi` streams:

- `SubscribeToAccountLogUpdatesAsync(snapshotHandler, updateHandler)`
- `SubscribeToBalanceUpdatesAsync(handler)`
- `SubscribeToHeartbeatUpdatesAsync(handler)`
- `SubscribeToMiniTickerUpdatesAsync(symbols, handler)`
- `SubscribeToNotificationUpdatesAsync(handler)`
- `SubscribeToOpenOrdersUpdatesAsync(verbose, snapshotHandler, updateHandler)`
- `SubscribeToOpenPositionUpdatesAsync(handler)`
- `SubscribeToOrderBookUpdatesAsync(symbols, snapshotHandler, updateHandler)`
- `SubscribeToTickerUpdatesAsync(symbols, handler)`
- `SubscribeToTradeUpdatesAsync(symbols, updateHandler)`
- `SubscribeToUserTradeUpdatesAsync(handler)`

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
- `ISpotOrderClientIdRestClient`
- `IBookTickerRestClient`

Implemented futures REST shared interfaces:

- `IBalanceRestClient`
- `IKlineRestClient`
- `IOrderBookRestClient`
- `IRecentTradeRestClient`
- `IFundingRateRestClient`
- `IFuturesSymbolRestClient`
- `IFuturesTickerRestClient`
- `IMarkPriceKlineRestClient`
- `IOpenInterestRestClient`
- `ILeverageRestClient`
- `IFuturesOrderRestClient`
- `IFeeRestClient`
- `IFuturesOrderClientIdRestClient`
- `IFuturesTpSlRestClient`
- `IBookTickerRestClient`

Socket shared clients:

- `socket.SpotApi.SharedClient`
- `socket.FuturesApi.SharedClient`

Implemented spot socket shared interfaces:

- `ITickerSocketClient`
- `ITradeSocketClient`
- `IBookTickerSocketClient`
- `IKlineSocketClient`
- `IBalanceSocketClient`
- `ISpotOrderSocketClient`

Implemented futures socket shared interfaces:

- `ITickerSocketClient`
- `ITradeSocketClient`
- `IBookTickerSocketClient`
- `IBalanceSocketClient`
- `IFuturesOrderSocketClient`
- `IUserTradeSocketClient`
- `IPositionSocketClient`

Call `SharedClient.Discover()` before relying on optional shared features.

## Symbols

- REST spot symbols use names such as `ETHUSDT`.
- Spot websocket symbols use names such as `ETH/USDT`.
- Use `KrakenSymbol.WebsocketName` from `SpotApi.ExchangeData.GetSymbolsAsync(...)` when converting REST symbols to websocket symbols.
- Futures symbols use names such as `PF_ETHUSD`.
- `KrakenExchange.FormatSymbol("ETH", "USDT", TradingMode.Spot)` returns `ETHUSDT`.
- `KrakenExchange.FormatSymbol("ETH", "USD", TradingMode.PerpetualLinear)` returns `PF_ETHUSD`.
- SharedApis uses `SharedSymbol`, for example `new SharedSymbol(TradingMode.Spot, "ETH", "USDT")`.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Websocket subscriptions: `WebSocketResult<UpdateSubscription>`
- Spot websocket request/order methods: `QueryResult<T>` or `QueryResult`
- Shared non-I/O helpers: `ExchangeCallResult<T>`

Always check `Success` before using `Data`. Batch order methods can return nested per-item `CallResult<T>` values.
