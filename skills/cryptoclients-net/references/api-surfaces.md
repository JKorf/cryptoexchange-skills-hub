# CryptoClients.Net API Surfaces

## Package And Aggregate Types

- Package: `CryptoClients.Net`
- Shared API V2 aggregate: `ExchangeSharedApiClient`, `IExchangeSharedApiClient`
- REST: `ExchangeRestClient`, `IExchangeRestClient`
- Socket: `ExchangeSocketClient`, `IExchangeSocketClient`
- DI: `services.AddCryptoClients(...)`
- Credentials: `ExchangeCredentials`, `DynamicCredentials`
- Factories: `IExchangeOrderBookFactory`, `IExchangeTrackerFactory`
- Multi-user clients: `IExchangeUserClientProvider`

The package currently targets .NET 8, 9, 10, .NET Standard 2.0, and .NET Standard 2.1.

## Shared API V2 Aggregation

`IExchangeSharedApiClient` is the V2 entry point. It exposes:

- Typed exchange aggregates such as `Binance`, `Kraken`, and `OKX`.
- `GetClient(exchange)` for a dynamically selected exchange aggregate.
- `GetCapability` for one preferred capability on one exchange.
- `GetCapabilities` for one preferred matching implementation per exchange.
- `GetImplementations` for every matching transport or API surface, including multiple matches from one exchange.

Pass a typed descriptor such as `SharedCapabilities.Tickers.GetTicker.Rest` or `SharedCapabilities.Tickers.SubscribeTicker`. Descriptors let the compiler infer the strict capability interface; they are not implementations and do not guarantee support.

Specify `TradingMode` when an exchange exposes more than one relevant market. Use a transport-specific descriptor or `SharedTransport` when REST versus WebSocket behavior matters.

## Executing Resolved Capabilities

Capability lookup is synchronous and does not send an exchange request. Execute the returned `Capability` explicitly:

```csharp
var matches = shared.GetCapabilities(
    SharedCapabilities.Tickers.GetTicker.Rest,
    TradingMode.Spot,
    exchanges);

var tasks = matches.Select(async match =>
    (Match: match,
     Result: await match.Capability.GetTickerAsync(request)));

await foreach (var item in tasks.ParallelEnumerateAsync())
{
    // Keep item.Match with item.Result for exchange and transport metadata.
}
```

Use `Task.WhenAll` when all results are needed before processing. Use `ParallelEnumerateAsync` when results should be handled as they complete.

## Strict Capability Families

V2 uses one interface per operation. Representative families include:

- Tickers: `IGetTickerRest`, `IGetAllTickersRest`, `ISubscribeTickerSocket`
- Symbols: `IGetSpotSymbolsRest`, `IGetFuturesSymbolsRest`
- Market data: order books, book tickers, trades, klines, funding rates, and open interest
- Account and funding: balances, assets, fees, deposits, withdrawals, and transfers
- Trading: separate place, get, cancel, history, and trade capabilities for spot and futures
- Subscriptions: ticker, trades, klines, order book, balances, orders, positions, and user trades

Inspect the capability's options for supported trading modes, request parameter rules, exchange parameter rules, and operation-specific metadata.

## Asset Classification And Symbol Catalogs

`SharedSpotSymbol` and `SharedFuturesSymbol` classify both sides of a market with:

- `BaseAssetType` and `QuoteAssetType`: `SharedAssetType.Unspecified`, `Crypto`, `Fiat`, or `TradFi`
- `BaseAssetSubType` and `QuoteAssetSubType`: nullable `SharedAssetSubType.StableCoin`, `Equity`, or `Commodity`

Use `Unspecified` when the exchange or client cannot classify an asset. Valid subtype relationships are:

- `Crypto` + `StableCoin`
- `TradFi` + `Equity`
- `TradFi` + `Commodity`
- `Fiat` without a subtype

`GetSymbolsRequest` accepts `baseAssetType`, `baseAssetSubType`, `quoteAssetType`, and `quoteAssetSubType` filters in addition to `tradingMode` and `exchangeParameters`. The same request works with spot and futures symbol methods. Invalid type/subtype combinations fail request validation.

For cached lookup, retrieve a concrete symbol client:

- `IGetSpotSymbolsRest`; after a successful `GetSpotSymbolsAsync(...)`, read `SpotSymbolCatalog`.
- `IGetFuturesSymbolsRest`; after a successful `GetFuturesSymbolsAsync(...)`, read `FuturesSymbolCatalog`.

Both properties are nullable `SharedSymbolCatalog` instances and are maintained separately. `SharedSymbolCatalog.Exchange` identifies the exchange, `Assets` is keyed by asset name and contains `SharedAssetInfo` (`Name`, `Type`, `SubType`), and `Symbols` is keyed by the exchange symbol name. Do not read either catalog before its corresponding symbol fetch.

## Direct Native Clients

`ExchangeRestClient` exposes `Aster`, `Binance`, `BingX`, `Bitfinex`, `Bitget`, `BitMart`, `BitMEX`, `Bitstamp`, `BloFin`, `Bybit`, `Coinbase`, `CoinEx`, `CoinGecko`, `CoinW`, `CryptoCom`, `DeepCoin`, `GateIo`, `HTX`, `HyperLiquid`, `Kraken`, `Kucoin`, `LBank`, `Lighter`, `Mexc`, `OKX`, `Pionex`, `Polymarket`, `Toobit`, `Upbit`, `Weex`, `WhiteBit`, and `XT`.

`ExchangeSocketClient` exposes the same socket-capable native clients except CoinGecko.

CoinGecko and Polymarket direct properties exist, but they are not currently exposed by `IExchangeSharedApiClient`.

## Credentials And Options

`GlobalExchangeOptions` can apply credentials, environment names, proxy, original-data output, timeout, rate limiting, caching, reconnect policy, and reconnect interval across clients.

`GlobalExchangeOptions.EnabledExchanges` limits which exchanges are available. Exchange clients, trackers, order-book factories, and user-client providers are initialized lazily; accessing a disabled exchange fails instead of silently constructing it.

`ExchangeCredentials` stores typed credentials per exchange. `DynamicCredentials` carries `TradingMode`, key, and up to three exchange-specific parameters. Inspect requirements through `ExchangeCredentials.GetDynamicCredentialInfo(mode, exchange)` before constructing dynamic credentials.

Set credentials on aggregate REST/socket clients through `SetApiCredentials(ExchangeCredentials)` or `SetApiCredentials(exchange, DynamicCredentials)`.

## DI And Factories

`AddCryptoClients(...)` registers `IExchangeSharedApiClient`, all bundled exchange Shared API aggregates, strict capabilities, native clients, and supporting aggregate interfaces. It accepts global options, per-exchange library option delegates, optional socket-client lifetime, or an `IConfiguration` section.

`IExchangeOrderBookFactory` creates one/many local books and `ICrossExchangeBook` instances. `IExchangeTrackerFactory` creates kline, trade, and user-data trackers; use `CanCreateKlineTracker(...)` and `CanCreateTradeTracker(...)` for dynamic capability checks. `IExchangeUserClientProvider` caches aggregate REST/socket clients by user identifier.

## Result Types

- REST capabilities: `HttpResult<T>`
- Websocket request capabilities: `QueryResult<T>`
- Websocket subscriptions: `WebSocketResult<UpdateSubscription>`
- Transport-independent capabilities: `IExchangeCallResult<T>`
- Lookup: `SharedCapabilityResolution<T>`

Retain each `SharedCapabilityResolution<T>` alongside its result. Check `Success` and `Error` independently, and use the resolution's `Exchange` and `Transport` metadata for attribution.
