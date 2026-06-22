# CryptoCom.Net API Surfaces

Use this file when choosing native Crypto.com client roots, endpoint groups, shared interfaces, result types, or symbol formats.

## Package And Client Types

- NuGet package: `CryptoCom.Net`
- REST client: `CryptoComRestClient`
- Socket client: `CryptoComSocketClient`
- REST interface: `ICryptoComRestClient`
- Socket interface: `ICryptoComSocketClient`
- Credentials: `CryptoComCredentials`
- DI extension: `services.AddCryptoCom(...)`

## Client Roots

REST root:

- `client.ExchangeApi`

Socket root:

- `socket.ExchangeApi`

Do not use exchange roots such as `SpotApi`, `UsdFuturesApi`, `CoinFuturesApi`, `FuturesApi`, `UnifiedApi`, or `V5Api`.

## REST Endpoint Groups

`client.ExchangeApi.ExchangeData`:

- `GetServerTimeAsync()`
- `GetRiskParametersAsync()`
- `GetSymbolsAsync()`
- `GetOrderBookAsync(symbol, depth)`
- `GetTickersAsync(symbol)`
- `GetTradeHistoryAsync(symbol, startTime, endTime, limit)`
- `GetKlinesAsync(symbol, interval, startTime, endTime, limit)`
- `GetValuationsAsync(symbol, type, startTime, endTime, limit)`
- `GetExpiredSettlementPriceAsync(symbolType, pageNumber)`
- `GetInsuranceAsync(asset, startTime, endTime, limit)`
- `GetAnnouncementsAsync(category, productType)`

`client.ExchangeApi.Account`:

- `GetBalancesAsync()`
- `GetBalanceHistoryAsync(interval, endTime, limit)`
- `GetAccountInfoAsync(page, pageSize)`
- `SetAccountLeverageAsync(accountId, leverage)`
- `SetAccountSettingsAsync(stpScope, stpMode, stpId, leverage)`
- `GetAccountSettingsAsync()`
- `GetTransactionHistoryAsync(symbol, transactionType, isolationId, startTime, endTime, limit)`
- `GetFeeRatesAsync()`
- `GetSymbolFeeRateAsync(symbol)`
- `WithdrawAsync(asset, quantity, address, addressTag, network, clientWithdrawId)`
- `GetAssetsAsync()`
- `GetDepositAddressesAsync(asset)`
- `GetDepositHistoryAsync(asset, startTime, endTime, status, page, pageSize)`
- `GetWithdrawalHistoryAsync(asset, startTime, endTime, status, page, pageSize)`
- `CreateIsolatedMarginTransferAsync(isolationId, direction, quantity)`
- `SetIsolatedMarginLeverageAsync(isolationId, leverage)`

`client.ExchangeApi.Trading`:

- `GetPositionsAsync(symbol)`
- `PlaceOrderAsync(symbol, side, type, quantity, quoteQuantity, price, clientOrderId, postOnly, timeInForce, triggerPrice, triggerPriceType, margin, selfTradePreventionScope, selfTradePreventionMode, selfTradePreventionId, smartPostOnly, isolatedMargin, isolationId, leverage, isolatedMarginQuantity, reduceOnly)`
- `CancelOrderAsync(orderId, clientOrderId)`
- `CancelAllOrdersAsync(symbol, type)`
- `ClosePositionAsync(symbol, orderType, price, isolationId)`
- `GetOpenOrdersAsync(symbol)`
- `GetOrderAsync(orderId, clientOrderId)`
- `GetClosedOrdersAsync(symbol, isolationId, startTime, endTime, limit)`
- `GetUserTradesAsync(symbol, isolationId, startTime, endTime, limit)`
- `PlaceMultipleOrdersAsync(orders)`
- `CancelOrdersAsync(orders)`
- `PlaceOcoOrderAsync(order1, order2)`
- `CancelOcoOrderAsync(symbol, listId)`
- `GetOcoOrderAsync(symbol, listId)`
- `EditOrderAsync(newQuantity, newPrice, orderId, clientOrderId)`

`client.ExchangeApi.Staking`:

- `StakeAsync(symbol, quantity)`
- `UnstakeAsync(symbol, quantity)`
- `GetStakingPositionsAsync(symbol)`
- `GetStakingSymbolsAsync()`
- `GetOpenStakingRequestsAsync(symbol, startTime, endTime, limit)`
- `GetStakingHistoryAsync(symbol, startTime, endTime, limit)`
- `GetStakingRewardHistoryAsync(symbol, startTime, endTime, limit)`
- `ConvertAsync(fromSymbol, toSymbol, expectedRate, quantity, slippageToleranceBps)`
- `GetOpenConvertRequestsAsync(startTime, endTime, limit)`
- `GetConvertRateAsync(symbol)`

## Socket Subscriptions

`socket.ExchangeApi` public streams:

- `SubscribeToOrderBookSnapshotUpdatesAsync(symbol, depth, handler)`
- `SubscribeToOrderBookUpdatesAsync(symbol, depth, handler)`
- `SubscribeToKlineUpdatesAsync(symbol, interval, handler)`
- `SubscribeToTickerUpdatesAsync(symbol, handler)`
- `SubscribeToTradeUpdatesAsync(symbol, handler)`
- `SubscribeToIndexPriceUpdatesAsync(symbol, handler)`
- `SubscribeToMarkPriceUpdatesAsync(symbol, handler)`
- `SubscribeToSettlementUpdatesAsync(...)`
- `SubscribeToFundingRateUpdatesAsync(symbol, handler)`
- `SubscribeToEstimatedFundingRateUpdatesAsync(symbol, handler)`

Most public stream methods also have `IEnumerable<string>` overloads for multiple symbols.

`socket.ExchangeApi` private streams:

- `SubscribeToOrderUpdatesAsync(...)`
- `SubscribeToUserTradeUpdatesAsync(...)`
- `SubscribeToBalanceUpdatesAsync(handler)`
- `SubscribeToPositionUpdatesAsync(handler)`
- `SubscribeToPositionBalanceUpdatesAsync(handler)`

Private streams require credentials on the socket client.

## Socket API Requests

The socket API also exposes private request methods that return `QueryResult<T>` or `QueryResult`:

- `GetBalancesAsync()`
- `GetPositionsAsync(symbol)`
- `PlaceOrderAsync(...)`
- `CancelOrderAsync(orderId, clientOrderId)`
- `CancelAllOrdersAsync(symbol, type)`
- `ClosePositionAsync(symbol, orderType, price, isolationId)`
- `GetOpenOrdersAsync(symbol)`
- `PlaceMultipleOrdersAsync(orders)`
- `CancelOrdersAsync(orders)`
- `PlaceOcoOrderAsync(order1, order2)`
- `CancelOcoOrderAsync(symbol, listId)`
- `WithdrawAsync(asset, quantity, address, addressTag, network, clientWithdrawId)`
- `SetCancelOnDisconnectAsync()`

Use REST methods by default unless the user specifically asks for socket request/response trading.

## SharedApis Interfaces

REST shared client: `client.ExchangeApi.SharedClient`

Implemented REST shared interfaces:

- `IAssetsRestClient`
- `IBalanceRestClient`
- `IDepositRestClient`
- `IKlineRestClient`
- `IOrderBookRestClient`
- `IRecentTradeRestClient`
- `IWithdrawalRestClient`
- `IWithdrawRestClient`
- `ISpotSymbolRestClient`
- `ISpotTickerRestClient`
- `ISpotOrderRestClient`
- `IFundingRateRestClient`
- `IFuturesSymbolRestClient`
- `IFuturesTickerRestClient`
- `ILeverageRestClient`
- `IOpenInterestRestClient`
- `IFuturesOrderRestClient`
- `IFeeRestClient`
- `ISpotOrderClientIdRestClient`
- `IFuturesOrderClientIdRestClient`
- `ISpotTriggerOrderRestClient`
- `IFuturesTriggerOrderRestClient`
- `IFuturesTpSlRestClient`
- `IBookTickerRestClient`

Socket shared client: `socket.ExchangeApi.SharedClient`

Implemented socket shared interfaces:

- `ITickerSocketClient`
- `IBookTickerSocketClient`
- `IKlineSocketClient`
- `IOrderBookSocketClient`
- `ITradeSocketClient`
- `IUserTradeSocketClient`
- `ISpotOrderSocketClient`
- `IFuturesOrderSocketClient`
- `IPositionSocketClient`
- `IBalanceSocketClient`

Supported shared trading modes include `TradingMode.Spot`, `TradingMode.PerpetualLinear`, and `TradingMode.DeliveryLinear`. Call `SharedClient.Discover()` before relying on optional shared features.

## Symbols

- Native spot symbols use underscore format: `BTC_USDT`, `ETH_USDT`.
- Derivative examples use exchange instrument names such as `ETHUSD_PERP`.
- `CryptoComExchange.FormatSymbol("BTC", "USDT", TradingMode.Spot)` returns `BTC_USDT`.
- `CryptoComExchange.FormatSymbol("ETH", "USD", TradingMode.PerpetualLinear)` returns `ETHUSD-PERP`; use `GetSymbolsAsync()` to confirm the exact live instrument name before trading.
- Delivery futures formatting requires a delivery date.
- SharedApis uses `SharedSymbol`, for example `new SharedSymbol(TradingMode.Spot, "BTC", "USDT")`.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Websocket subscriptions: `WebSocketResult<UpdateSubscription>`
- Socket API requests: `QueryResult<T>` or `QueryResult`
- Shared non-I/O helpers: `ExchangeCallResult<T>`
- Batch REST order placement: `HttpResult<CallResult<CryptoComOrderResult>[]>`
- Batch socket order placement: `QueryResult<CallResult<CryptoComListOrderResult>[]>`

Always check the outer result first. For batch methods, inspect each inner item after the outer result succeeds.
