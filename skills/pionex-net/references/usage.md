# Pionex.Net Usage

Use these compact patterns for native Pionex Spot workflows. They assume:

```csharp
using CryptoExchange.Net.Objects;
using Pionex.Net;
using Pionex.Net.Clients;
using Pionex.Net.Enums;
```

## Public Ticker

```csharp
var client = new PionexRestClient();

var result = await client.SpotApi.ExchangeData.GetTickersAsync(
    "BTC_USDT",
    SymbolType.Spot);

if (!result.Success)
{
    Console.WriteLine($"Ticker failed: {result.Error}");
    return;
}

Console.WriteLine(result.Data.Single().ClosePrice);
```

Ticker calls return arrays even with one symbol.

## Symbols and Trading Rules

```csharp
var result = await client.SpotApi.ExchangeData.GetSymbolsAsync(
    symbols: "ETH_USDT",
    symbolType: SymbolType.Spot);

if (!result.Success)
{
    Console.WriteLine($"Symbols failed: {result.Error}");
    return;
}

var symbol = result.Data.Single();
Console.WriteLine(
    $"Enabled={symbol.Enable}, precision={symbol.QuantityPrecision}, min={symbol.MinSpotQuantity}");
```

Validate current metadata before constructing live orders.

## Order Book and Klines

```csharp
var book = await client.SpotApi.ExchangeData.GetOrderBookAsync(
    "BTC_USDT",
    limit: 20);

var klines = await client.SpotApi.ExchangeData.GetKlinesAsync(
    "BTC_USDT",
    KlineInterval.OneMinute,
    limit: 100);
```

Check each result separately before using `Data`.

## Authenticated Client and Balances

```csharp
var client = new PionexRestClient(options =>
{
    options.ApiCredentials =
        new PionexCredentials("API_KEY", "API_SECRET");
});

var result = await client.SpotApi.Account.GetBalancesAsync();
if (!result.Success)
{
    Console.WriteLine($"Balances failed: {result.Error}");
    return;
}

foreach (var balance in result.Data.Where(
    x => x.Free != 0 || x.Frozen != 0))
{
    Console.WriteLine(
        $"{balance.Asset}: free={balance.Free}, frozen={balance.Frozen}");
}
```

## Limit Order Lifecycle

This calls live trading endpoints:

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "ETH_USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.01m,
    price: 2000m,
    clientOrderId: $"example-{Guid.NewGuid():N}");

if (!order.Success)
{
    Console.WriteLine($"Order failed: {order.Error}");
    return;
}

var details = await client.SpotApi.Trading.GetOrderAsync(
    order.Data.OrderId);

if (!details.Success)
{
    Console.WriteLine($"Lookup failed: {details.Error}");
    return;
}

var cancel = await client.SpotApi.Trading.CancelOrderAsync(
    "ETH_USDT",
    order.Data.OrderId);

if (!cancel.Success)
    Console.WriteLine($"Cancel failed: {cancel.Error}");
```

Do not describe the example price as a safety guarantee.

## Market Orders

Market buy with quote amount:

```csharp
var buy = await client.SpotApi.Trading.PlaceOrderAsync(
    "ETH_USDT",
    OrderSide.Buy,
    OrderType.Market,
    quoteQuantity: 25m);
```

Market sell with base quantity:

```csharp
var sell = await client.SpotApi.Trading.PlaceOrderAsync(
    "ETH_USDT",
    OrderSide.Sell,
    OrderType.Market,
    quantity: 0.01m);
```

## Public Websocket Streams

```csharp
var socket = new PionexSocketClient();

var trades = await socket.SpotApi.SubscribeToTradeUpdatesAsync(
    "BTC_USDT",
    update =>
    {
        foreach (var trade in update.Data)
            Console.WriteLine($"{trade.Quantity} @ {trade.Price}");
    });

var book = await socket.SpotApi.SubscribeToOrderBookUpdatesAsync(
    "BTC_USDT",
    depth: 20,
    update => Console.WriteLine(
        update.Data.Bids.FirstOrDefault()?.Price));

if (!trades.Success || !book.Success)
{
    if (trades.Success)
        await socket.UnsubscribeAsync(trades.Data);

    Console.WriteLine(trades.Error ?? book.Error);
    return;
}

await socket.UnsubscribeAsync(book.Data);
await socket.UnsubscribeAsync(trades.Data);
```

## Private Websocket Stream

```csharp
var socket = new PionexSocketClient(options =>
{
    options.ApiCredentials =
        new PionexCredentials("API_KEY", "API_SECRET");
});

var balances = await socket.SpotApi.SubscribeToBalanceUpdatesAsync(
    update => Console.WriteLine(update.Data.Length));

if (!balances.Success)
{
    Console.WriteLine(balances.Error);
    return;
}

await socket.UnsubscribeAsync(balances.Data);
```

No listen key is required.

## SharedApis Ticker

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers =
    new PionexRestClient().SpotApi.SharedClient;

var symbol = new SharedSymbol(
    TradingMode.Spot,
    "BTC",
    "USDT");

var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(symbol));

if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

## Dependency Injection

```csharp
services.AddPionex(options =>
{
    options.ApiCredentials =
        new PionexCredentials("API_KEY", "API_SECRET");
});
```

Inject the client or factory interface required by the service.

## Transient Retry

```csharp
var result = await client.SpotApi.ExchangeData
    .GetOrderBookAsync("BTC_USDT");

if (!result.Success)
{
    if (result.Error?.IsTransient == true)
    {
        // Retry with bounded exponential backoff and cancellation.
    }

    Console.WriteLine(
        $"{result.Error?.Code}: {result.Error?.Message}");
}
```
