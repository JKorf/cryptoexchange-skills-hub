# CryptoClients.Net Usage

## Contents

- Aggregate REST fan-out
- Async-enumerable fan-out
- Capability discovery
- Asset-type filtering and symbol catalogs
- Full native API access
- Aggregate websocket subscriptions
- Typed and dynamic credentials
- Dependency injection
- Order books and trackers

## Aggregate REST Fan-Out

```csharp
using CryptoClients.Net;
using CryptoClients.Net.Enums;
using CryptoExchange.Net.SharedApis;

var client = new ExchangeRestClient();
var request = new GetTickerRequest(
    new SharedSymbol(TradingMode.Spot, "BTC", "USDT"));

var results = await client.GetSpotTickerAsync(
    request,
    new[] { Exchange.Binance, Exchange.Bybit, Exchange.Kraken, Exchange.OKX });

foreach (var result in results)
{
    Console.WriteLine(result.Success
        ? $"{result.Exchange}: {result.Data.LastPrice}"
        : $"{result.Exchange}: {result.Error}");
}
```

For one exchange, put the exchange first and receive one result:

```csharp
var result = await client.GetSpotTickerAsync(Exchange.Binance, request);
```

## Async-Enumerable Fan-Out

Process results as exchanges respond:

```csharp
await foreach (var result in client.GetSpotTickerAsyncEnumerable(
    request,
    new[] { Exchange.Binance, Exchange.Bybit, Exchange.OKX }))
{
    if (result.Success)
        Console.WriteLine($"{result.Exchange}: {result.Data.LastPrice}");
}
```

## Capability Discovery

```csharp
var client = new ExchangeRestClient();
var ticker = client.GetSpotTickerClient(Exchange.Binance);

if (ticker != null)
{
    var result = await ticker.GetSpotTickerAsync(request);
}

foreach (var supported in client.GetFuturesTickerClients(TradingMode.PerpetualLinear))
    Console.WriteLine(supported.Exchange);
```

Use `GetExchangeSharedClients(exchange, mode)` with shared-client extension helpers when dynamically composing capabilities.

## Asset-Type Filtering And Symbol Catalogs

Filter aggregate symbol requests without relying on exchange-specific names:

```csharp
var request = new GetSymbolsRequest(
    baseAssetType: SharedAssetType.TradFi,
    baseAssetSubType: SharedAssetSubType.Equity,
    quoteAssetType: SharedAssetType.Crypto,
    quoteAssetSubType: SharedAssetSubType.StableCoin);

var results = await client.GetFuturesSymbolsAsync(
    request,
    new[] { Exchange.Binance, Exchange.Bybit, Exchange.OKX });

foreach (var result in results.Where(x => x.Success))
    Console.WriteLine($"{result.Exchange}: {result.Data.Length} equity markets");
```

Use a concrete shared symbol client when a reusable catalog is needed. Fetch symbols before reading the nullable catalog:

```csharp
var spotSymbols = client.GetSpotSymbolClient(Exchange.Binance)
    ?? throw new NotSupportedException("Binance spot symbols are unavailable");

var result = await spotSymbols.GetSpotSymbolsAsync(new GetSymbolsRequest());
if (!result.Success)
    throw new InvalidOperationException(result.Error?.ToString());

var catalog = spotSymbols.SpotSymbolCatalog!;

if (catalog.Assets.TryGetValue("USDT", out var usdt))
    Console.WriteLine($"{usdt.Name}: {usdt.Type} / {usdt.SubType}");

if (catalog.Symbols.TryGetValue("BTCUSDT", out var btcUsdt))
    Console.WriteLine($"{btcUsdt.BaseAsset}/{btcUsdt.QuoteAsset}");
```

Futures catalogs follow the same lifecycle, but select the trading mode when retrieving the client:

```csharp
var futuresSymbols = client.GetFuturesSymbolClient(
    TradingMode.PerpetualLinear,
    Exchange.Binance);

if (futuresSymbols != null)
{
    var result = await futuresSymbols.GetFuturesSymbolsAsync(
        new GetSymbolsRequest(tradingMode: TradingMode.PerpetualLinear));

    if (result.Success)
        Console.WriteLine(futuresSymbols.FuturesSymbolCatalog!.Symbols.Count);
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

## Aggregate Websocket Subscriptions

```csharp
var socket = new ExchangeSocketClient();
var request = new SubscribeTickerRequest(
    new SharedSymbol(TradingMode.Spot, "BTC", "USDT"));

var subscriptions = await socket.SubscribeToTickerUpdatesAsync(
    request,
    update => Console.WriteLine($"{update.Exchange}: {update.Data.LastPrice}"),
    new[] { Exchange.Binance, Exchange.Bybit, Exchange.OKX });

foreach (var failed in subscriptions.Where(x => !x.Success))
    Console.WriteLine($"{failed.Exchange}: {failed.Error}");

await socket.UnsubscribeAllAsync();
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

Inject `IExchangeRestClient`, `IExchangeSocketClient`, `IExchangeOrderBookFactory`, `IExchangeTrackerFactory`, or `IExchangeUserClientProvider`.

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
