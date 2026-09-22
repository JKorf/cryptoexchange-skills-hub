---
name: cryptoclients-net
description: Build C#/.NET multi-exchange integrations with CryptoClients.Net, focusing on Shared API V2 capability discovery and execution through IExchangeSharedApiClient, exchange aggregates, strict REST and websocket capabilities, shared symbols and models, async fan-out, credentials, dependency injection, native exchange access, cross-exchange order books, trackers, and user client providers.
---

# CryptoClients.Net

## Overview

Use `CryptoClients.Net` when an application needs broad exchange coverage from one package. Prefer `IExchangeSharedApiClient` and strict Shared API V2 capabilities for portable workflows, then use the exposed native clients for exchange-specific endpoints or models.

Use an individual exchange skill instead when the application targets only one exchange or needs substantial exchange-specific behavior.

## Setup

This skill targets `CryptoClients.Net` 5.6.0, the latest published stable release verified on 2026-08-24.

```bash
dotnet add package CryptoClients.Net --version 5.6.0
```

```csharp
using CryptoClients.Net;
using CryptoClients.Net.Clients;
using CryptoClients.Net.Enums;
using CryptoClients.Net.Interfaces;
using CryptoClients.Net.Models;
using CryptoExchange.Net;
using CryptoExchange.Net.Objects;
using CryptoExchange.Net.SharedApis;
```

## Aggregate Clients

```csharp
var rest = new ExchangeRestClient();
var socket = new ExchangeSocketClient();
var shared = new ExchangeSharedApiClient(new CryptoClientsConfiguration());
```

Use `IExchangeSharedApiClient` for Shared API V2. Use `IExchangeRestClient` and `IExchangeSocketClient` for the legacy aggregate API or direct native clients.

## Shared API V2 First

Resolve strict capabilities with typed `SharedCapabilities` descriptors:

```csharp
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");
var request = new GetTickerRequest(symbol);
var exchanges = new[] { Exchange.Binance, Exchange.Bybit, Exchange.Kraken, Exchange.OKX };

var matches = shared.GetCapabilities(
    SharedCapabilities.Tickers.GetTicker.Rest,
    TradingMode.Spot,
    exchanges);

var calls = matches.Select(async match =>
    (Match: match,
     Result: await match.Capability.GetTickerAsync(request)));

await foreach (var item in calls.ParallelEnumerateAsync())
{
    if (!item.Result.Success)
    {
        Console.WriteLine($"{item.Match.Exchange}: {item.Result.Error}");
        continue;
    }

    Console.WriteLine($"{item.Match.Exchange}: {item.Result.Data.LastPrice}");
}
```

`GetCapabilities` returns one preferred implementation per exchange. Use `GetImplementations` when every matching transport or API surface is needed, including multiple implementations from one exchange.

## Result Shapes

- REST capabilities such as `IGetTickerRest`: `HttpResult<T>`
- Websocket request capabilities: `QueryResult<T>`
- Subscription capabilities: `WebSocketResult<UpdateSubscription>`
- Transport-independent capabilities: `IExchangeCallResult<T>`
- Capability lookup: `SharedCapabilityResolution<T>`

Check each result independently. One exchange failure must not hide successful results from other exchanges.

## Capability Discovery

Get one preferred capability when the exchange is known dynamically:

```csharp
var match = shared.GetCapability(
    Exchange.Binance,
    SharedCapabilities.Tickers.GetTicker.Rest,
    TradingMode.Spot);

if (match is not null)
{
    var result = await match.Capability.GetTickerAsync(
        new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "ETH", "USDT")));
}
```

Use typed exchange properties such as `shared.Binance`, `shared.Kraken`, and `shared.OKX` when the exchange is known at compile time. `GetClient(exchange)` returns the exchange aggregate as `ISharedApiClientBase` for dynamic inspection.

## Asset Types And Symbol Catalogs

Use `SharedAssetType` and `SharedAssetSubType` to classify or filter the base and quote assets returned by shared symbol APIs. Pass the filters through `GetSymbolsRequest`; do not infer asset classes from exchange-specific symbol names.

Access `SpotSymbolCatalog` through `IGetSpotSymbolsRest` and `FuturesSymbolCatalog` through `IGetFuturesSymbolsRest`. Fetch the corresponding symbols successfully before reading a catalog because each property is `null` until its client cache has been populated.

Read `references/api-surfaces.md` for enum values, model properties, valid type/subtype combinations, and catalog structure. Read `references/usage.md` for filtering and catalog lookup examples.

## Full Exchange API Access Through REST And Socket Clients

`IExchangeRestClient` and `IExchangeSocketClient` expose the bundled native clients directly. This is separate from the Shared API V2 aggregation provided by `IExchangeSharedApiClient`:

```csharp
var binance = await rest.Binance.SpotApi.ExchangeData.GetTickerAsync("ETHUSDT");
var kucoin = await rest.Kucoin.SpotApi.ExchangeData.GetTickerAsync("ETH-USDT");
var bitfinex = await rest.Bitfinex.ExchangeApi.ExchangeData.GetTickerAsync("tETHUSD");
```

Use native access when the shared request lacks an option, the shared model omits required fields, or the endpoint has no shared interface. Follow the corresponding exchange skill for roots, credentials, symbols, and safety.

`IExchangeRestClient` exposes direct REST clients for all bundled libraries, including CoinGecko and Polymarket. `IExchangeSocketClient` exposes direct clients for socket-capable libraries, including Polymarket. CoinGecko and Polymarket are not currently exposed by `IExchangeSharedApiClient`, so access them through these native REST or socket properties.

Configure `GlobalExchangeOptions.EnabledExchanges` when only a subset should be available; native clients and factories are created lazily on first access.

## Websocket Capability Fan-Out

```csharp
var matches = shared.GetCapabilities(
    SharedCapabilities.Tickers.SubscribeTicker,
    TradingMode.Spot,
    new[] { Exchange.Binance, Exchange.Bybit, Exchange.OKX });

var subscriptions = await Task.WhenAll(matches.Select(async match =>
    (Match: match,
     Result: await match.Capability.SubscribeToTickerUpdatesAsync(
         new SubscribeTickerRequest(
             new SharedSymbol(TradingMode.Spot, "BTC", "USDT")),
         update => Console.WriteLine(
             $"{match.Exchange}: {update.Data.LastPrice}")))));

foreach (var subscription in subscriptions.Where(x => x.Result.Success))
    await subscription.Result.Data.CloseAsync();
```

Keep handlers fast and close every successful subscription during shutdown.

## Credentials

Use typed `ExchangeCredentials` when exchanges are known at compile time. Use `DynamicCredentials` only when the exchange is selected at runtime, after calling `ExchangeCredentials.GetDynamicCredentialInfo(...)` to discover required fields.

Do not use the obsolete `SetApiCredentials(exchange, apiKey, apiSecret, apiPass)` overload; it cannot represent every exchange credential shape.

## Dependency Injection

```csharp
services.AddCryptoClients(options =>
{
    options.RequestTimeout = TimeSpan.FromSeconds(15);
    options.EnabledExchanges = new[] { Exchange.Binance, Exchange.Kraken, Exchange.OKX };
});
```

`AddCryptoClients` registers `IExchangeSharedApiClient`, exchange-specific Shared API aggregates, strict capabilities, native exchange clients, `IExchangeRestClient`, `IExchangeSocketClient`, factories, and providers. Prefer injecting `IExchangeSharedApiClient` for new portable workflows.

## Order Books And Trackers

Use `IExchangeOrderBookFactory` for individual books or `CreateCrossExchange(...)` for a combined cross-exchange book. Use `IExchangeTrackerFactory` for kline, trade, spot-user-data, and futures-user-data trackers. Factory methods can return `null` when an exchange or trading mode is unsupported.

Call `CanCreateKlineTracker(exchange, symbol, interval)` or `CanCreateTradeTracker(exchange, symbol)` before creating those trackers when support is determined dynamically.

## References

- Read `references/api-surfaces.md` for aggregate methods, direct clients, discovery, asset classifications, symbol catalogs, DI, credentials, factories, and result types.
- Read `references/usage.md` for practical aggregate, native, symbol filtering/catalog, websocket, credentials, and DI patterns.
- Read `references/safety.md` before credentials, private data, orders, leverage, transfers, withdrawals, or production fan-out.
- In a maintainer checkout, read `../CryptoClients.Net/Examples/ai-friendly/` for current examples. When an example and a public interface differ, follow the current interface signature and result type.
