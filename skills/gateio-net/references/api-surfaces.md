# GateIo.Net API Surfaces

Use this file when choosing native GateIo client roots, endpoint groups, shared interfaces, result types, or symbol formats.

## Package And Client Types

- NuGet package: `GateIo.Net`
- REST client: `GateIoRestClient`
- Socket client: `GateIoSocketClient`
- REST interface: `IGateIoRestClient`
- Socket interface: `IGateIoSocketClient`
- Credentials: `GateIoCredentials`
- DI extension: `services.AddGateIo(...)`

## Client Roots

REST roots:

- `client.SpotApi`
- `client.PerpetualFuturesApi`
- `client.RebateApi`
- `client.AlphaApi`

Socket roots:

- `socket.SpotApi`
- `socket.PerpetualFuturesApi`

Do not use exchange roots such as `UsdFuturesApi`, `CoinFuturesApi`, `UnifiedApi`, `V5Api`, or `ExchangeApi`.

## Spot REST Endpoint Groups

`client.SpotApi.ExchangeData`:

- `GetServerTimeAsync()`
- `GetAssetsAsync()`
- `GetAssetAsync(asset)`
- `GetSymbolsAsync()`
- `GetSymbolAsync(symbol)`
- `GetTickersAsync(symbol, timezone)`
- `GetOrderBookAsync(symbol, mergeDepth, limit)`
- `GetTradesAsync(symbol, limit, lastId, reverse, startTime, endTime, page)`
- `GetKlinesAsync(symbol, interval, startTime, endTime, limit)`
- `GetNetworksAsync(asset)`
- `GetDiscountTiersAsync()`
- `GetLendingSymbolsAsync()`
- `GetLendingSymbolAsync(symbol)`

`client.SpotApi.Account`:

- `GetBalancesAsync(asset)`
- `GetLedgerAsync(...)`
- `GetDepositsAsync(...)`
- `GetWithdrawalsAsync(...)`
- `WithdrawAsync(asset, quantity, address, network, memo, clientOrderId)`
- `CancelWithdrawalAsync(withdrawalId)`
- `GenerateDepositAddressAsync(asset)`
- `TransferAsync(asset, from, to, quantity, marginSymbol, settleAsset)`
- `GetTransferStatusAsync(clientOrderId, transactionId)`
- `GetWithdrawStatusAsync(asset)`
- `GetSavedAddressAsync(asset, network, limit, page)`
- `GetTradingFeeAsync(symbol, settleAsset)`
- `GetAccountBalancesAsync(valuationAsset)`
- `GetSmallBalancesAsync()`
- `ConvertSmallBalancesAsync(...)`
- `GetSmallBalanceConversionsAsync(...)`
- `GetUnifiedAccountInfoAsync()`
- `GetUnifiedAccountBorrowableAsync(...)`
- `GetUnifiedAccountTransferableAsync(...)`
- `UnifiedAccountBorrowOrRepayAsync(...)`
- `GetUnifiedAccountLoansAsync(...)`
- `GetUnifiedAccountLoanHistoryAsync(...)`
- Margin borrow/repay, leverage, loan, repayment, and risk methods

`client.SpotApi.Trading`:

- `PlaceOrderAsync(symbol, side, type, quantity, price, timeInForce, ...)`
- `PlaceMultipleOrderAsync(orders)`
- `GetOpenOrdersAsync(page, limit, accountType)`
- `GetOrdersAsync(open, symbol, page, limit, accountType, startTime, endTime, side)`
- `GetOrderAsync(symbol, orderId, clientOrderId, accountType)`
- `EditOrderAsync(symbol, orderId, clientOrderId, price, quantity, ...)`
- `CancelOrderAsync(symbol, orderId, clientOrderId, accountType)`
- `CancelOrdersAsync(cancelRequests)`
- `CancelAllOrdersAsync(symbol, side, accountType)`
- `CancelOrdersAfterAsync(cancelAfter, symbol)`
- `GetUserTradesAsync(...)`
- `PlaceTriggerOrderAsync(...)`
- `GetTriggerOrdersAsync(open, symbol, accountType, limit, offset)`
- `GetTriggerOrderAsync(id)`
- `CancelTriggerOrderAsync(id)`
- `CancelAllTriggerOrdersAsync(symbol, accountType)`

## Perpetual Futures REST Endpoint Groups

Most native futures methods require `settlementAsset`, commonly `"usdt"`, `"btc"`, or `"usd"`, before the contract.

`client.PerpetualFuturesApi.ExchangeData`:

- `GetServerTimeAsync()`
- `GetContractsAsync(settlementAsset)`
- `GetContractAsync(settlementAsset, contract)`
- `GetOrderBookAsync(settlementAsset, contract, mergeDepth, depth)`
- `GetTradesAsync(settlementAsset, contract, limit, lastId, startTime, endTime)`
- `GetKlinesAsync(settlementAsset, contract, interval, startTime, endTime, limit)`
- `GetIndexKlinesAsync(settlementAsset, contract, interval, startTime, endTime, limit)`
- `GetTickersAsync(settlementAsset, contract)`
- `GetFundingRateHistoryAsync(settlementAsset, contract, limit)`
- `GetInsuranceBalanceHistoryAsync(settlementAsset, limit)`
- `GetContractStatsAsync(settlementAsset, contract, startTime, interval, limit)`
- `GetIndexConstituentsAsync(settlementAsset, index)`
- `GetLiquidationsAsync(settlementAsset, contract, startTime, endTime, limit)`
- `GetRiskLimitTiersAsync(settlementAsset, contract)`

`client.PerpetualFuturesApi.Account`:

- `GetAccountAsync(settlementAsset)`
- `GetLedgerAsync(settlementAsset, contract, startTime, endTime, page, limit, type)`
- `UpdatePositionModeAsync(settlementAsset, dualMode)`
- `SetMarginModeAsync(settlementAsset, contract, marginMode)`
- `GetTradingFeeAsync(settlementAsset, contract)`

`client.PerpetualFuturesApi.Trading`:

- `GetPositionsAsync(settlementAsset, holding, offset, limit)`
- `GetPositionAsync(settlementAsset, contract)`
- `UpdatePositionMarginAsync(settlementAsset, contract, change)`
- `UpdatePositionLeverageAsync(settlementAsset, contract, leverage, crossLeverageLimit)`
- `UpdatePositionRiskLimitAsync(settlementAsset, contract, riskLimit)`
- `GetDualModePositionsAsync(settlementAsset, contract)`
- `UpdateDualModePositionMarginAsync(settlementAsset, contract, change, mode)`
- `UpdateDualModePositionLeverageAsync(settlementAsset, contract, leverage, crossLeverageLimit)`
- `UpdateDualModePositionRiskLimitAsync(settlementAsset, contract, riskLimit)`
- `PlaceOrderAsync(settlementAsset, contract, orderSide, quantity, price, ...)`
- `PlaceMultipleOrderAsync(settlementAsset, requests)`
- `GetOrdersAsync(settlementAsset, contract, status, ...)`
- `CancelOrdersAfterAsync(settlementAsset, cancelAfter, contract)`
- `CancelOrdersAsync(settlementAsset, orderIds)`
- `GetOrdersByTimestampAsync(...)`
- `CancelAllOrdersAsync(settlementAsset, contract, side)`
- `GetOrderAsync(settlementAsset, orderId, clientOrderId)`
- `CancelOrderAsync(settlementAsset, orderId, clientOrderId)`
- `EditOrderAsync(settlementAsset, orderId, contract, price, quantity, ...)`
- `EditMultipleOrdersAsync(settlementAsset, requests)`
- `GetUserTradesAsync(...)`
- `GetPositionCloseHistoryAsync(...)`
- `GetLiquidationHistoryAsync(...)`
- `GetAutoDeleveragingHistoryAsync(...)`
- `PlaceTriggerOrderAsync(...)`
- `GetTriggerOrdersAsync(...)`
- `CancelTriggerOrdersAsync(...)`
- `GetTriggerOrderAsync(...)`
- `CancelTriggerOrderAsync(...)`

Futures order `quantity` is an `int` number of contracts. Use `GetContractAsync` to inspect multiplier and sizing constraints.

## Socket Subscriptions And Requests

`socket.SpotApi` public streams:

- `SubscribeToTradeUpdatesAsync(symbols, handler)`
- `SubscribeToTickerUpdatesAsync(symbols, handler)`
- `SubscribeToKlineUpdatesAsync(symbol, interval, handler)`
- `SubscribeToBookTickerUpdatesAsync(symbols, handler)`
- `SubscribeToOrderBookUpdatesAsync(symbol, handler)`
- `SubscribeToOrderBookV2UpdatesAsync(symbol, depth, handler)`
- `SubscribeToPartialOrderBookUpdatesAsync(symbol, depth, updateMs, handler)`

`socket.SpotApi` private streams:

- `SubscribeToOrderUpdatesAsync(handler)`
- `SubscribeToUserTradeUpdatesAsync(handler)`
- `SubscribeToBalanceUpdatesAsync(handler)`
- `SubscribeToMarginBalanceUpdatesAsync(handler)`
- `SubscribeToFundingBalanceUpdatesAsync(handler)`
- `SubscribeToCrossMarginBalanceUpdatesAsync(handler)`
- `SubscribeToTriggerOrderUpdatesAsync(handler)`

`socket.SpotApi` socket request methods return `QueryResult<T>`:

- `PlaceOrderAsync(...)`
- `PlaceMultipleOrdersAsync(...)`
- `CancelOrderAsync(...)`
- `CancelOrdersAsync(...)`
- `CancelAllOrdersAsync(...)`
- `EditOrderAsync(...)`
- `GetOrderAsync(...)`
- `GetOrdersAsync(...)`

`socket.PerpetualFuturesApi` public streams:

- `SubscribeToTradeUpdatesAsync(settlementAsset, contracts, handler)`
- `SubscribeToTickerUpdatesAsync(settlementAsset, contracts, handler)`
- `SubscribeToBookTickerUpdatesAsync(settlementAsset, contracts, handler)`
- `SubscribeToOrderBookV2UpdatesAsync(settlementAsset, contract, depth, handler)`
- `SubscribeToOrderBookUpdatesAsync(settlementAsset, contract, updateMs, depth, handler)`
- `SubscribeToKlineUpdatesAsync(settlementAsset, contract, interval, handler)`
- `SubscribeToContractStatsUpdatesAsync(settlementAsset, contract, interval, handler)`

`socket.PerpetualFuturesApi` private streams require `userId` for most user-specific streams:

- `SubscribeToOrderUpdatesAsync(userId, settlementAsset, handler)`
- `SubscribeToUserTradeUpdatesAsync(userId, settlementAsset, handler)`
- `SubscribeToUserLiquidationUpdatesAsync(userId, settlementAsset, handler)`
- `SubscribeToUserAutoDeleverageUpdatesAsync(userId, settlementAsset, handler)`
- `SubscribeToPositionCloseUpdatesAsync(userId, settlementAsset, handler)`
- `SubscribeToBalanceUpdatesAsync(userId, settlementAsset, handler)`
- `SubscribeToReduceRiskLimitUpdatesAsync(userId, settlementAsset, handler)`
- `SubscribeToPositionUpdatesAsync(userId, settlementAsset, handler)`
- `SubscribeToTriggerOrderUpdatesAsync(userId, settlementAsset, handler)`
- `SubscribeToAdlUpdatesAsync(settlementAsset, handler)`

`socket.PerpetualFuturesApi` socket request methods return `QueryResult<T>`:

- `PlaceOrderAsync(...)`
- `PlaceMultipleOrderAsync(...)`
- `CancelOrderAsync(...)`
- `GetOrderAsync(...)`
- `GetOrdersAsync(...)`
- `CancelOrdersAsync(...)`
- `EditOrderAsync(...)`

## SharedApis Interfaces

REST shared clients:

- `client.SpotApi.SharedClient`
- `client.PerpetualFuturesApi.SharedClient`

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

Implemented perpetual futures REST shared interfaces:

- `IBalanceRestClient`
- `IFuturesTickerRestClient`
- `IFuturesSymbolRestClient`
- `IFuturesOrderRestClient`
- `IKlineRestClient`
- `IIndexPriceKlineRestClient`
- `IRecentTradeRestClient`
- `ITradeHistoryRestClient`
- `ILeverageRestClient`
- `IOrderBookRestClient`
- `IOpenInterestRestClient`
- `IFundingRateRestClient`
- `IPositionModeRestClient`
- `IPositionHistoryRestClient`
- `IFeeRestClient`
- `IFuturesOrderClientIdRestClient`
- `IFuturesTriggerOrderRestClient`
- `IFuturesTpSlRestClient`
- `IBookTickerRestClient`

Socket shared clients:

- `socket.SpotApi.SharedClient`
- `socket.PerpetualFuturesApi.SharedClient`

Implemented spot socket shared interfaces:

- `ITickerSocketClient`
- `ITradeSocketClient`
- `IBookTickerSocketClient`
- `IKlineSocketClient`
- `IOrderBookSocketClient`
- `IBalanceSocketClient`
- `IUserTradeSocketClient`
- `ISpotOrderSocketClient`

Implemented perpetual futures socket shared interfaces:

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

- Native Gate.io symbols use underscores: `ETH_USDT`.
- Perpetual futures methods use contract symbols such as `ETH_USDT` plus a settlement asset such as `"usdt"`.
- `GateIoExchange.FormatSymbol("ETH", "USDT", TradingMode.Spot)` returns `ETH_USDT`.
- `GateIoExchange.FormatSymbol("ETH", "USDT", TradingMode.PerpetualLinear)` returns `ETH_USDT`.
- SharedApis uses `SharedSymbol`, for example `new SharedSymbol(TradingMode.Spot, "ETH", "USDT")`.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Websocket subscriptions: `WebSocketResult<UpdateSubscription>`
- Websocket request/order methods: `QueryResult<T>` or `QueryResult`
- Shared non-I/O helpers: `ExchangeCallResult<T>`

Always check `Success` before using `Data`. Batch order and cancellation methods may return arrays of per-item results that still need inspection after outer success.
