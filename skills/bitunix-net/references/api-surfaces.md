# Bitunix.Net API Surfaces

Use this reference when choosing native client roots, endpoint groups, result types, socket operations, SharedApis capabilities, environments, or symbol formats.

## Package And Client Types

- NuGet package: `Bitunix.Net`
- REST client/interface: `BitunixRestClient` / `IBitunixRestClient`
- Socket client/interface: `BitunixSocketClient` / `IBitunixSocketClient`
- Credentials: `BitunixCredentials`
- Shared aggregate: `IBitunixSharedApiClient`
- DI extension: `services.AddBitunix(...)`
- Local order book factory: `IBitunixOrderBookFactory`
- Tracker factory: `IBitunixTrackerFactory`
- User client provider: `IBitunixUserClientProvider`

## Client Roots

REST:

- `client.SpotApi`
- `client.FuturesApi`

Sockets:

- `socket.SpotApi`
- `socket.FuturesApi`

Do not use `GeneralApi`, `UsdtFuturesApi`, `UsdFuturesApi`, `CoinFuturesApi`, or `UnifiedApi`.

## Spot REST

`client.SpotApi.ExchangeData`:

- `GetLastPriceAsync(symbol)`
- `GetOrderBookAsync(symbol)`
- `GetKlinesAsync(symbol, interval)`
- `GetKlineHistoryAsync(symbol, interval, endTime, limit)`
- `GetSymbolsAsync()`
- `GetExchangeRatesAsync()`
- `GetAssetsAsync()`

`client.SpotApi.Account`:

- `GetBalancesAsync()`
- `WithdrawAsync(asset, network, address, quantity, memo)`
- `CancelWithdrawalAsync(withdrawId)`
- `TransferAsync(type, asset, quantity)`
- `GetWithdrawalHistoryAsync(id, outBizId, asset, hash, type, remark, startTime, endTime, limit)`
- `GetDepositsAsync(id, asset, type, hash, startTime, endTime, limit)`
- `GetDepositAddressAsync(asset, network)`

`client.SpotApi.Trading`:

- `PlaceOrderAsync(symbol, orderSide, orderType, quantity, price)`
- `CancelOrderAsync(symbol, orderId)`
- `CancelOrdersAsync(orders)`
- `GetOrderHistoryAsync(symbol, side, type, status, startTime, endTime, page, pageSize)`
- `GetOpenOrdersAsync(symbol)`
- `GetOrderAsync(id)`
- `GetUserTradesAsync(symbol, orderId)`

Direct Spot placement has no `timeInForce`, `clientOrderId`, or quote-quantity parameter.

## Futures REST

`client.FuturesApi.ExchangeData`:

- `GetTickersAsync(symbols)`
- `GetOrderBookAsync(symbol, limit)`
- `GetFundingRatesAsync()`
- `GetFundingRateHistoryAsync(symbol, startTime, endTime, limit)`
- `GetFundingRateAsync(symbol)`
- `GetKlinesAsync(symbol, interval, type, startTime, endTime, limit)`
- `GetSymbolsAsync(symbols)`
- `GetPositionTiersAsync(symbol)`

Direct order-book limits are 1, 5, 15, or 50; null requests the full book.

`client.FuturesApi.Account`:

- `AdjustMarginAsync(symbol, asset, quantity, side, id)`
- `GetBalancesAsync(marginAsset)`
- `GetTradingSettingsAsync(symbols)`
- `GetPositionModeAsync()`
- `GetLeverageMarginModeAsync(symbol, marginAsset)`
- `SetPositionModeAsync(positionMode)`
- `SetMarginModeAsync(symbol, asset, mode)`
- `SetLeverageAsync(symbol, marginAsset, leverage)`

`client.FuturesApi.Trading`:

- `PlaceOrderAsync(symbol, side, orderType, quantity, price, tradeSide, positionId, timeInForce, clientOrderId, reduceOnly, ...TP/SL...)`
- `EditOrderAsync(orderId, clientOrderId, quantity, price, ...TP/SL...)`
- `CancelOrderAsync(orderId, clientOrderId)`
- `CancelOrdersAsync(orders)`
- `CancelAllOrdersAsync(symbol)`
- `CloseAllPositionsAsync(symbol)`
- `ClosePositionAsync(positionId)`
- `GetOrderHistoryAsync(...)`
- `GetOrderAsync(orderId, clientOrderId)`
- `GetOpenOrdersAsync(...)`
- `GetUserTradesAsync(...)`
- `GetPositionsAsync(...)`
- `GetPositionHistoryAsync(...)`
- `PlaceTpSlOrderAsync(...)`
- `PlacePositionTpSlOrderAsync(...)`
- `CancelTpSlOrderAsync(...)`
- `GetOpenTpSlOrdersAsync(...)`
- `GetTpSlOrderHistoryAsync(...)`

The TP/SL and history methods have long signatures. Inspect `IBitunixRestClientFuturesApiTrading.cs` instead of inferring argument order.

## Socket Operations

`socket.SpotApi` authenticated query operations:

- `PlaceOrderAsync(symbol, orderSide, orderType, quantity, price)`
- `CancelOrderAsync(symbol, orderId)`
- `CancelOrdersAsync(orders)`

These return `QueryResult<T>` or `QueryResult`; they are not subscriptions. The current Spot socket exposes no public market or user-data streams.

`socket.FuturesApi` public subscriptions:

- `SubscribeToTickerUpdatesAsync(symbols, handler)`
- `SubscribeToAggregatedTickerUpdatesAsync(symbols, handler)`
- `SubscribeToOrderBookUpdatesAsync(symbols, limit, handler)`
- `SubscribeToKlineUpdatesAsync(symbols, interval, priceType, handler)`
- `SubscribeToMarkPriceUpdatesAsync(symbols, handler)`
- `SubscribeToTradeUpdatesAsync(symbols, handler)`

Each public Futures subscription has single-symbol and multi-symbol overloads. Socket order-book limits are 1, 5, or 15 for snapshots; null requests incremental updates.

Private Futures subscription:

- `SubscribeToUserUpdatesAsync(onBalanceMessage, onOrderMessage, onPositionMessage, onTpSlMessage)`

Private authentication uses socket client credentials directly. There is no listen-key endpoint or lifecycle.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Futures subscriptions: `WebSocketResult<UpdateSubscription>`
- Spot socket queries: `QueryResult<T>` or `QueryResult`
- Shared API methods: capability-specific CryptoExchange.Net result types

Check `Success` before using `Data`. Only successful Futures/shared subscriptions produce an `UpdateSubscription` for `UnsubscribeAsync`.

## Shared API V2

Strict surfaces:

- `restClient.SpotApi.SharedApi`
- `restClient.FuturesApi.SharedApi`
- `socketClient.SpotApi.SharedApi`
- `socketClient.FuturesApi.SharedApi`

The `IBitunixSharedApiClient` aggregate exposes `SpotRest`, `FuturesRest`, `SpotSocket`, and `FuturesSocket`.

Spot REST capabilities:

- Assets, balances, deposit history/addresses, klines, order book, Spot symbols, withdrawal history/placement
- Spot order placement, lookup, open/closed orders, trades, and cancellation
- No `IGetTickerRest`; native Spot only exposes last price

Futures REST capabilities:

- Balances, funding info/history, mark prices, klines and mark-price klines
- Leverage get/set and tiers, book ticker/order book, position mode get/set, Futures symbols, one/all tickers
- Futures order place/get/edit/cancel, client-order-ID variants, open/closed orders, trades and user-trade history, cancel-all
- Positions/history and Futures TP/SL set/cancel

Spot socket capabilities:

- `IPlaceSpotOrderSocket`
- `ICancelSpotOrderSocket`

Futures socket capabilities:

- Trades, ticker, klines, order book, funding info, mark price, index price
- Balances, Futures orders, and positions

Depend on the narrowest strict capability needed. Do not assume a capability exists because another exchange implements it.

## Symbols And Enums

- Spot and Futures native symbols: `ETHUSDT`
- Shared Spot: `new SharedSymbol(TradingMode.Spot, "ETH", "USDT")`
- Shared Futures: `TradingMode.PerpetualLinear` or `TradingMode.PerpetualInverse`
- Spot/Futures sides: `OrderSide.Buy`, `OrderSide.Sell`
- Order types: `OrderType.Limit`, `OrderType.Market`
- Hedge intent: `TradeSide.Open`, `TradeSide.Close`
- Position mode: `PositionMode.HedgeMode`, `PositionMode.OneWay`
- Margin mode: `MarginMode.Isolated`, `MarginMode.Cross`
- REST Futures klines: `FuturesKlineInterval`
- Socket Futures klines: `FuturesSocketKlineInterval`

## DI And Environments

`services.AddBitunix(...)` registers REST/socket clients, the SharedApi aggregate and strict interfaces, order-book/tracker factories, and user client provider.

Only `BitunixEnvironment.Live` is built in. `BitunixEnvironment.CreateCustom(...)` requires name plus four endpoint addresses: Spot REST, Spot socket, Futures REST, and Futures socket.
