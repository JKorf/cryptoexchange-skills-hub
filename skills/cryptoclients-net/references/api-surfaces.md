# CryptoClients.Net API Surfaces

## Package And Aggregate Types

- Package: `CryptoClients.Net`
- REST: `ExchangeRestClient`, `IExchangeRestClient`
- Socket: `ExchangeSocketClient`, `IExchangeSocketClient`
- DI: `services.AddCryptoClients(...)`
- Credentials: `ExchangeCredentials`, `DynamicCredentials`
- Factories: `IExchangeOrderBookFactory`, `IExchangeTrackerFactory`
- Multi-user clients: `IExchangeUserClientProvider`

The package currently targets .NET 8, 9, 10, .NET Standard 2.0, and .NET Standard 2.1.

## Shared REST Aggregation

Most aggregate operation families provide three shapes:

- one exchange: method accepting `string exchange`, returning `HttpResult<T>`
- selected/all supported exchanges: method accepting `IEnumerable<string>? exchanges`, returning `HttpResult<T>[]`
- response-as-completed processing: `...AsyncEnumerable(...)`, returning `IAsyncEnumerable<HttpResult<T>>`

Families include spot/futures tickers and symbols, book tickers, order books, klines, recent trades, trade history, assets, balances, deposits, withdrawals, fees, funding rates, open interest, leverage, position mode/history, spot/futures orders, trigger orders, TP/SL, transfers, and listen keys where supported.

The request's `TradingMode` disambiguates exchanges that expose multiple APIs. A one-exchange call can fail when multiple matching APIs exist and no mode is specified.

## Shared Socket Aggregation

Aggregate subscriptions include ticker/all-ticker, book ticker, order book, trade, kline, balance, spot order, futures order, position, and user-trade updates.

One-exchange methods return `WebSocketResult<UpdateSubscription>`. Multi-exchange methods return arrays. Handlers receive `DataEvent<T>` with `Exchange` populated.

Use `UnsubscribeAllAsync()` for aggregate teardown.

## Capability Discovery

Each operation family exposes getters for supported shared clients, for example:

- `GetSpotTickerClient(exchange)` / `GetSpotTickerClients()`
- `GetFuturesTickerClient(mode, exchange)` / `GetFuturesTickerClients(mode)`
- `GetOrderBookClient(mode, exchange)` / `GetOrderBookClients(mode)`
- `GetSpotOrderClient(exchange)`
- `GetFuturesOrderClient(mode, exchange)`
- socket equivalents such as `GetTickerClient(mode, exchange)`

Single-client getters are nullable. `GetExchangeSharedClients(exchange, tradingMode)` returns every registered `ISharedClient` matching that exchange and optional mode.

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

- `GetSpotSymbolClient(exchange)` returns nullable `ISpotSymbolRestClient`; after a successful `GetSpotSymbolsAsync(...)`, read `SpotSymbolCatalog`.
- `GetFuturesSymbolClient(tradingMode, exchange)` returns nullable `IFuturesSymbolRestClient`; after a successful `GetFuturesSymbolsAsync(...)`, read `FuturesSymbolCatalog`.

Both properties are nullable `SharedSymbolCatalog` instances and are maintained separately. `SharedSymbolCatalog.Exchange` identifies the exchange, `Assets` is keyed by asset name and contains `SharedAssetInfo` (`Name`, `Type`, `SubType`), and `Symbols` is keyed by the exchange symbol name. Do not read either catalog before its corresponding symbol fetch.

## Direct Native Clients

`ExchangeRestClient` exposes `Aster`, `Binance`, `BingX`, `Bitfinex`, `Bitget`, `BitMart`, `BitMEX`, `Bitstamp`, `BloFin`, `Bybit`, `Coinbase`, `CoinEx`, `CoinGecko`, `CoinW`, `CryptoCom`, `DeepCoin`, `GateIo`, `HTX`, `HyperLiquid`, `Kraken`, `Kucoin`, `Mexc`, `OKX`, `Polymarket`, `Toobit`, `Upbit`, `Weex`, `WhiteBit`, and `XT`.

`ExchangeSocketClient` exposes the same socket-capable native clients except CoinGecko.

CoinGecko and Polymarket direct properties exist, but current aggregate SharedApis initialization does not add them to `_sharedClients`.

## Credentials And Options

`GlobalExchangeOptions` can apply credentials, environment names, proxy, original-data output, timeout, rate limiting, caching, reconnect policy, and reconnect interval across clients.

`ExchangeCredentials` stores typed credentials per exchange. `DynamicCredentials` carries `TradingMode`, key, and up to three exchange-specific parameters. Inspect requirements through `ExchangeCredentials.GetDynamicCredentialInfo(mode, exchange)` before constructing dynamic credentials.

Set credentials on aggregate REST/socket clients through `SetApiCredentials(ExchangeCredentials)` or `SetApiCredentials(exchange, DynamicCredentials)`.

## DI And Factories

`AddCryptoClients(...)` registers all bundled native clients and aggregate interfaces. It accepts global options, per-exchange library option delegates, optional socket-client lifetime, or an `IConfiguration` section.

`IExchangeOrderBookFactory` creates one/many local books and `ICrossExchangeBook` instances. `IExchangeTrackerFactory` creates kline, trade, and user-data trackers. `IExchangeUserClientProvider` caches aggregate REST/socket clients by user identifier.

## Result Types

- REST: `HttpResult<T>` and arrays of `HttpResult<T>`
- Socket: `WebSocketResult<UpdateSubscription>` and arrays thereof
- Symbol/capability helpers: `ExchangeCallResult<T>`

Always check each result's `Success`, `Exchange`, and `Error` independently.
