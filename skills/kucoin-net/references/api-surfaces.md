# Kucoin.Net API Surfaces

Use this file when choosing native Kucoin client roots, endpoint groups, shared interfaces, result types, or symbol formats.

## Package And Client Types

- NuGet package: `Kucoin.Net`
- REST client: `KucoinRestClient`
- Socket client: `KucoinSocketClient`
- REST interface: `IKucoinRestClient`
- Socket interface: `IKucoinSocketClient`
- Credentials: `KucoinCredentials`
- DI extension: `services.AddKucoin(...)`
- Tracker factory: `IKucoinTrackerFactory`

## Client Roots

REST roots:

- `client.SpotApi`
- `client.FuturesApi`
- `client.UnifiedApi`

Socket roots:

- `socket.SpotApi`
- `socket.FuturesApi`
- `socket.UnifiedApi`

Do not use exchange roots such as `UsdFuturesApi`, `CoinFuturesApi`, `V5Api`, or `ExchangeApi`.

## Spot REST Endpoint Groups

`client.SpotApi.ExchangeData`:

- `GetServerTimeAsync()`
- `GetServiceStatusAsync()`
- `GetSymbolsAsync(...)`
- `GetSymbolAsync(symbol)`
- `GetTickerAsync(symbol)`
- `GetTickersAsync()`
- `GetKlinesAsync(symbol, interval, startTime, endTime)`
- `GetAggregatedPartialOrderBookAsync(symbol, depth)`
- `GetAggregatedFullOrderBookAsync(symbol)`
- `GetTradeHistoryAsync(symbol)`
- `GetFiatPricesAsync(...)`
- `GetCallAuctionPartOrderBookAsync(symbol)`
- `GetCallAuctionInfoAsync(symbol)`

`client.SpotApi.Account`:

- `GetUserInfoAsync()`
- `GetAccountsAsync(asset, accountType)`
- `GetAccountAsync(accountId)`
- `GetBasicUserFeeAsync(...)`
- `GetSymbolTradingFeesAsync(symbols)`
- `GetAccountLedgersAsync(...)`
- `GetTransferableAsync(asset, accountType, isolatedMarginSymbol)`
- `UniversalTransferAsync(...)`
- `GetDepositsAsync(...)`
- `GetDepositAddressesV3Async(asset, networkId, quantity)`
- `CreateDepositAddressV3Async(asset, networkId, toAccount, quantity)`
- `GetWithdrawalsAsync(...)`
- `GetWithdrawalAsync(id)`
- `GetWithdrawalQuotasAsync(asset, network)`
- `WithdrawAsync(withdrawalType, asset, toAddress, quantity, ...)`
- `CancelWithdrawalAsync(withdrawalId)`
- `GetMarginAccountAsync()`
- `GetCrossMarginAccountsAsync(quoteAsset)`
- `GetIsolatedMarginAccountsAsync()`
- `GetIsolatedMarginAccountAsync(symbol)`
- `GetApiKeyInfoAsync()`

`client.SpotApi.SubAccount`:

- `GetSubAccountsAsync()`
- `CreateSubAccountAsync(subName, password, permissions, remarks)`
- `GetSubAccountBalancesAsync(subAccountId, includeZeroBalances)`
- `GetSubAccountsBalancesAsync(page, pageSize)`
- `GetSubAccountApiKeyAsync(subAccountName, apiKey)`
- `CreateSubAccountApiKeyAsync(...)`
- `EditSubAccountApiKeyAsync(...)`
- `DeleteSubAccountApiKeyAsync(...)`
- `EnableMarginPermissionsAsync(subAccountId)`
- `EnableFuturesPermissionsAsync(subAccountId)`

`client.SpotApi.Trading`:

- `PlaceOrderAsync(...)`
- `PlaceTestOrderAsync(...)`
- `PlaceMarginOrderAsync(...)`
- `PlaceTestMarginOrderAsync(...)`
- `PlaceOcoOrderAsync(...)`
- `PlaceBulkOrderAsync(symbol, orders)`
- `CancelOrderAsync(orderId)`
- `CancelOrderByClientOrderIdAsync(clientOrderId)`
- `CancelAllOrdersAsync(symbol, tradeType)`
- `GetOrdersAsync(...)`
- `GetRecentOrdersAsync()`
- `GetOrderAsync(orderId)`
- `GetOrderByClientOrderIdAsync(clientOrderId)`
- `GetUserTradesAsync(...)`
- `GetRecentUserTradesAsync()`
- `PlaceStopOrderAsync(...)`
- `CancelStopOrderAsync(orderId)`
- `CancelStopOrdersAsync(...)`
- `GetStopOrdersAsync(...)`
- `GetStopOrderAsync(orderId)`

`client.SpotApi.HfTrading`:

- High-frequency account, order, margin order, open order, closed order, trade, cancel-after, and cancel-by-symbol methods.
- Use only when the user asks for KuCoin high-frequency or pro account flows.

`client.SpotApi.Margin`:

- Margin configuration, mark prices, symbols, risk limits, borrow, repay, borrow/repay history, interest history, lending assets/rates, subscribe/redeem lending orders, leverage multiplier, and collateral ratio methods.

`client.SpotApi.Earn`:

- Earn product, holding, purchase, redeem, and redeem-preview methods.
- Generate live purchase or redeem calls only when explicitly requested.

## Futures REST Endpoint Groups

`client.FuturesApi.ExchangeData`:

- `GetSymbolsAsync()`
- `GetContractAsync(symbol)`
- `GetTickerAsync(symbol)`
- `GetTickersAsync()`
- `GetAggregatedFullOrderBookAsync(symbol)`
- `GetAggregatedPartialOrderBookAsync(symbol, depth)`
- `GetInterestRatesAsync(symbol, ...)`
- `GetIndexListAsync(symbol, ...)`
- `GetCurrentMarkPriceAsync(symbol)`
- `GetPremiumIndexAsync(symbol, ...)`
- `GetCurrentFundingRateAsync(symbol)`
- `GetTradeHistoryAsync(symbol)`
- `GetServerTimeAsync()`
- `GetServiceStatusAsync()`
- `GetKlinesAsync(symbol, interval, startTime, endTime)`

`client.FuturesApi.Account`:

- `GetAccountOverviewAsync(asset)`
- `GetTransactionHistoryAsync(...)`
- `GetOpenOrderValueAsync(symbol)`
- `GetPositionAsync(symbol)`
- `GetPositionsAsync(asset)`
- `GetPositionHistoryAsync(...)`
- `ToggleAutoDepositMarginAsync(symbol, enabled)`
- `AddMarginAsync(symbol, quantity, clientId)`
- `RemoveMarginAsync(symbol, quantity)`
- `GetFundingHistoryAsync(symbol, ...)`
- `GetRiskLimitLevelAsync(symbol)`
- `SetRiskLimitLevelAsync(symbol, level)`
- `GetTradingFeeAsync(symbol)`
- `GetMarginModeAsync(symbol)`
- `SetMarginModeAsync(symbol, marginMode)`
- `SetMarginModesAsync(symbols, marginMode)`
- `GetCrossMarginLeverageAsync(symbol)`
- `SetCrossMarginLeverageAsync(symbol, leverage)`
- `SetPositionModeAsync(positionMode)`
- `GetPositionModeAsync()`

`client.FuturesApi.Trading`:

- `PlaceOrderAsync(...)`
- `PlaceMultipleOrdersAsync(...)`
- `PlaceTestOrderAsync(...)`
- `PlaceTpSlOrderAsync(...)`
- `CancelOrderAsync(orderId)`
- `CancelOrderByClientOrderIdAsync(clientOrderId)`
- `CancelOrdersAsync(orderIds)`
- `CancelAllOrdersAsync(symbol)`
- `CancelAllStopOrdersAsync(symbol)`
- `GetOrdersAsync(...)`
- `GetOrderAsync(orderId)`
- `GetOrderByClientOrderIdAsync(clientOrderId)`
- `GetOpenOrdersAsync(symbol)`
- `GetRecentUserTradesAsync()`
- `GetUserTradesAsync(...)`
- `GetUntriggeredStopOrdersAsync(...)`
- `GetPositionHistoryAsync(...)`
- `SetDcpAsync(...)`
- `GetDcpAsync(...)`

Futures symbols are contract names such as `ETHUSDTM` or `XBTUSDTM`.

## Unified REST Endpoint Groups

`client.UnifiedApi.Account`:

- `GetAccountOverviewAsync()`
- `GetBalancesAsync()`
- `GetClassicBalancesAsync(...)`
- `GetSubAccountBalancesAsync(...)`
- `GetTransferQuotasAsync(...)`
- `TransferAsync(...)`
- `SetSubAccountTransferPermissionAsync(...)`
- `GetAccountModeAsync()`
- `SetAccountModeAsync(accountMode)`
- `GetFeeRateAsync(...)`
- `GetAccountLedgerAsync(...)`
- `GetInterestHistoryAsync(...)`
- `SetLeverageAsync(symbol, leverage)`
- `GetDepositAddressAsync(asset, network)`
- `GetApiKeyInfoAsync()`
- `GetWithdrawalQuotasAsync(...)`
- `WithdrawAsync(...)`
- `SetCrossMarginLeverageAsync(...)`
- `GetLeverageAsync(...)`
- `GetFundingFeeHistoryAsync(...)`

`client.UnifiedApi.ExchangeData`:

- `GetAnnouncementsAsync(...)`
- `GetSpotSymbolsAsync(symbol)`
- `GetFuturesSymbolsAsync(symbol)`
- `GetCrossMarginSymbolsAsync(symbol)`
- `GetIsolatedMarginSymbolsAsync(symbol)`
- `GetAssetAsync(asset, network)`
- `GetAssetsAsync(assets, network)`
- `GetTickersAsync(productType, symbol)`
- `GetOrderBookAsync(productType, symbol, limit)`
- `GetRecentTradesAsync(productType, symbol)`
- `GetKlinesAsync(productType, symbol, interval, startTime, endTime)`
- `GetFundingRateAsync(symbol)`
- `GetFundingHistoryAsync(symbol, startTime, endTime)`
- `GetCrossMarginConfigAsync()`
- `GetCollateralRatioAsync()`
- `GetFuturesOpenInterestAsync(symbols)`
- `GetFuturesOpenInterestHistoryAsync(...)`
- `GetServiceStatusAsync(productType)`
- `GetKYCRegionsAsync()`

`client.UnifiedApi.Trading`:

- `PlaceOrderAsync(...)`
- `CancelOrderAsync(...)`
- `CancelOrdersAsync(...)`
- `CancelSymbolOrdersAsync(...)`
- `GetOrderAsync(...)`
- `GetOpenOrdersAsync(...)`
- `GetOrderHistoryAsync(...)`
- `GetUserTradesAsync(...)`
- `SetDcpAsync(tradeType, timeout, symbols)`
- `GetDcpAsync(tradeType)`
- `GetPositionsAsync(accountMode, symbol, page, pageSize)`
- `GetPositionHistoryAsync(...)`
- `GetPositionTiersAsync(...)`

Unified APIs are native KuCoin account APIs. They do not expose `SharedClient`.

## Socket Subscriptions

`socket.SpotApi` public streams:

- `SubscribeToTickerUpdatesAsync(symbols, handler)`
- `SubscribeToAllTickerUpdatesAsync(handler)`
- `SubscribeToSnapshotUpdatesAsync(symbolOrMarkets, handler)`
- `SubscribeToAggregatedOrderBookUpdatesAsync(symbols, handler)`
- `SubscribeToTradeUpdatesAsync(symbols, handler)`
- `SubscribeToKlineUpdatesAsync(symbols, interval, handler)`
- `SubscribeToBookTickerUpdatesAsync(symbols, handler)`
- `SubscribeToOrderBookUpdatesAsync(symbols, limit, handler)`
- `SubscribeToIndexPriceUpdatesAsync(symbols, handler)`
- `SubscribeToMarkPriceUpdatesAsync(symbols, handler)`
- `SubscribeToCallAuctionOrderBookUpdatesAsync(symbols, handler)`
- `SubscribeToCallAuctionInfoUpdatesAsync(symbols, handler)`

`socket.SpotApi` private streams:

- `SubscribeToOrderUpdatesAsync(onNewOrder, onOrderData, onTradeData)`
- `SubscribeToBalanceUpdatesAsync(handler)`
- `SubscribeToStopOrderUpdatesAsync(handler)`
- `SubscribeToMarginPositionUpdatesAsync(...)`
- `SubscribeToMarginOrderUpdatesAsync(symbol, ...)`
- `SubscribeToIsolatedMarginPositionUpdatesAsync(symbol, handler)`

`socket.FuturesApi` streams:

- `SubscribeToTradeUpdatesAsync(symbols, handler)`
- `SubscribeToKlineUpdatesAsync(symbols, interval, handler)`
- `SubscribeToBookTickerUpdatesAsync(symbols, handler)`
- `SubscribeToOrderBookUpdatesAsync(symbols, handler)`
- `SubscribeToPartialOrderBookUpdatesAsync(symbols, limit, handler)`
- `SubscribeToSymbolUpdatesAsync(symbols, markIndexHandler, fundingRateHandler)`
- `SubscribeToFundingFeeSettlementUpdatesAsync(handler)`
- `SubscribeTo24HTickerUpdatesAsync(symbol, handler)`
- `SubscribeTo24HourSnapshotUpdatesAsync(symbols, handler)`
- `SubscribeToBalanceUpdatesAsync(...)`
- `SubscribeToPositionUpdatesAsync(...)`
- `SubscribeToMarginModeUpdatesAsync(handler)`
- `SubscribeToCrossMarginLeverageUpdatesAsync(handler)`
- `SubscribeToOrderUpdatesAsync(symbol, handler)`
- `SubscribeToStopOrderUpdatesAsync(handler)`

`socket.UnifiedApi` streams:

- `SubscribeToTickerUpdatesAsync(tradeType, symbols, handler)`
- `SubscribeToKlineUpdatesAsync(tradeType, symbol, interval, handler)`
- `SubscribeToOrderBookUpdatesAsync(tradeType, symbol, depth, handler)`
- `SubscribeToTradeUpdatesAsync(tradeType, symbol, handler)`
- `SubscribeToBalanceUpdatesAsync(tradeType, handler)`
- `SubscribeToOrderUpdatesAsync(tradeType, handler)`
- `SubscribeToUserTradeUpdatesAsync(tradeType, handler)`
- `SubscribeToLiteUserTradeUpdatesAsync(tradeType, handler)`
- `SubscribeToPositionUpdatesAsync(tradeType, handler)`
- `SubscribeToLeverageUpdatesAsync(tradeType, handler)`
- `SubscribeToLiquidationWarningUpdatesAsync(tradeType, handler)`

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
- `ISpotTriggerOrderRestClient`
- `IBookTickerRestClient`
- `ITransferRestClient`

Implemented futures REST shared interfaces:

- `IBalanceRestClient`
- `IFuturesTickerRestClient`
- `IFuturesSymbolRestClient`
- `IFuturesOrderRestClient`
- `IKlineRestClient`
- `IRecentTradeRestClient`
- `IOrderBookRestClient`
- `IOpenInterestRestClient`
- `IFundingRateRestClient`
- `IPositionHistoryRestClient`
- `IFeeRestClient`
- `IFuturesOrderClientIdRestClient`
- `IFuturesTpSlRestClient`
- `IBookTickerRestClient`
- `ILeverageRestClient`

Socket shared clients:

- `socket.SpotApi.SharedClient`
- `socket.FuturesApi.SharedClient`

Implemented spot socket shared interfaces:

- `ITickerSocketClient`
- `ITradeSocketClient`
- `IBookTickerSocketClient`
- `IKlineSocketClient`
- `IOrderBookSocketClient`
- `IBalanceSocketClient`
- `ISpotOrderSocketClient`

Implemented futures socket shared interfaces:

- `ITickerSocketClient`
- `ITradeSocketClient`
- `IBookTickerSocketClient`
- `IKlineSocketClient`
- `IOrderBookSocketClient`
- `IBalanceSocketClient`
- `IFuturesOrderSocketClient`
- `IPositionSocketClient`

Call `SharedClient.Discover()` before relying on optional shared features.

## Symbols

- Spot symbols use names such as `BTC-USDT` and `ETH-USDT`.
- Futures symbols use contract names such as `ETHUSDTM` and `XBTUSDTM`.
- `KucoinExchange.FormatSymbol("BTC", "USDT", TradingMode.Spot)` returns `BTC-USDT`.
- `KucoinExchange.FormatSymbol("BTC", "USDT", TradingMode.PerpetualLinear)` returns `XBTUSDTM`.
- `KucoinExchange.FormatSymbol("ETH", "USDT", TradingMode.PerpetualLinear)` returns `ETHUSDTM`.
- Unified methods use KuCoin symbol strings plus explicit account/trade type parameters where required.
- SharedApis uses `SharedSymbol`, for example `new SharedSymbol(TradingMode.Spot, "BTC", "USDT")`.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Websocket subscriptions: `WebSocketResult<UpdateSubscription>`
- Shared non-I/O helpers: `ExchangeCallResult<T>`

Always check `Success` before using `Data`. Bulk order methods can return nested per-item `CallResult<T>` values.
