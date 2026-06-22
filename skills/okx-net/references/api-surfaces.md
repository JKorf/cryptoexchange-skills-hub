# OKX.Net API Surfaces

Use this file when choosing native OKX client roots, endpoint groups, shared interfaces, result types, socket request methods, or symbol formats.

## Package And Client Types

- NuGet package: `JK.OKX.Net`
- REST client: `OKXRestClient`
- Socket client: `OKXSocketClient`
- REST interface: `IOKXRestClient`
- Socket interface: `IOKXSocketClient`
- Credentials: `OKXCredentials`
- DI extension: `services.AddOKX(...)`
- Tracker factory: `IOKXTrackerFactory`

## Client Roots

REST root:

- `client.UnifiedApi`

Socket root:

- `socket.UnifiedApi`

Do not use exchange roots such as `SpotApi`, `UsdFuturesApi`, `CoinFuturesApi`, `V5Api`, or `ExchangeApi`.

## Unified REST Endpoint Groups

`client.UnifiedApi.ExchangeData`:

- `Get24HourVolumeAsync()`
- `GetTickerAsync(symbol)`
- `GetTickersAsync(instrumentType, underlying, instrumentFamily)`
- `GetSymbolsAsync(instrumentType, underlying, symbol, instrumentFamily)`
- `GetOrderBookAsync(symbol, depth)`
- `GetRecentTradesAsync(symbol, limit)`
- `GetTradeHistoryAsync(symbol, type, startTime, endTime, limit)`
- `GetKlinesAsync(symbol, period, startTime, endTime, limit)`
- `GetKlineHistoryAsync(symbol, period, startTime, endTime, limit)`
- `GetIndexKlinesAsync(symbol, period, startTime, endTime, limit)`
- `GetMarkPriceKlinesAsync(symbol, period, startTime, endTime, limit)`
- `GetIndexTickersAsync(quoteAsset, symbol)`
- `GetMarkPricesAsync(instrumentType, underlying, symbol, instrumentFamily)`
- `GetOpenInterestsAsync(instrumentType, underlying, symbol, instrumentFamily)`
- `GetFundingRatesAsync(symbol)`
- `GetFundingRateHistoryAsync(symbol, startTime, endTime, limit)`
- `GetPriceLimitsAsync(symbol)`
- `GetPositionTiersAsync(...)`
- `GetOptionMarketDataAsync(underlying, expiryDate, instrumentFamily)`
- `GetServerTimeAsync()`
- `GetUnderlyingAsync(instrumentType)`
- `UnitConvertAsync(type, symbol, quantity, unit, price)`
- `GetAnnouncementsAsync(...)`
- `GetEstimatedFuturesSettlementPriceAsync(symbol)`
- `GetSettlementHistoryAsync(symbol, startTime, endTime, limit)`
- `GetPremiumHistoryAsync(instrumentId, startTime, endTime, limit)`

`client.UnifiedApi.Account`:

- `GetAccountBalanceAsync(asset)`
- `GetAccountConfigurationAsync()`
- `GetPositionsAsync(instrumentType, symbol, positionId)`
- `GetPositionHistoryAsync(...)`
- `GetPositionRiskAsync(instrumentType)`
- `GetLeverageAsync(symbols, marginMode, asset)`
- `SetLeverageAsync(leverage, marginMode, asset, symbol, positionSide)`
- `GetEstimatedLeverageInfoAsync(...)`
- `SetPositionModeAsync(positionMode)`
- `SetAccountModeAsync(mode)`
- `PrecheckAccountModeSwitchAsync(mode)`
- `PresetAccountModeSwitchAsync(mode, leverage, riskOffsetType)`
- `GetAssetsAsync(asset)`
- `GetFundingBalanceAsync(asset)`
- `GetDepositAddressAsync(asset)`
- `GetDepositHistoryAsync(...)`
- `GetWithdrawalHistoryAsync(...)`
- `WithdrawAsync(asset, amount, destination, toAddress, fee, network, areaCode, clientId)`
- `CancelWithdrawalAsync(withdrawalId)`
- `TransferAsync(asset, amount, type, fromAccount, toAccount, ...)`
- `GetTransferAsync(transferId, clientTransferId, type)`
- `GetFeeRatesAsync(...)`
- `GetMaximumAmountAsync(...)`
- `GetMaximumAvailableAmountAsync(...)`
- `GetMaximumWithdrawalsAsync(asset)`
- `SetMarginAmountAsync(...)`
- `SavingPurchaseRedemptionAsync(asset, amount, side, rate)`
- `EasyConvertDustAsync(assets, targetAsset, sourceAccount)`
- `ManualBorrowRepay(asset, side, quantity)`
- `SetAutoRepayAsync(autoRepay)`
- `GetBorrowRepayHistoryAsync(...)`
- `GetSymbolsAsync(instrumentType, underlying, symbol, instrumentFamily)`
- `GetGreeksAsync(asset)`

`client.UnifiedApi.Trading`:

- `PlaceOrderAsync(...)`
- `CheckOrderAsync(...)`
- `PlaceMultipleOrdersAsync(orders)`
- `CancelOrderAsync(symbol, orderId, clientOrderId)`
- `CancelMultipleOrdersAsync(orders)`
- `AmendOrderAsync(...)`
- `AmendMultipleOrdersAsync(orders)`
- `CancelAllAfterAsync(timeout, tag)`
- `ClosePositionAsync(symbol, marginMode, positionSide, asset, autoCancel, clientOrderId)`
- `GetOrderDetailsAsync(symbol, orderId, clientOrderId)`
- `GetOrdersAsync(...)`
- `GetOrderHistoryAsync(...)`
- `GetOrderArchiveAsync(...)`
- `GetUserTradesAsync(...)`
- `GetUserTradesArchiveAsync(...)`
- `PlaceAlgoOrderAsync(...)`
- `GetAlgoOrderAsync(algoId, clientAlgoId)`
- `GetAlgoOrderListAsync(...)`
- `GetAlgoOrderHistoryAsync(...)`
- `CancelAlgoOrderAsync(orders)`
- `AmendAlgoOrderAsync(...)`
- `GetAccountRateLimitAsync()`
- `OneClickRepayAsync(...)`
- `OneClickRepayV2Async(...)`

`client.UnifiedApi.SubAccounts`:

- `GetSubAccountsAsync(enable, subAccountName, endTime, startTime, limit)`
- `CreateSubAccountAsync(subAccountName, label)`
- `GetSubAccountFundingBalancesAsync(subAccountName, asset)`
- `GetSubAccountTradingBalancesAsync(subAccountName)`
- `GetSubAccountBillsAsync(...)`
- `GetManagedSubAccountBillsAsync(...)`
- `TransferBetweenSubAccountsAsync(...)`
- `GetSubAccountMaxWithdrawalsAsync(...)`
- `GetSubAccountApiKeysAsync(subAccountName, apiKey)`
- `CreateSubAccountApiKeyAsync(...)`
- `ResetSubAccountApiKeyAsync(...)`
- `DeleteSubAccountApiKeyAsync(...)`
- `SetSubAccountTransferOutAsync(subAccountName, canTransferOut)`

`client.UnifiedApi.CopyTrading`:

- `GetAccountConfigurationAsync()`
- `GetLeadPositionsAsync(...)`
- `GetLeadTraderCurrentLeadPositionsAsync(...)`
- `GetLeadPositionHistoryAsync(...)`
- `PlaceLeadStopOrderAsync(...)`
- `CloseLeadPositionAsync(subPositionId, instrumentType)`
- `GetLeadingInstrumentsAsync(instrumentType)`
- `AmendLeadingInstrumentsAsync(symbols, instrumentType)`

## Socket Subscriptions And Requests

`socket.UnifiedApi.ExchangeData` public streams:

- `SubscribeToTickerUpdatesAsync(symbols, handler)`
- `SubscribeToTradeUpdatesAsync(symbols, handler)`
- `SubscribeToKlineUpdatesAsync(symbols, period, handler)`
- `SubscribeToOrderBookUpdatesAsync(symbols, orderBookType, handler)`
- `SubscribeToFundingRateUpdatesAsync(symbols, handler)`
- `SubscribeToIndexTickerUpdatesAsync(symbols, handler)`
- `SubscribeToIndexKlineUpdatesAsync(symbol, period, handler)`
- `SubscribeToOpenInterestUpdatesAsync(symbols, handler)`
- `SubscribeToMarkPriceUpdatesAsync(symbols, handler)`
- `SubscribeToMarkPriceKlineUpdatesAsync(symbol, period, handler)`
- `SubscribeToEstimatedPriceUpdatesAsync(instrumentType, instrumentFamily, symbol, handler)`
- `SubscribeToPriceLimitUpdatesAsync(symbols, handler)`
- `SubscribeToSymbolUpdatesAsync(instrumentType, handler)`
- `SubscribeToSystemStatusUpdatesAsync(handler)`
- `SubscribeToOptionSummaryUpdatesAsync(instrumentFamily, handler)`

`socket.UnifiedApi.Account` private streams:

- `SubscribeToAccountUpdatesAsync(asset, regularUpdates, handler)`
- `SubscribeToBalanceAndPositionUpdatesAsync(handler)`
- `SubscribeToDepositUpdatesAsync(handler)`
- `SubscribeToWithdrawalUpdatesAsync(handler)`
- `SubscribeToGreeksUpdatesAsync(...)`

`socket.UnifiedApi.Trading` private streams:

- `SubscribeToOrderUpdatesAsync(instrumentType, symbol, instrumentFamily, handler)`
- `SubscribeToUserTradeUpdatesAsync(...)`
- `SubscribeToPositionUpdatesAsync(instrumentType, symbol, instrumentFamily, regularUpdates, handler)`
- `SubscribeToAlgoOrderUpdatesAsync(...)`
- `SubscribeToAdvanceAlgoOrderUpdatesAsync(...)`
- `SubscribeToLiquidationWarningUpdatesAsync(...)`

`socket.UnifiedApi.Trading` request methods return `QueryResult<T>`:

- `PlaceOrderAsync(symbolCode, side, type, tradeMode, quantity, ...)`
- `PlaceMultipleOrdersAsync(orders)`
- `CancelOrderAsync(symbolCode, orderId, clientOrderId)`
- `CancelMultipleOrdersAsync(ordersToCancel)`
- `AmendOrderAsync(symbolCode, ...)`
- `AmendMultipleOrdersAsync(ordersToCancel)`

Socket order requests are live trading requests. They use numeric `OKXInstrument.SymbolCode` values, not display symbol strings such as `ETH-USDT`. Retrieve symbol codes with `client.UnifiedApi.ExchangeData.GetSymbolsAsync(...)`.

## SharedApis Interfaces

REST shared client:

- `client.UnifiedApi.SharedClient`

Implemented REST shared interfaces:

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
- `IFuturesSymbolRestClient`
- `IFuturesOrderRestClient`
- `ILeverageRestClient`
- `IMarkPriceKlineRestClient`
- `IIndexPriceKlineRestClient`
- `IOpenInterestRestClient`
- `IFuturesTickerRestClient`
- `IFundingRateRestClient`
- `IPositionModeRestClient`
- `IPositionHistoryRestClient`
- `IFeeRestClient`
- `ISpotTriggerOrderRestClient`
- `IFuturesTriggerOrderRestClient`
- `IFuturesTpSlRestClient`
- `ISpotOrderClientIdRestClient`
- `IFuturesOrderClientIdRestClient`
- `IBookTickerRestClient`
- `ITransferRestClient`

Socket shared client:

- `socket.UnifiedApi.SharedClient`

Implemented socket shared interfaces:

- `ITickerSocketClient`
- `ITradeSocketClient`
- `IBookTickerSocketClient`
- `IKlineSocketClient`
- `IOrderBookSocketClient`
- `IBalanceSocketClient`
- `ISpotOrderSocketClient`
- `IFuturesOrderSocketClient`
- `IUserTradeSocketClient`
- `IPositionSocketClient`

Call `SharedClient.Discover()` before relying on optional shared features.

## Symbols

- Spot symbols use names such as `ETH-USDT`.
- Perpetual swap symbols use names such as `ETH-USDT-SWAP`.
- Delivery futures symbols use names such as `ETH-USDT-260327`.
- Options use OKX option instrument ids and often require `InstrumentType.Option`, `underlying`, or `instrumentFamily`.
- `OKXExchange.FormatSymbol("ETH", "USDT", TradingMode.Spot)` returns `ETH-USDT`.
- `OKXExchange.FormatSymbol("ETH", "USDT", TradingMode.PerpetualLinear)` returns `ETH-USDT-SWAP`.
- Websocket order request methods use `OKXInstrument.SymbolCode` from instrument metadata.
- Europe SharedApis futures formatting can use `SharedApiEuropeUseXPerps` and `OKXExchange.FormatSymbolEurope(...)` behavior.
- SharedApis uses `SharedSymbol`, for example `new SharedSymbol(TradingMode.Spot, "ETH", "USDT")`.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Websocket subscriptions: `WebSocketResult<UpdateSubscription>`
- Websocket request methods: `QueryResult<T>` or `QueryResult<CallResult<T>[]>`
- Shared non-I/O helpers: `ExchangeCallResult<T>`

Always check `Success` before using `Data`. Batch order methods can return nested per-item `CallResult<T>` values.
