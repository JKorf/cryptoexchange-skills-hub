# CryptoClients.Net Usage

## Contents

- Shared API V2 capability fan-out
- Response-as-completed execution
- Exchange aggregate access
- Asset-type filtering and symbol catalogs
- Full native API access
- Shared websocket subscriptions
- Typed and dynamic credentials
- Dependency injection
- Order books and trackers

## Shared API V2 Capability Fan-Out

```csharp
using CryptoClients.Net;
using CryptoClients.Net.Clients;
using CryptoClients.Net.Enums;
using CryptoClients.Net.Interfaces;
using CryptoExchange.Net;
using CryptoExchange.Net.SharedApis;

IExchangeSharedApiClient shared =
    new ExchangeSharedApiClient(new CryptoClientsConfiguration());
var request = new GetTickerRequest(
    new SharedSymbol(TradingMode.Spot, "BTC", "USDT"));

var matches = shared.GetCapabilities(
    SharedCapabilities.Tickers.GetTicker.Rest,
    TradingMode.Spot,
    new[] { Exchange.Binance, Exchange.Bybit, Exchange.Kraken, Exchange.OKX });

var calls = matches.Select(async match =>
    (Match: match,
     Result: await match.Capability.GetTickerAsync(request)));

await foreach (var item in calls.ParallelEnumerateAsync())
{
    Console.WriteLine(item.Result.Success
        ? $"{item.Match.Exchange}: {item.Result.Data.LastPrice}"
        : $"{item.Match.Exchange}: {item.Result.Error}");
}
```

For one exchange, resolve one preferred capability:

```csharp
var match = shared.GetCapability(
    Exchange.Binance,
    SharedCapabilities.Tickers.GetTicker.Rest,
    TradingMode.Spot);

if (match is not null)
{
    var result = await match.Capability.GetTickerAsync(request);
}
```

## Response-As-Completed Execution

Capability lookup does not send requests. Build tasks from the returned resolutions, then use `ParallelEnumerateAsync` to process results as exchanges respond:

```csharp
var calls = matches.Select(async match =>
    (Match: match,
     Result: await match.Capability.GetTickerAsync(request)));

await foreach (var item in calls.ParallelEnumerateAsync())
{
    if (item.Result.Success)
        Console.WriteLine($"{item.Match.Exchange}: {item.Result.Data.LastPrice}");
}
```

## Exchange Aggregate Access

```csharp
var binance = shared.Binance;
IGetTickerRest ticker = binance.SpotRest;

var result = await ticker.GetTickerAsync(request);

var dynamicClient = shared.GetClient(Exchange.Binance);
var info = dynamicClient?.Discover();
```

Typed properties expose each exchange's aggregate at compile time. Use `GetClient` for dynamic exchange selection, `GetCapability` for one preferred operation, `GetCapabilities` for one preferred match per exchange, and `GetImplementations` when all matching transports or API surfaces are needed.

## Asset-Type Filtering And Symbol Catalogs

Filter aggregate symbol requests without relying on exchange-specific names:

```csharp
var request = new GetSymbolsRequest(
    baseAssetType: SharedAssetType.TradFi,
    baseAssetSubType: SharedAssetSubType.Equity,
    quoteAssetType: SharedAssetType.Crypto,
    quoteAssetSubType: SharedAssetSubType.StableCoin);

var matches = shared.GetCapabilities(
    SharedCapabilities.Symbols.GetFuturesSymbols.Rest,
    TradingMode.PerpetualLinear,
    new[] { Exchange.Binance, Exchange.Bybit, Exchange.OKX });

foreach (var match in matches)
{
    var result = await match.Capability.GetFuturesSymbolsAsync(request);
    if (result.Success)
        Console.WriteLine($"{match.Exchange}: {result.Data.Length} equity markets");
}
```

Use a concrete shared symbol client when a reusable catalog is needed. Fetch symbols before reading the nullable catalog:

```csharp
var spotSymbols = shared.GetCapability(
    Exchange.Binance,
    SharedCapabilities.Symbols.GetSpotSymbols.Rest,
    TradingMode.Spot)
    ?? throw new NotSupportedException("Binance spot symbols are unavailable");

var result = await spotSymbols.Capability.GetSpotSymbolsAsync(new GetSymbolsRequest());
if (!result.Success)
    throw new InvalidOperationException(result.Error?.ToString());

var catalog = spotSymbols.Capability.SpotSymbolCatalog!;

if (catalog.Assets.TryGetValue("USDT", out var usdt))
    Console.WriteLine($"{usdt.Name}: {usdt.Type} / {usdt.SubType}");

if (catalog.Symbols.TryGetValue("BTCUSDT", out var btcUsdt))
    Console.WriteLine($"{btcUsdt.BaseAsset}/{btcUsdt.QuoteAsset}");
```

Futures catalogs follow the same lifecycle, but select the trading mode when retrieving the client:

```csharp
var futuresSymbols = shared.GetCapability(
    Exchange.Binance,
    SharedCapabilities.Symbols.GetFuturesSymbols.Rest,
    TradingMode.PerpetualLinear);

if (futuresSymbols != null)
{
    var result = await futuresSymbols.Capability.GetFuturesSymbolsAsync(
        new GetSymbolsRequest(tradingMode: TradingMode.PerpetualLinear));

    if (result.Success)
        Console.WriteLine(futuresSymbols.Capability.FuturesSymbolCatalog!.Symbols.Count);
}
```

Treat `SharedAssetType.Unspecified` and a `null` subtype as unknown classification, not as evidence that the asset is cryptocurrency. `StableCoin` belongs to `Crypto`; `Equity` and `Commodity` belong to `TradFi`; fiat assets have no subtype.

## Full Native API Access

```csharp
var client = new ExchangeRestClient();

var binance = await client.Binance.SpotApi.ExchangeData.GetTickerAsync("ETHUSDT");
var bitfinex = await client.Bitfinex.ExchangeApi.ExchangeData.GetTickerAsync("tETHUSD");
var okx = await client.OKX.UnifiedApi.ExchangeData.GetTickerAsync("ETH-USDT");
```

Native methods use exchange-specific symbols, enums, requests, models, and result details. Read that exchange's skill before generating non-trivial native code.

## Shared Websocket Subscriptions

```csharp
var request = new SubscribeTickerRequest(
    new SharedSymbol(TradingMode.Spot, "BTC", "USDT"));

var matches = shared.GetCapabilities(
    SharedCapabilities.Tickers.SubscribeTicker,
    TradingMode.Spot,
    new[] { Exchange.Binance, Exchange.Bybit, Exchange.OKX });

var subscriptions = await Task.WhenAll(matches.Select(async match =>
    (Match: match,
     Result: await match.Capability.SubscribeToTickerUpdatesAsync(
         request,
         update => Console.WriteLine(
             $"{match.Exchange}: {update.Data.LastPrice}")))));

foreach (var subscription in subscriptions.Where(x => x.Result.Success))
    await subscription.Result.Data.CloseAsync();
```

## Typed And Dynamic Credentials

```csharp
using Binance.Net;
using CryptoClients.Net.Models;
using OKX.Net;

var client = new ExchangeRestClient(options =>
{
    options.ApiCredentials = new ExchangeCredentials
    {
        Binance = new BinanceCredentials("BINANCE_KEY", "BINANCE_SECRET"),
        OKX = new OKXCredentials("OKX_KEY", "OKX_SECRET", "OKX_PASSPHRASE")
    };
});
```

For runtime-selected exchanges:

```csharp
var info = ExchangeCredentials.GetDynamicCredentialInfo(TradingMode.Spot, Exchange.OKX);
if (info != null)
{
    client.SetApiCredentials(Exchange.OKX, new DynamicCredentials(
        TradingMode.Spot,
        "OKX_KEY",
        param1: "OKX_SECRET",
        param2: "OKX_PASSPHRASE"));
}
```

## Dependency Injection

```csharp
services.AddCryptoClients(options =>
{
    options.RequestTimeout = TimeSpan.FromSeconds(15);
    options.ApiCredentials = new ExchangeCredentials
    {
        Binance = new BinanceCredentials("API_KEY", "API_SECRET")
    };
});
```

Inject `IExchangeSharedApiClient` for Shared API V2. Inject `IExchangeRestClient`, `IExchangeSocketClient`, `IExchangeOrderBookFactory`, `IExchangeTrackerFactory`, or `IExchangeUserClientProvider` for legacy aggregation, native access, and supporting services.

## Order Books And Trackers

```csharp
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");

var crossBook = orderBookFactory.CreateCrossExchange(
    symbol,
    minimalDepth: 20,
    exchanges: new[] { Exchange.Binance, Exchange.Bybit, Exchange.OKX });

var tracker = trackerFactory.CreateTradeTracker(
    Exchange.Binance,
    symbol,
    limit: 100,
    period: TimeSpan.FromMinutes(10));
```

Start and stop order books explicitly. Handle nullable single-exchange factory results.
