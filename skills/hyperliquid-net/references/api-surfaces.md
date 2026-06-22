# HyperLiquid.Net API Surfaces

Use this file when choosing native HyperLiquid client roots, endpoint groups, shared interfaces, result types, or symbol formats.

## Package And Client Types

- NuGet package: `HyperLiquid.Net`
- REST client: `HyperLiquidRestClient`
- Socket client: `HyperLiquidSocketClient`
- REST interface: `IHyperLiquidRestClient`
- Socket interface: `IHyperLiquidSocketClient`
- Credentials: `HyperLiquidCredentials`
- DI extension: `services.AddHyperLiquid(...)`
- Tracker factory: `IHyperLiquidTrackerFactory`

## Client Roots

REST roots:

- `client.SpotApi`
- `client.FuturesApi`

Socket roots:

- `socket.SpotApi`
- `socket.FuturesApi`

Do not use exchange roots such as `UsdFuturesApi`, `CoinFuturesApi`, `UnifiedApi`, `V5Api`, or `ExchangeApi`.

## Shared Base REST Groups

The spot and futures APIs share many base account, exchange-data, and trading methods. Use the appropriate root for symbol format and account type.

`client.SpotApi.ExchangeData` and `client.FuturesApi.ExchangeData` shared methods:

- `GetPricesAsync(dex)`
- `GetOrderBookAsync(symbol, numberSignificantFigures, mantissa)`
- `GetKlinesAsync(symbol, interval, startTime, endTime)`

`client.SpotApi.Account` and `client.FuturesApi.Account` shared methods:

- `GetFeeInfoAsync(address)`
- `GetAccountLedgerAsync(startTime, endTime, address)`
- `GetRateLimitsAsync(address)`
- `GetApprovedBuilderFeeAsync(builderAddress, address)`
- `TransferUsdAsync(destinationAddress, quantity)`
- `WithdrawAsync(destinationAddress, quantity)`
- `TransferInternalAsync(direction, quantity, subAccount)`
- `SendAssetAsync(destination, sourceDex, destinationDex, tokenName, tokenId, quantity, fromSubAccount)`
- `DepositIntoStakingAsync(wei)`
- `WithdrawFromStakingAsync(wei)`
- `DelegateOrUndelegateStakeFromValidatorAsync(direction, validator, wei)`
- `DepositOrWithdrawFromVaultAsync(direction, vaultAddress, usd, expireAfter)`
- `ApproveBuilderFeeAsync()`
- `ApproveBuilderFeeAsync(builderAddress, maxFeePercentage)`
- `GetSubAccountsAsync(address)`
- `GetUserRoleAsync(address)`
- `GetExtraAgentsAsync(address)`
- `GetStakingDelegationsAsync(address)`
- `GetStakingSummaryAsync(address)`
- `GetStakingHistoryAsync(address)`
- `GetStakingRewardsAsync(address)`
- `GetUserAbstractionStateAsync(address)`

`client.SpotApi.Trading` and `client.FuturesApi.Trading` shared methods:

- `GetOpenOrdersAsync(address, dex)`
- `GetOpenOrdersExtendedAsync(address, dex)`
- `GetUserTradesAsync(address)`
- `GetUserTradesByTimeAsync(address, startTime, endTime, aggregateByTime)`
- `GetOrderAsync(orderId, clientOrderId, address)`
- `GetOrderHistoryAsync(address)`
- `PlaceOrderAsync(symbol, side, orderType, quantity, price, timeInForce, reduceOnly, clientOrderId, triggerPrice, tpSlType, tpSlGrouping, vaultAddress, expireAfter)`
- `PlaceMultipleOrdersAsync(orders, tpSlGrouping, vaultAddress, expireAfter)`
- `CancelOrderAsync(symbol, orderId, vaultAddress, expireAfter)`
- `CancelOrdersAsync(requests, vaultAddress, expireAfter)`
- `CancelOrderByClientOrderIdAsync(symbol, clientOrderId, vaultAddress, expireAfter)`
- `CancelOrdersByClientOrderIdAsync(requests, vaultAddress, expireAfter)`
- `CancelAfterAsync(timeout, vaultAddress, expireAfter)`
- `EditOrderAsync(...)`
- `EditOrdersAsync(...)`
- `PlaceTwapOrderAsync(...)`
- `CancelTwapOrderAsync(symbol, twapId, vaultAddress, expireAfter)`

Batch place/edit/cancel methods return nested per-item `CallResult` values that need inspection after the outer result succeeds.

## Spot REST Endpoint Groups

`client.SpotApi.ExchangeData`:

- `GetExchangeInfoAsync()`
- `GetExchangeInfoAndTickersAsync()`
- `GetAssetInfoAsync(assetId)`
- `GetQuestionsAndOutcomesInfoAsync()`
- `GetSettledOutcomeAsync(outcomeId)`
- Shared methods from the base exchange-data group

`client.SpotApi.Account`:

- `GetBalancesAsync(address)`
- `TransferSpotAsync(destinationAddress, asset, quantity)`
- Shared methods from the base account group

`client.SpotApi.Trading`:

- Shared methods from the base trading group

Spot symbols use `Base/Quote`, for example `HYPE/USDC`.

## Futures REST Endpoint Groups

`client.FuturesApi.ExchangeData`:

- `GetPerpDexesAsync()`
- `GetExchangeInfoAsync(dex)`
- `GetExchangeInfoAllDexesAsync()`
- `GetExchangeInfoAndTickersAsync()`
- `GetFundingRateHistoryAsync(symbol, startTime, endTime)`
- `GetSymbolsAtMaxOpenInterestAsync(dex)`
- `GetPerpDexMarketLimitsAsync(dex)`
- `GetPerpDexMarketStatusAsync(dex)`
- Shared methods from the base exchange-data group

`client.FuturesApi.Account`:

- `GetAccountInfoAsync(address, dex)`
- `GetFundingHistoryAsync(startTime, endTime, address)`
- `GetUserSymbolAsync(symbol, address)`
- `GetHip3DexAbstractionAsync(user)`
- `ToggleHip3DexAbstractionAsync(enabled, address)`
- Shared methods from the base account group

`client.FuturesApi.Trading`:

- `SetLeverageAsync(symbol, leverage, marginType, vaultAddress, expireAfter)`
- `UpdateIsolatedMarginAsync(symbol, updateValue, vaultAddress, expireAfter)`
- Shared methods from the base trading group

Futures symbols use base asset only, for example `ETH`. HIP-3 DEX-aware methods take optional `dex` parameters where exposed.

## Socket Queries And Subscriptions

Socket exchange-data, account, and trading request methods generally mirror REST methods and return `QueryResult<T>` or `QueryResult`.

`socket.SpotApi.ExchangeData`:

- Query methods: `GetExchangeInfoAsync`, `GetExchangeInfoAndTickersAsync`, `GetAssetInfoAsync`, `GetQuestionsAndOutcomesInfoAsync`, `GetSettledOutcomeAsync`
- Shared query methods: `GetPricesAsync`, `GetOrderBookAsync`, `GetKlinesAsync`
- Streams: `SubscribeToSymbolUpdatesAsync`, `SubscribeToOutcomeInfoUpdatesAsync`
- Shared streams: `SubscribeToPriceUpdatesAsync`, `SubscribeToKlineUpdatesAsync`, `SubscribeToOrderBookUpdatesAsync`, `SubscribeToTradeUpdatesAsync`, `SubscribeToBookTickerUpdatesAsync`

`socket.FuturesApi.ExchangeData`:

- Query methods: `GetPerpDexesAsync`, `GetExchangeInfoAsync`, `GetExchangeInfoAllDexesAsync`, `GetExchangeInfoAndTickersAsync`, `GetFundingRateHistoryAsync`, `GetSymbolsAtMaxOpenInterestAsync`, `GetPerpDexMarketLimitsAsync`, `GetPerpDexMarketStatusAsync`
- Shared query methods: `GetPricesAsync`, `GetOrderBookAsync`, `GetKlinesAsync`
- Streams: `SubscribeToSymbolUpdatesAsync`, `SubscribeToPriceUpdatesAsync(dex, handler)`
- Shared streams: `SubscribeToKlineUpdatesAsync`, `SubscribeToOrderBookUpdatesAsync`, `SubscribeToTradeUpdatesAsync`, `SubscribeToBookTickerUpdatesAsync`

`socket.SpotApi.Account`:

- Query methods: `GetBalancesAsync`, `TransferSpotAsync`, plus base account query methods
- Streams: `SubscribeToBalanceUpdatesAsync`, plus base user event, ledger, user, and web-data streams

`socket.FuturesApi.Account`:

- Query methods: `GetAccountInfoAsync`, `GetFundingHistoryAsync`, `GetUserSymbolAsync`, `GetHip3DexAbstractionAsync`, `ToggleHip3DexAbstractionAsync`, plus base account query methods
- Streams: `SubscribeToUserSymbolUpdatesAsync`, `SubscribeToUserFundingUpdatesAsync`, plus base user event, ledger, user, and web-data streams

`socket.SpotApi.Trading` and `socket.FuturesApi.Trading`:

- Query methods mirror base trading methods and return `QueryResult<T>` or `QueryResult`
- Streams: `SubscribeToOrderUpdatesAsync`, `SubscribeToOpenOrderUpdatesAsync`, `SubscribeToUserTradeUpdatesAsync`, `SubscribeToTwapTradeUpdatesAsync`, `SubscribeToTwapOrderUpdatesAsync`
- Futures-only streams: `SubscribeToBalanceAndPositionUpdatesAsync`, `SubscribeToBalanceAndPositionUpdatesAllDexesAsync`

## SharedApis Interfaces

REST shared clients:

- `client.SpotApi.SharedClient`
- `client.FuturesApi.SharedClient`

Implemented spot REST shared interfaces:

- `IBalanceRestClient`
- `IKlineRestClient`
- `IOrderBookRestClient`
- `ISpotTickerRestClient`
- `ISpotSymbolRestClient`
- `ISpotOrderRestClient`
- `IAssetsRestClient`
- `IFeeRestClient`
- `IWithdrawRestClient`
- `ISpotOrderClientIdRestClient`
- `IBookTickerRestClient`
- `ITransferRestClient`

Implemented futures REST shared interfaces:

- `IBalanceRestClient`
- `IKlineRestClient`
- `IOrderBookRestClient`
- `IFuturesTickerRestClient`
- `IFuturesSymbolRestClient`
- `IFeeRestClient`
- `IFundingRateRestClient`
- `ILeverageRestClient`
- `IOpenInterestRestClient`
- `IFuturesOrderRestClient`
- `IFuturesOrderClientIdRestClient`
- `IFuturesTpSlRestClient`
- `IBookTickerRestClient`

Socket shared clients:

- `socket.SpotApi.SharedClient`
- `socket.FuturesApi.SharedClient`

Implemented spot socket shared interfaces:

- `ITickerSocketClient`
- `ITradeSocketClient`
- `IKlineSocketClient`
- `IOrderBookSocketClient`
- `ISpotOrderSocketClient`
- `IUserTradeSocketClient`
- `IBalanceSocketClient`
- `IBookTickerSocketClient`

Implemented futures socket shared interfaces:

- `ITickerSocketClient`
- `ITradeSocketClient`
- `IKlineSocketClient`
- `IOrderBookSocketClient`
- `IUserTradeSocketClient`
- `IFuturesOrderSocketClient`
- `IBalanceSocketClient`
- `IPositionSocketClient`
- `IBookTickerSocketClient`

Call `SharedClient.Discover()` before relying on optional shared features.

## Symbols

- Native spot symbols use `Base/Quote`: `HYPE/USDC`.
- Native futures symbols use base asset only: `ETH`.
- `HyperLiquidExchange.FormatSymbol("HYPE", "USDC", TradingMode.Spot)` returns `HYPE/USDC`.
- `HyperLiquidExchange.FormatSymbol("ETH", "USDC", TradingMode.PerpetualLinear)` returns `ETH`.
- SharedApis uses `SharedSymbol`, for example `new SharedSymbol(TradingMode.PerpetualLinear, "ETH", "USDC")`.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Websocket request/query methods: `QueryResult<T>` or `QueryResult`
- Websocket subscriptions: `WebSocketResult<UpdateSubscription>`
- Shared non-I/O helpers: `ExchangeCallResult<T>`

Always check `Success` before using `Data`. Batch methods can return nested per-item result objects.
