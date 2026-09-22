# Weex.Net API Surfaces

Use this file to select Weex client roots, endpoint groups, streams, SharedApis, and result types.

## Package And Clients

- Package: `Weex.Net`
- REST: `WeexRestClient`, `IWeexRestClient`
- Socket: `WeexSocketClient`, `IWeexSocketClient`
- Credentials: `WeexCredentials(key, secret, passphrase)`
- DI: `services.AddWeex(...)`
- Factories: `IWeexOrderBookFactory`, `IWeexTrackerFactory`
- Multi-user clients: `IWeexUserClientProvider`

## Roots

- `client.SpotApi`
- `client.FuturesApi`
- `socket.SpotApi`
- `socket.FuturesApi`

Each API exposes `SharedApi`.

## Spot REST

`SpotApi.ExchangeData`:

- `GetServerTimeAsync()`
- `GetAssetsAsync()`
- `GetExchangeInfoAsync(symbols, symbolStatus)`
- `GetPricesAsync(symbols)`
- `GetTickersAsync(symbols)`
- `GetRecentTradesAsync(symbol, limit)`
- `GetKlinesAsync(symbol, interval)`
- `GetOrderBookAsync(symbol, limit)`
- `GetBookTickersAsync(symbols)`

`SpotApi.Account`:

- `GetTradingSymbolsAsync()`
- `GetAccountInfoAsync()`
- `GetAccountBillsAsync(...)`
- `GetFundingBillsAsync(...)`
- `GetTransferHistoryAsync(...)`

`SpotApi.Trading`:

- `PlaceOrderAsync(symbol, side, orderType, quantity, price, timeInForce, clientOrderId)`
- `CancelOrderAsync(orderId, clientOrderId)`
- `CancelAllSymbolOrdersAsync(symbol)`
- `CancelOrdersAsync(orderIds, clientOrderIds)`
- `GetOrderAsync(orderId, clientOrderId)`
- `GetOpenOrdersAsync(symbol)`
- `GetOrderHistoryAsync(symbol, startTime, endTime, page, limit)`
- `GetUserTradesAsync(symbol, orderId, startTime, endTime, limit)`

## Futures REST

`FuturesApi.ExchangeData`:

- exchange info, order book, tickers, book tickers, trades
- regular/index/mark klines and kline history
- price, open interest, funding rate/history, trading symbols

`FuturesApi.Account`:

- balances, fees, account/symbol configuration, bills
- `SetMarginModeAsync(...)`
- `SetLeverageAsync(...)`
- `AdjustIsolatedMarginAsync(...)`
- `SetAutoAppendMarginAsync(...)`

`FuturesApi.Trading`:

- positions and position lookup
- regular order placement/cancellation/history/trades
- `ClosePositionsAsync(symbol)`
- conditional order placement/cancellation/history
- TP/SL placement and editing

Regular `PlaceOrderAsync` uses `OrderType`. `PlaceConditionalOrderAsync` uses `FuturesOrderType`.

## Socket Streams

Spot public:

- ticker, book ticker, order book, kline, trade

Spot private:

- account, order, user trade

Futures public:

- ticker, kline, order book, trade

Futures private:

- account, position, order, user trade

All subscription methods return `WebSocketResult<UpdateSubscription>`.

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.SpotApi.SharedApi`
- `restClient.FuturesApi.SharedApi`
- `socketClient.SpotApi.SharedApi`
- `socketClient.FuturesApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `IWeexSharedApiClient`. It exposes:

- `SpotRest` as `IWeexRestClientSpotSharedApi`
- `FuturesRest` as `IWeexRestClientFuturesSharedApi`
- `SpotSocket` as `IWeexSocketClientSpotSharedApi`
- `FuturesSocket` as `IWeexSocketClientFuturesSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `IWeexSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.

## Environment And Symbols

- Built-in environment: `WeexEnvironment.Live`
- Custom environment: `WeexEnvironment.CreateCustom(...)`
- No built-in testnet is exposed by the current source.
- Spot and futures commonly use compact symbols such as `ETHUSDT`.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Socket: `WebSocketResult<UpdateSubscription>`
- Shared helpers: `ExchangeCallResult<T>`

Check `Success` before using `Data`.
