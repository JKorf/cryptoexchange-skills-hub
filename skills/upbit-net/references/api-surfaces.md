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

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.SpotApi.SharedApi`
- `socketClient.SpotApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `IUpbitSharedApiClient`. It exposes:

- `SpotRest` as `IUpbitRestClientSpotSharedApi`
- `SpotSocket` as `IUpbitSocketClientSpotSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `IUpbitSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.

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
