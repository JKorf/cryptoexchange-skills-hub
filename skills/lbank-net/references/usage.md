# LBank.Net Usage

Use these compact patterns for native LBank Spot workflows. They assume:

```csharp
using CryptoExchange.Net.Objects;
using LBank.Net;
using LBank.Net.Clients;
using LBank.Net.Enums;
```

## Public Ticker

```csharp
var client = new LBankRestClient();
var result = await client.SpotApi.ExchangeData
    .GetTickersAsync("eth_usdt");

if (!result.Success)
{
    Console.WriteLine($"Ticker failed: {result.Error}");
    return;
}

Console.WriteLine(result.Data.Single().Ticker.LastPrice);
```

Ticker calls return arrays even when filtered to one symbol.

## Symbols and Trading Rules

```csharp
var result = await client.SpotApi.ExchangeData
    .GetSymbolsAsync("eth_usdt");

if (!result.Success)
{
    Console.WriteLine($"Symbols failed: {result.Error}");
    return;
}

var symbol = result.Data.Single();
Console.WriteLine(
    $"Min={symbol.MinOrderQuantity}, quantity decimals={symbol.QuantityAccuracy}, price decimals={symbol.PriceAccuracy}");
```

Query current metadata before constructing a live order.

## Order Book and Klines

```csharp
var book = await client.SpotApi.ExchangeData
    .GetOrderBookAsync("eth_usdt", limit: 20);

var klines = await client.SpotApi.ExchangeData.GetKlinesAsync(
    "eth_usdt",
    KlineInterval.OneMinute,
    limit: 100,
    afterTime: DateTime.UtcNow.AddHours(-2));
```

Check each result separately before using `Data`. Klines require both `limit` and `afterTime`.

## Authenticated Client and Balances

```csharp
var client = new LBankRestClient(options =>
{
    options.ApiCredentials =
        new LBankCredentials("API_KEY", "API_SECRET");
});

var result = await client.SpotApi.Account.GetUserAssetsAsync();
if (!result.Success)
{
    Console.WriteLine($"Balances failed: {result.Error}");
    return;
}

foreach (var balance in result.Data.Where(
    x => x.Quantity != 0 || x.FrozenQuantity != 0))
{
    Console.WriteLine(
        $"{balance.Asset}: total={balance.Quantity}, available={balance.UsableQuantity}, frozen={balance.FrozenQuantity}");
}
```

## Limit Order Lifecycle

This calls live trading endpoints:

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "eth_usdt",
    orderType: OrderType.BuyLimit,
    quantity: 0.01m,
    price: 2000m,
    clientOrderId: $"example-{Guid.NewGuid():N}");

if (!order.Success)
{
    Console.WriteLine($"Order failed: {order.Error}");
    return;
}

var details = await client.SpotApi.Trading.GetOrderAsync(
    "eth_usdt",
    orderId: order.Data.OrderId);

var cancel = await client.SpotApi.Trading.CancelOrderAsync(
    "eth_usdt",
    orderId: order.Data.OrderId);

if (!cancel.Success)
    Console.WriteLine($"Cancel failed: {cancel.Error}");
```

Check `details.Success` before reading its data. Do not describe an example price as a safety guarantee.

## Paginated Orders

```csharp
var openOrders = await client.SpotApi.Trading.GetOpenOrdersAsync(
    "eth_usdt",
    page: 1,
    pageSize: 50);
```

Both page arguments are required. Page size is at most 200.

## Public Websocket Streams

```csharp
var socket = new LBankSocketClient();

var ticker = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "eth_usdt",
    update => Console.WriteLine(update.Data.LastPrice));

var book = await socket.SpotApi.SubscribeToOrderBookUpdatesAsync(
    "eth_usdt",
    depth: 50,
    update => Console.WriteLine(
        update.Data.Bids.FirstOrDefault()?.Price));

if (!ticker.Success || !book.Success)
{
    if (ticker.Success)
        await socket.UnsubscribeAsync(ticker.Data);

    Console.WriteLine(ticker.Error ?? book.Error);
    return;
}

await socket.UnsubscribeAsync(book.Data);
await socket.UnsubscribeAsync(ticker.Data);
```

## Private Websocket Stream

```csharp
var socket = new LBankSocketClient(options =>
{
    options.ApiCredentials =
        new LBankCredentials("API_KEY", "API_SECRET");
});

var orders = await socket.SpotApi.SubscribeToOrderUpdatesAsync(
    listenKey: null,
    update => Console.WriteLine(update.Data.Status));

if (!orders.Success)
{
    Console.WriteLine(orders.Error);
    return;
}

await socket.UnsubscribeAsync(orders.Data);
```

Passing `null` lets the authenticated client acquire and maintain a listen key.

## Shared API V2

Use a narrow capability interface when the operation is known at compile time:

```csharp
using CryptoExchange.Net.SharedApis;

using var client = new LBankRestClient();
IGetTickerRest ticker = client.SpotApi.SharedApi;
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USDT");

var result = await ticker.GetTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

Inject `ILBankSharedApiClient` when a service needs multiple LBank Shared API surfaces. It exposes `SpotRest`, `SpotSocket`. When the operation or transport is selected dynamically, use a typed capability descriptor:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is null)
    return;

Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

For WebSocket workflows, use the narrow subscription interface exposed by the applicable socket surface, such as `ISubscribeTickerSocket` or `ISubscribeTradesSocket`. Prefer an aggregate property or direct capability injection when the required surface is known at compile time.

## Dependency Injection and Local Book

```csharp
services.AddLBank(options =>
{
    options.ApiCredentials =
        new LBankCredentials("API_KEY", "API_SECRET");
});
```

Resolve `ILBankOrderBookFactory`, then:

```csharp
var book = factory.CreateSpot(
    "eth_usdt",
    options => options.Limit = 50);

var start = await book.StartAsync();
if (!start.Success)
{
    Console.WriteLine(start.Error);
    return;
}

var snapshot = book.Book;
await book.StopAsync();
```

## Transient Retry

```csharp
var result = await client.SpotApi.ExchangeData
    .GetOrderBookAsync("eth_usdt", limit: 20);

if (!result.Success)
{
    if (result.Error?.IsTransient == true)
    {
        // Retry with bounded exponential backoff, jitter, and cancellation.
    }

    Console.WriteLine(
        $"{result.Error?.Code}: {result.Error?.Message}");
}
```
