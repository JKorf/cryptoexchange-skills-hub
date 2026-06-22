# Upbit.Net Usage

Use these snippets as compact patterns for Upbit public market data. They assume:

```csharp
using CryptoExchange.Net.Objects;
using Upbit.Net;
using Upbit.Net.Clients;
using Upbit.Net.Enums;
```

## Public Ticker

```csharp
var client = new UpbitRestClient();

var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("USDT-ETH");
if (!ticker.Success)
{
    Console.WriteLine($"Ticker failed: {ticker.Error}");
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

## Select A Region

```csharp
var client = new UpbitRestClient(options =>
{
    options.Environment = UpbitEnvironment.Singapore;
});
```

Use `Live`, `Singapore`, `Indonesia`, or `Thailand` according to the requested market.

## Symbols And Regional Validation

```csharp
var symbols = await client.SpotApi.ExchangeData.GetSymbolsAsync(
    includeNotifications: true);

if (!symbols.Success)
{
    Console.WriteLine($"Symbols failed: {symbols.Error}");
    return;
}

var supported = symbols.Data.Any(x =>
    string.Equals(x.Symbol, "USDT-ETH", StringComparison.OrdinalIgnoreCase));
```

Native Upbit symbols use `QUOTE-BASE` format.

## Tickers By Quote Asset

```csharp
var tickers = await client.SpotApi.ExchangeData.GetTickersByQuoteAssetsAsync(
    new[] { "KRW", "BTC", "USDT" });

if (!tickers.Success)
{
    Console.WriteLine($"Tickers failed: {tickers.Error}");
    return;
}
```

Use regional fiat quotes such as `SGD`, `IDR`, or `THB` with their matching environment.

## Trades And Klines

```csharp
var trades = await client.SpotApi.ExchangeData.GetTradeHistoryAsync(
    "USDT-ETH",
    limit: 20);

var klines = await client.SpotApi.ExchangeData.GetKlinesAsync(
    "USDT-ETH",
    KlineInterval.OneMinute,
    endTime: DateTime.UtcNow,
    limit: 20);
```

Check each result separately before using `Data`.

## Order Book

```csharp
var book = await client.SpotApi.ExchangeData.GetOrderBookAsync(
    "USDT-ETH",
    levels: 15);

if (!book.Success)
{
    Console.WriteLine($"Order book failed: {book.Error}");
    return;
}

var best = book.Data.Entries.FirstOrDefault();
if (best != null)
    Console.WriteLine($"Bid {best.BidPrice}, ask {best.AskPrice}");
```

## Symbol Configuration

```csharp
var config = await client.SpotApi.ExchangeData.GetSymbolConfigAsync("USDT-ETH");
if (!config.Success)
{
    Console.WriteLine($"Symbol config failed: {config.Error}");
    return;
}
```

## Public Websocket Streams

```csharp
var socket = new UpbitSocketClient();

var tickerSub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "USDT-ETH",
    update => Console.WriteLine(update.Data.LastPrice));

var tradeSub = await socket.SpotApi.SubscribeToTradeUpdatesAsync(
    "USDT-ETH",
    update => Console.WriteLine($"{update.Data.Quantity} @ {update.Data.Price}"));

if (!tickerSub.Success || !tradeSub.Success)
{
    if (tickerSub.Success)
        await socket.UnsubscribeAsync(tickerSub.Data);
    Console.WriteLine(tickerSub.Error ?? tradeSub.Error);
    return;
}

await socket.UnsubscribeAsync(tradeSub.Data);
await socket.UnsubscribeAsync(tickerSub.Data);
```

## Order Book Websocket

```csharp
var sub = await socket.SpotApi.SubscribeToOrderBookUpdatesAsync(
    "USDT-ETH",
    levels: 15,
    update => Console.WriteLine(update.Data.Entries.FirstOrDefault()?.BidPrice),
    aggregation: 0.01m);
```

Subscription methods return `WebSocketResult<UpdateSubscription>`.

## SharedApis Spot Ticker

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new UpbitRestClient().SpotApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USDT");

var result = await tickers.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

## Dependency Injection

```csharp
services.AddUpbit(options =>
{
    options.Environment = UpbitEnvironment.Live;
});
```

Inject `IUpbitRestClient`, `IUpbitSocketClient`, `IUpbitOrderBookFactory`, or `IUpbitTrackerFactory`.

## Error Handling

```csharp
var result = await client.SpotApi.ExchangeData.GetTickerAsync("USDT-ETH");
if (!result.Success)
{
    if (result.Error?.IsTransient == true)
    {
        // Retry with bounded backoff.
    }

    Console.WriteLine($"{result.Error?.Code}: {result.Error?.Message}");
    return;
}
```

Normal API, network, and rate-limit failures are returned on `.Error`.
