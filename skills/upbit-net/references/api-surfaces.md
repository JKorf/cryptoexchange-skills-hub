# Upbit.Net API Surfaces

Use this file when choosing Upbit client roots, public market-data methods, regional environments, shared interfaces, result types, or websocket streams.

## Package And Client Types

- NuGet package: `JKorf.Upbit.Net`
- REST client: `UpbitRestClient`
- Socket client: `UpbitSocketClient`
- REST interface: `IUpbitRestClient`
- Socket interface: `IUpbitSocketClient`
- DI extension: `services.AddUpbit(...)`
- Local order book factory: `IUpbitOrderBookFactory`
- Tracker factory: `IUpbitTrackerFactory`

There is no credential type because the current library surface is public quotation data only.

## Client Roots

- REST: `client.SpotApi`
- Socket: `socket.SpotApi`

Do not use `SpotApi.Account`, `SpotApi.Trading`, `FuturesApi`, `UnifiedApi`, or `ExchangeApi`.

## REST Exchange Data

`client.SpotApi.ExchangeData`:

- `GetSymbolsAsync(includeNotifications)`
- `GetTradeHistoryAsync(symbol, endTime, limit, cursor)`
- `GetTickerAsync(symbol)`
- `GetTickersAsync(symbols)`
- `GetTickersByQuoteAssetsAsync(quoteAssets)`
- `GetOrderBookAsync(symbol, levels, aggregation)`
- `GetOrderBooksAsync(symbols, levels, aggregation)`
- `GetKlinesAsync(symbol, interval, endTime, limit)`
- `GetSymbolConfigAsync(symbols)`

`includeNotifications: true` maps to Upbit's detailed market information option.

## Socket Streams

`socket.SpotApi`:

- `SubscribeToTradeUpdatesAsync(symbols, handler)`
- `SubscribeToTickerUpdatesAsync(symbols, handler)`
- `SubscribeToOrderBookUpdatesAsync(symbols, levels, handler, aggregation)`
- `SubscribeToKlineUpdatesAsync(symbols, interval, handler)`

Order-book subscription levels are explicit. Common supported shared levels are `1`, `5`, `15`, and `30`.

## Regional Environments

- `UpbitEnvironment.Live`: South Korea
- `UpbitEnvironment.Singapore`: Singapore
- `UpbitEnvironment.Indonesia`: Indonesia
- `UpbitEnvironment.Thailand`: Thailand
- `UpbitEnvironment.CreateCustom(name, restAddress, socketAddress)`: custom endpoint environment

Environment names are `live`, `live-singapore`, `live-indonesia`, and `live-thailand`.

## SharedApis Interfaces

REST shared client:

- `client.SpotApi.SharedClient`

Implemented REST interfaces:

- `IKlineRestClient`
- `IOrderBookRestClient`
- `IRecentTradeRestClient`
- `ISpotSymbolRestClient`
- `ISpotTickerRestClient`
- `ITradeHistoryRestClient`
- `IBookTickerRestClient`

Socket shared client:

- `socket.SpotApi.SharedClient`

Implemented socket interfaces:

- `ITickerSocketClient`
- `ITradeSocketClient`
- `IBookTickerSocketClient`
- `IKlineSocketClient`
- `IOrderBookSocketClient`

Call `SharedClient.Discover()` before relying on optional capabilities.

## Symbols

- Native format is `QUOTE-BASE`.
- South Korea examples: `KRW-BTC`, `USDT-ETH`.
- Singapore example: `SGD-BTC`.
- Indonesia example: `IDR-BTC`.
- Thailand example: `THB-BTC`.
- SharedApis uses `new SharedSymbol(TradingMode.Spot, "ETH", "USDT")` and formats it natively.

## Result Types

- REST: `HttpResult<T>`
- Websocket subscriptions: `WebSocketResult<UpdateSubscription>`
- Shared symbol/cache helpers: `ExchangeCallResult<T>`

Always check `Success` before using `Data`.

## Factories

`IUpbitOrderBookFactory`:

- `Spot`
- `Create(SharedSymbol, options)`
- `CreateSpot(symbol, options)`

`IUpbitTrackerFactory` implements the shared `ITrackerFactory` for public kline and trade trackers.
