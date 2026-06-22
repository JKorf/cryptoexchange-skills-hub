# Toobit.Net API Surfaces

Use this file when choosing native Toobit client roots, endpoint groups, shared interfaces, result types, user streams, or symbol formats.

## Package And Client Types

- NuGet package: `Toobit.Net`
- REST client: `ToobitRestClient`
- Socket client: `ToobitSocketClient`
- REST interface: `IToobitRestClient`
- Socket interface: `IToobitSocketClient`
- Credentials: `ToobitCredentials`
- DI extension: `services.AddToobit(...)`
- Local order book factory: `IToobitOrderBookFactory`
- Tracker factory: `IToobitTrackerFactory`
- User client provider: `IToobitUserClientProvider`

## Client Roots

REST roots:

- `client.SpotApi`
- `client.UsdtFuturesApi`

Socket roots:

- `socket.SpotApi`
- `socket.UsdtFuturesApi`

Do not use `UsdFuturesApi`, `CoinFuturesApi`, `UnifiedApi`, or `ExchangeApi`.

## Spot REST Endpoint Groups

`client.SpotApi.ExchangeData`:

- `GetServerTimeAsync()`
- `GetExchangeInfoAsync()`
- `GetOrderBookAsync(symbol, limit)`
- `GetRecentTradesAsync(symbol, limit)`
- `GetKlinesAsync(symbol, interval, startTime, endTime, limit)`
- `GetTickersAsync(symbol)`
- `GetPricesAsync(symbol)`
- `GetBookTickersAsync(symbol)`

`client.SpotApi.Account`:

- `GetBalancesAsync()`
- `WithdrawAsync(asset, address, quantity, network, tag, vaspCode, targetPersonFirstName, targetPersonLastName, clientOrderId)`
- `GetWithdrawalsAsync(asset, fromId, withdrawOrderId, startTime, endTime, limit)`
- `GetDepositAddressAsync(asset, network)`
- `GetDepositsAsync(asset, fromId, startTime, endTime, limit)`
- `TransferAsync(fromUid, toUid, fromAccountType, toAccountType, asset, quantity)`
- `GetTransactionHistoryAsync(accountType, asset, flowType, fromId, startTime, endTime, limit)`
- `GetSubAccountsAsync()`
- `StartUserStreamAsync()`
- `KeepAliveUserStreamAsync(listenKey)`
- `StopUserStreamAsync(listenKey)`

`client.SpotApi.Trading`:

- `PlaceTestOrderAsync(symbol, orderSide, orderType, quantity, timeInForce, price, clientOrderId)`
- `PlaceOrderAsync(symbol, orderSide, orderType, quantity, timeInForce, price, clientOrderId)`
- `PlaceMultipleOrdersAsync(orders)`
- `CancelOrderAsync(orderId, clientOrderId)`
- `CancelAllOrdersAsync(symbol, side)`
- `CancelMultipleOrdersAsync(orderIds)`
- `GetOrderAsync(orderId, clientOrderId)`
- `GetOpenOrdersAsync(orderId, symbol, limit)`
- `GetOrdersAsync(orderId, symbol, startTime, endTime, limit)`
- `GetUserTradesAsync(symbol, fromId, toId, startTime, endTime, limit)`

## USDT Futures REST Endpoint Groups

`client.UsdtFuturesApi.ExchangeData`:

- `GetServerTimeAsync()`
- `GetExchangeInfoAsync()`
- `GetOrderBookAsync(symbol, limit)`
- `GetRecentTradesAsync(symbol, limit)`
- `GetKlinesAsync(symbol, interval, startTime, endTime, limit)`
- `GetMarkPriceKlinesAsync(symbol, interval, startTime, endTime, limit)`
- `GetIndexPriceKlinesAsync(symbol, interval, startTime, endTime, limit)`
- `GetTickersAsync(symbol, interval)`
- `GetPricesAsync(symbol)`
- `GetBookTickersAsync(symbol)`
- `GetIndexPricesAsync(symbol)`
- `GetMarkPriceAsync(symbol)`
- `GetFundingRateAsync(symbol)`
- `GetFundingRateHistoryAsync(symbol, fromId, endId, limit)`

`client.UsdtFuturesApi.Account`:

- `SetMarginTypeAsync(symbol, marginType)`
- `SetLeverageAsync(symbol, leverage)`
- `GetLeverageInfoAsync(symbol)`
- `GetBalancesAsync()`
- `SetPositionMarginAsync(symbol, positionSide, quantity)`
- `GetTransactionHistoryAsync(asset, flowType, fromId, endId, startTime, endTime, limit)`
- `GetFeesAsync(symbol)`
- `StartUserStreamAsync()`
- `KeepAliveUserStreamAsync(listenKey)`
- `StopUserStreamAsync(listenKey)`

`client.UsdtFuturesApi.Trading`:

- `PlaceOrderAsync(symbol, orderSide, orderType, quantity, price, priceType, stopPrice, timeInForce, clientOrderId, takeProfit, takeProfitTriggerType, takeProfitLimitPrice, takeProfitOrderType, stopLoss, stopLossTriggerType, stopLossLimitPrice, stopLossOrderType)`
- `PlaceMultipleOrdersAsync(orders)`
- `GetOrderAsync(orderId, clientOrderId, orderType)`
- `CancelOrderAsync(orderId, clientOrderId, orderType)`
- `CancelAllOrdersAsync(symbol, side)`
- `CancelMultipleOrdersAsync(orderIds)`
- `GetOpenOrdersAsync(symbol, orderId, orderType, limit)`
- `GetPositionsAsync(symbol, positionSide)`
- `SetTradingStopAsync(symbol, positionSide, takeProfitPrice, stopLossPrice, takeProfitTriggerType, StopLossTriggerType)`
- `GetOrderHistoryAsync(symbol, toId, orderType, startTime, endTime, limit)`
- `GetUserTradesAsync(symbol, fromId, toId, startTime, endTime, limit)`

## Socket Streams

`socket.SpotApi` public streams:

- `SubscribeToTradeUpdatesAsync(symbols, handler)`
- `SubscribeToTickerUpdatesAsync(symbols, handler)`
- `SubscribeToKlineUpdatesAsync(symbols, interval, handler)`
- `SubscribeToPartialOrderBookUpdatesAsync(symbols, handler)`
- `SubscribeToOrderBookUpdatesAsync(symbols, handler)`

`socket.SpotApi` private stream:

- `SubscribeToUserDataUpdatesAsync(onAccountMessage, onOrderMessage, onUserTradeMessage)`
- `SubscribeToUserDataUpdatesAsync(listenKey, onAccountMessage, onOrderMessage, onUserTradeMessage)`

`socket.UsdtFuturesApi` public streams:

- `SubscribeToTradeUpdatesAsync(symbols, handler)`
- `SubscribeToKlineUpdatesAsync(symbols, interval, handler)`
- `SubscribeToPartialOrderBookUpdatesAsync(symbols, handler)`
- `SubscribeToOrderBookUpdatesAsync(symbols, handler)`
- `SubscribeToTickerUpdatesAsync(symbols, handler)`
- `SubscribeToIndexPriceUpdatesAsync(symbols, handler)`

`socket.UsdtFuturesApi` private stream:

- `SubscribeToUserDataUpdatesAsync(onAccountMessage, onOrderMessage, onPositionMessage, onUserTradeMessage)`
- `SubscribeToUserDataUpdatesAsync(listenKey, onAccountMessage, onOrderMessage, onPositionMessage, onUserTradeMessage)`

User stream overloads without a listen key require API credentials and manage listen key acquisition/renewal. Listen-key overloads require `StartUserStreamAsync`, keepalive, and stop calls.

## SharedApis Interfaces

REST shared clients:

- `client.SpotApi.SharedClient`
- `client.UsdtFuturesApi.SharedClient`

Spot REST shared interfaces:

- `IKlineRestClient`
- `ISpotSymbolRestClient`
- `ISpotTickerRestClient`
- `IBookTickerRestClient`
- `IRecentTradeRestClient`
- `IOrderBookRestClient`
- `IBalanceRestClient`
- `ISpotOrderRestClient`
- `ISpotOrderClientIdRestClient`
- `IAssetsRestClient`
- `IDepositRestClient`
- `IWithdrawalRestClient`
- `IWithdrawRestClient`
- `ITransferRestClient`

USDT futures REST shared interfaces:

- `IKlineRestClient`
- `IMarkPriceKlineRestClient`
- `IIndexPriceKlineRestClient`
- `IFuturesSymbolRestClient`
- `IFuturesTickerRestClient`
- `IBookTickerRestClient`
- `IRecentTradeRestClient`
- `IFuturesOrderRestClient`
- `IFuturesOrderClientIdRestClient`
- `ILeverageRestClient`
- `IOrderBookRestClient`
- `IFundingRateRestClient`
- `IBalanceRestClient`
- `IFeeRestClient`
- `IFuturesTriggerOrderRestClient`

Socket shared clients:

- `socket.SpotApi.SharedClient`
- `socket.UsdtFuturesApi.SharedClient`

Use `SharedClient.Discover()` before relying on optional shared features.

## Symbols

- Spot symbols use compact names such as `BTCUSDT`.
- USDT futures contract symbols use names such as `ETH-SWAP-USDT`.
- Futures index price endpoints can use index symbols such as `ETHUSDT`.
- SharedApis uses `SharedSymbol`, for example `new SharedSymbol(TradingMode.Spot, "BTC", "USDT")`.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Websocket subscriptions: `WebSocketResult<UpdateSubscription>`
- Batch order methods: `HttpResult<CallResult<ToobitOrder>[]>` or `HttpResult<CallResult<ToobitFuturesOrder>[]>`
- Shared non-I/O helpers: `ExchangeCallResult<T>`

Always check `Success` before using `Data`. Batch order methods can return nested per-item `CallResult<T>` values.
