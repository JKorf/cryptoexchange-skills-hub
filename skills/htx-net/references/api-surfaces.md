# HTX.Net API Surfaces

Use this file when choosing native HTX client roots, endpoint groups, shared interfaces, result types, or symbol formats.

## Package And Client Types

- NuGet package: `JKorf.HTX.Net`
- REST client: `HTXRestClient`
- Socket client: `HTXSocketClient`
- REST interface: `IHTXRestClient`
- Socket interface: `IHTXSocketClient`
- Credentials: `HTXCredentials`
- DI extension: `services.AddHTX(...)`
- Tracker factory: `IHTXTrackerFactory`

## Client Roots

REST roots:

- `client.SpotApi`
- `client.UsdtFuturesApi`
- `client.UsdtFuturesV5Api`

Socket roots:

- `socket.SpotApi`
- `socket.UsdtFuturesApi`
- `socket.UsdtFuturesV5Api`

Do not use exchange roots such as `UsdFuturesApi`, `CoinFuturesApi`, `UnifiedApi`, `V5Api`, or `ExchangeApi`.

## Spot REST Endpoint Groups

`client.SpotApi.ExchangeData`:

- `GetSystemStatusAsync()`
- `GetSymbolsAsync()`
- `GetAssetsAsync()`
- `GetSymbolConfigAsync(symbols)`
- `GetNetworksAsync(descFilter, asset)`
- `GetAssetsAndNetworksAsync(asset)`
- `GetTickersAsync()`
- `GetTickerAsync(symbol)`
- `GetKlinesAsync(symbol, period, limit)`
- `GetOrderBookAsync(symbol, mergeStep, limit)`
- `GetLastTradeAsync(symbol)`
- `GetTradeHistoryAsync(symbol, limit)`
- `GetSymbolDetails24HAsync(symbol)`
- `GetFullOrderBookAsync(symbol)`
- `GetSymbolStatusAsync()`
- `GetServerTimeAsync()`

`client.SpotApi.Account`:

- `GetUserIdAsync()`
- `GetAccountsAsync()`
- `GetBalancesAsync(accountId)`
- `GetPlatformValuationAsync(accountType, valuationAsset)`
- `GetAssetValuationAsync(accountType, valuationAsset, subUserId)`
- `GetAccountHistoryAsync(accountId, ...)`
- `GetAccountLedgerAsync(accountId, ...)`
- `TransferAsync(fromAccount, toAccount, asset, quantity, marginAccount)`
- `GetDepositAddressesAsync(asset)`
- `WithdrawAsync(address, asset, quantity, fee, network, addressTag, clientOrderId)`
- `GetWithdrawDepositHistoryAsync(type, asset, from, size, direction)`
- `GetTradingFeesAsync(symbols)`
- `GetPointBalanceAsync(subUserId)`
- `TransferPointsAsync(fromUserId, toUserId, groupId, quantity)`
- `GetWithdrawalQuotasAsync(asset)`
- `GetWithdrawalAddressesAsync(asset, network, note, limit, fromId)`
- `GetWithdrawalByClientOrderIdAsync(clientOrderId)`
- `CancelWithdrawalAsync(id)`
- `GetApiKeyInfoAsync(userId, apiKey)`
- `InternalTransferAsync(...)`

`client.SpotApi.Margin`:

- `TransferSpotToIsolatedMarginAsync(symbol, asset, quantity)`
- `TransferIsolatedMarginToSpotAsync(symbol, asset, quantity)`
- `RequestIsolatedMarginLoanAsync(symbol, asset, quantity)`
- `RepayIsolatedMarginLoanAsync(orderId, quantity)`
- `GetIsolatedMarginBalanceAsync(symbol, subUserId)`
- `TransferSpotToCrossMarginAsync(asset, quantity)`
- `TransferCrossMarginToSpotAsync(asset, quantity)`
- `RequestCrossMarginLoanAsync(asset, quantity)`
- `RepayCrossMarginLoanAsync(orderId, quantity)`
- `GetCrossMarginBalanceAsync(subUserId)`
- Margin rate, quota, closed-order, limit, and repayment-history methods

`client.SpotApi.Trading`:

- `PlaceOrderAsync(accountId, symbol, side, type, quantity, price, clientOrderId, source, stopPrice, stopOperator)`
- `PlaceMultipleOrderAsync(orders)`
- `PlaceMarginOrderAsync(...)`
- `GetOpenOrdersAsync(...)`
- `CancelOrderAsync(orderId)`
- `CancelOrderByClientOrderIdAsync(clientOrderId)`
- `CancelOrdersAsync(orderIds, clientOrderIds)`
- `CancelAllOrdersAsync(symbol)`
- `CancelOrdersByCriteriaAsync(...)`
- `GetOrderAsync(orderId)`
- `GetOrderByClientOrderIdAsync(clientOrderId)`
- `GetOrderTradesAsync(orderId)`
- `GetClosedOrdersAsync(...)`
- `GetUserTradesAsync(...)`
- `GetHistoricalOrdersAsync(...)`
- `PlaceConditionalOrderAsync(...)`
- `CancelConditionalOrdersAsync(clientOrderIds)`
- `GetOpenConditionalOrdersAsync(...)`
- `GetClosedConditionalOrdersAsync(...)`
- `GetConditionalOrderAsync(clientOrderId)`

Spot order placement and balances require an account id from `GetAccountsAsync()`.

## USDT Futures REST Endpoint Groups

Use `UsdtFuturesApi` for the established USDT futures API and SharedApis support. Contract codes use hyphens, for example `ETH-USDT`.

`client.UsdtFuturesApi.ExchangeData`:

- `GetContractsAsync(contractCode, supportMarginMode, pair, contractType, businessType)`
- `GetTickerAsync(contractCode)`
- `GetTickersAsync(contractCode, businessType)`
- `GetKlinesAsync(contractCode, interval, startTime, endTime, limit)`
- `GetOrderBookAsync(contractCode, mergeStep)`
- `GetRecentTradesAsync(contractCode, limit)`
- `GetBookTickerAsync(contractCode, type)`
- `GetFundingRateAsync(contractCode)`
- `GetFundingRatesAsync(contractCode)`
- `GetHistoricalFundingRatesAsync(contractCode, page, pageSize)`
- `GetSwapOpenInterestAsync(contractCode, pair, contractType, businessType)`
- `GetOpenInterestHistoryAsync(period, unit, contractCode, symbol, type, limit)`
- `GetSwapIndexPriceAsync(contractCode)`
- `GetMarkPriceKlinesAsync(contractCode, klineInterval, limit)`
- `GetPremiumIndexKlinesAsync(contractCode, interval, limit)`
- `GetBasisDataAsync(contractCode, interval, limit, basisPriceType)`
- `GetSwapPriceLimitationAsync(...)`
- `GetSwapRiskInfoAsync(...)`
- `GetContractElementsAsync(contractCode)`
- `GetServerTimeAsync()`

`client.UsdtFuturesApi.Account`:

- `GetAssetValuationAsync(asset)`
- `GetIsolatedMarginAccountInfoAsync(contractCode)`
- `GetCrossMarginAccountInfoAsync(marginAccount)`
- `GetCrossMarginAssetsAndPositionsAsync(marginAccount)`
- `GetIsolatedMarginPositionsAsync(contractCode)`
- `GetCrossMarginPositionsAsync(contractCode)`
- `GetIsolatedMarginAvailableLeverageAsync(contractCode)`
- `GetCrossMarginAvailableLeverageAsync(contractCode, pair, contractType, businessType)`
- `GetTradingFeesAsync(contractCode, pair, contractType, businessType)`
- `SetCrossMarginPositionModeAsync(marginAccount, positionMode)`
- `SetIsolatedMarginPositionModeAsync(marginAccount, positionMode)`
- `TransferMarginAccountsAsync(asset, fromMarginAccount, toMarginAccount, quantity, clientOrderId)`
- Limits, risk, settlement, financial-record, and trading-status methods

`client.UsdtFuturesApi.Trading`:

- `SetCrossMarginLeverageAsync(leverageRate, contractCode, pair, contractType)`
- `SetIsolatedMarginLeverageAsync(contractCode, leverageRate)`
- `PlaceCrossMarginOrderAsync(quantity, side, leverageRate, orderPriceType, contractCode, pair, contractType, price, offset, ..., reduceOnly, clientOrderId)`
- `PlaceIsolatedMarginOrderAsync(contractCode, quantity, side, leverageRate, orderPriceType, price, offset, ..., reduceOnly, clientOrderId)`
- `CancelCrossMarginOrderAsync(...)`
- `CancelIsolatedMarginOrderAsync(...)`
- `CancelAllCrossMarginOrdersAsync(...)`
- `CancelAllIsolatedMarginOrdersAsync(...)`
- `GetCrossMarginOpenOrdersAsync(...)`
- `GetIsolatedMarginOpenOrdersAsync(...)`
- `GetCrossMarginOrderAsync(...)`
- `GetIsolatedMarginOrderAsync(...)`
- `GetCrossMarginClosedOrdersAsync(...)`
- `GetIsolatedMarginClosedOrdersAsync(...)`
- `GetCrossMarginUserTradesAsync(...)`
- `GetIsolatedMarginUserTradesAsync(...)`
- `CloseCrossMarginPositionAsync(direction, contractCode, ..., orderPriceType)`
- `CloseIsolatedMarginPositionAsync(contractCode, direction, ..., orderPriceType)`
- `PlaceCrossMarginTriggerOrderAsync(...)`
- `PlaceIsolatedMarginTriggerOrderAsync(...)`
- TP/SL and trailing-order placement, cancellation, open-order, and history methods

USDT futures order `quantity` is contract quantity. Cross and isolated margin are separate method families.

## USDT Futures V5 REST And Socket

Use `UsdtFuturesV5Api` for native V5 futures features. It is not the SharedApis futures root.

`client.UsdtFuturesV5Api.ExchangeData`:

- `GetFundingRateAsync(contractCode)`
- `GetFundingRateHistoryAsync(contractCode, ...)`
- `GetOpenInterestAsync(contractCode)`
- `GetPriceLimitsAsync(contractCode)`
- `GetRiskLimitsAsync(contractCode, marginMode, tier)`

`client.UsdtFuturesV5Api.Account`:

- `GetAssetModeAsync()`
- `SetAssetModeAsync(assetMode)`
- `GetAccountBalanceAsync()`
- `GetBillsAsync(...)`
- `GetPositionModeAsync()`
- `SetPositionModeAsync(positionMode)`
- `GetLeverageAsync(contractCode, marginMode, positionSide)`
- `SetLeverageAsync(contractCode, marginMode, leverageRate, positionSide)`
- `GetOpenPositionsAsync(contractCode)`
- `GetRiskLimitsAsync(contractCode, marginMode, positionSide)`

`client.UsdtFuturesV5Api.Trading`:

- `PlaceOrderAsync(contractCode, marginMode, side, type, quantity, positionSide, price, priceMatch, clientOrderId, reduceOnly, timeInForce, ...)`
- `CancelOrderAsync(contractCode, orderId, clientOrderId)`
- `GetOrderAsync(contractCode, marginMode, orderId, clientOrderId)`
- `GetOpenOrdersAsync(...)`
- `GetOrderDetailsAsync(...)`
- `GetOrderHistoryAsync(...)`
- `PlaceAlgoOrderAsync(...)`
- `GetOpenAlgoOrdersAsync(...)`
- `GetAlgoOrderHistoryAsync(...)`
- `CancelAlgoOrdersAsync(...)`

`socket.UsdtFuturesV5Api` private streams:

- `SubscribeToAccountUpdatesAsync(handler)`
- `SubscribeToOrderUpdatesAsync(contractCode, handler)`
- `SubscribeToPositionUpdatesAsync(contractCode, handler)`
- `SubscribeToUserTradeUpdatesAsync(contractCode, handler)`

## Socket Subscriptions And Queries

`socket.SpotApi` public streams:

- `SubscribeToTickerUpdatesAsync(symbol, handler)`
- `SubscribeToTickerUpdatesAsync(handler)`
- `SubscribeToKlineUpdatesAsync(symbol, period, handler)`
- `SubscribeToBookTickerUpdatesAsync(symbol, handler)`
- `SubscribeToTradeUpdatesAsync(symbol, handler)`
- `SubscribeToPartialOrderBookUpdates100MillisecondAsync(symbol, levels, handler)`
- `SubscribeToPartialOrderBookUpdates1SecondAsync(symbol, mergeStep, handler)`
- `SubscribeToOrderBookChangeUpdatesAsync(symbol, levels, handler)`

`socket.SpotApi` socket queries return `QueryResult<T>`:

- `GetKlinesAsync(symbol, period)`
- `GetOrderBookWithMergeStepAsync(symbol, mergeStep)`
- `GetOrderBookAsync(symbol, levels)`
- `GetTradeHistoryAsync(symbol)`
- `GetSymbolDetailsAsync(symbol)`

`socket.SpotApi` private streams and socket order methods:

- `SubscribeToAccountUpdatesAsync(handler, updateMode)`
- `SubscribeToOrderUpdatesAsync(symbol, onOrderSubmitted, onOrderMatched, onOrderCancelation, ...)`
- `SubscribeToOrderDetailsUpdatesAsync(symbol, ...)`
- `PlaceOrderAsync(...)`
- `PlaceMultipleOrdersAsync(...)`
- `PlaceMarginOrderAsync(...)`
- `CancelAllOrdersAsync(...)`
- `CancelOrderAsync(orderId, clientOrderId)`
- `CancelOrdersAsync(...)`

`socket.UsdtFuturesApi` public streams:

- `SubscribeToTickerUpdatesAsync(contractCode, handler)`
- `SubscribeToTradeUpdatesAsync(contractCode, handler)`
- `SubscribeToBookTickerUpdatesAsync(contractCode, handler)`
- `SubscribeToKlineUpdatesAsync(contractCode, period, handler)`
- `SubscribeToOrderBookUpdatesAsync(contractCode, mergeStep, handler)`
- `SubscribeToIncrementalOrderBookUpdatesAsync(contractCode, snapshot, limit, handler)`
- `SubscribeToBasisUpdatesAsync(contractCode, period, priceType, handler)`
- Mark/index/premium kline, funding-rate, liquidation, contract, and system-status streams

`socket.UsdtFuturesApi` private streams:

- `SubscribeToOrderUpdatesAsync(mode, handler)`
- `SubscribeToIsolatedMarginBalanceUpdatesAsync(handler)`
- `SubscribeToCrossMarginBalanceUpdatesAsync(handler)`
- `SubscribeToIsolatedMarginPositionUpdatesAsync(handler)`
- `SubscribeToCrossMarginPositionUpdatesAsync(handler)`
- `SubscribeToIsolatedMarginUserTradeUpdatesAsync(handler)`
- `SubscribeToCrossMarginUserTradeUpdatesAsync(handler)`
- Trigger-order update streams

## SharedApis Interfaces

REST shared clients:

- `client.SpotApi.SharedClient`
- `client.UsdtFuturesApi.SharedClient`

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
- `ISpotTriggerOrderRestClient`
- `IBookTickerRestClient`
- `ITransferRestClient`

Implemented USDT futures REST shared interfaces:

- `IBalanceRestClient`
- `IFuturesTickerRestClient`
- `IFuturesSymbolRestClient`
- `IFuturesOrderRestClient`
- `IKlineRestClient`
- `IMarkPriceKlineRestClient`
- `IIndexPriceKlineRestClient`
- `IOrderBookRestClient`
- `IRecentTradeRestClient`
- `IFundingRateRestClient`
- `IOpenInterestRestClient`
- `IPositionModeRestClient`
- `IFeeRestClient`
- `IFuturesOrderClientIdRestClient`
- `IFuturesTriggerOrderRestClient`
- `IFuturesTpSlRestClient`
- `IBookTickerRestClient`

Socket shared clients:

- `socket.SpotApi.SharedClient`
- `socket.UsdtFuturesApi.SharedClient`

Implemented spot socket shared interfaces:

- `ITickerSocketClient`
- `ITickersSocketClient`
- `ITradeSocketClient`
- `IBookTickerSocketClient`
- `IKlineSocketClient`
- `IOrderBookSocketClient`
- `IBalanceSocketClient`
- `ISpotOrderSocketClient`
- `IUserTradeSocketClient`

Implemented USDT futures socket shared interfaces:

- `ITickerSocketClient`
- `ITradeSocketClient`
- `IBookTickerSocketClient`
- `IKlineSocketClient`
- `IOrderBookSocketClient`
- `IBalanceSocketClient`
- `IFuturesOrderSocketClient`
- `IUserTradeSocketClient`
- `IPositionSocketClient`

Call `SharedClient.Discover()` before relying on optional shared features.

## Symbols

- Native spot symbols use no separator: `ETHUSDT`.
- Native USDT futures contract codes use hyphens: `ETH-USDT`.
- `HTXExchange.FormatSymbol("ETH", "USDT", TradingMode.Spot)` returns lowercase `ethusdt`.
- `HTXExchange.FormatSymbol("ETH", "USDT", TradingMode.PerpetualLinear)` returns `ETH-USDT`.
- SharedApis uses `SharedSymbol`, for example `new SharedSymbol(TradingMode.Spot, "ETH", "USDT")`.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Websocket subscriptions: `WebSocketResult<UpdateSubscription>`
- Spot socket query/order methods: `QueryResult<T>` or `QueryResult`
- Shared non-I/O helpers: `ExchangeCallResult<T>`

Always check `Success` before using `Data`. Batch methods can return nested per-item results that still need inspection after outer success.
