# Lighter.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../Lighter.Net/Examples/ai-friendly/` for complete programs.

## Base Imports

```csharp
using Lighter.Net;
using Lighter.Net.Clients;
using Lighter.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Spot Ticker

```csharp
var client = new LighterRestClient();
var details = await client.ExchangeApi.ExchangeData.GetSymbolDetailsAsync("ETH/USDC");

if (!details.Success)
{
    Console.WriteLine($"Failed to get symbol details: {details.Error}");
    return;
}

var spot = details.Data.SpotSymbols.FirstOrDefault();
Console.WriteLine($"ETH/USDC last price: {spot?.LastPrice}");
```

## Public Perpetual Order Book

```csharp
var book = await client.ExchangeApi.ExchangeData.GetOrderBookAsync("ETH", limit: 50);
if (!book.Success)
{
    Console.WriteLine($"Failed to get order book: {book.Error}");
    return;
}

foreach (var bid in book.Data.Bids.Take(5))
    Console.WriteLine($"Bid {bid.Price}: {bid.Quantity}");
```

## Klines

```csharp
var klines = await client.ExchangeApi.ExchangeData.GetKlinesAsync(
    symbol: "ETH/USDC",
    interval: KlineInterval.OneMinute,
    startTime: DateTime.UtcNow.AddHours(-1),
    endTime: DateTime.UtcNow);

if (!klines.Success)
{
    Console.WriteLine($"Failed to get klines: {klines.Error}");
    return;
}
```

## Authenticated Account Read

```csharp
var client = new LighterRestClient(options =>
{
    options.ApiCredentials = new LighterCredentials(EthKey.FromPrivateKey("PRIVATE_KEY"), 123, 5, "API_SECRET");
});

var account = await client.ExchangeApi.Account.GetAccountsAsync();
if (!account.Success)
{
    Console.WriteLine($"Failed to get account: {account.Error}");
    return;
}
```

## Limit Order

Prefer limit orders in examples because they make price intent explicit. Lighter order placement is signer-backed and needs credentials.

```csharp
var order = await client.ExchangeApi.Trading.PlaceOrderAsync(
    symbol: "ETH",
    side: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 0.1m,
    price: 2000m,
    timeInForce: TimeInForce.GoodTillTime);

if (!order.Success)
{
    Console.WriteLine($"Failed to place order: {order.Error}");
    return;
}

Console.WriteLine($"Transaction hash: {order.Data.TransactionHash}");
```

Let the library set `nonce` unless the user explicitly needs a manually controlled nonce. Pass `clientOrderIndex` only when the caller needs external order correlation.

## Order Lookup And Cleanup

```csharp
var openOrders = await client.ExchangeApi.Trading.GetOpenOrdersAsync(symbol: "ETH");
if (!openOrders.Success)
{
    Console.WriteLine($"Failed to get open orders: {openOrders.Error}");
    return;
}

var first = openOrders.Data.Orders.FirstOrDefault();
if (first != null)
{
    var cancel = await client.ExchangeApi.Trading.CancelOrderAsync("ETH", first.OrderIndex);
    if (!cancel.Success)
        Console.WriteLine($"Failed to cancel order: {cancel.Error}");
}
```

## Leverage And Margin

Leverage and margin calls change real account risk. Only generate them when the user asks for this behavior.

```csharp
var leverage = await client.ExchangeApi.Account.SetLeverageAsync(
    symbol: "ETH",
    leverage: 5,
    marginMode: MarginMode.IsolatedMargin);

if (!leverage.Success)
{
    Console.WriteLine($"Failed to set leverage: {leverage.Error}");
    return;
}
```

## Public Websocket Subscription

```csharp
var socket = new LighterSocketClient();

var tickerSub = await socket.ExchangeApi.ExchangeData.SubscribeToSpotTickerUpdatesAsync(
    "ETH/USDC",
    update => Console.WriteLine($"ETH/USDC: {update.Data.Ticker.LastPrice}"));

if (!tickerSub.Success)
{
    Console.WriteLine($"Failed to subscribe: {tickerSub.Error}");
    return;
}

// On shutdown:
await socket.UnsubscribeAsync(tickerSub.Data);
```

Keep handlers fast; push expensive processing to a queue or channel.

## Kline Websocket Subscription

```csharp
var klineSub = await socket.ExchangeApi.ExchangeData.SubscribeToKlineUpdatesAsync(
    "ETH/USDC",
    KlineInterval.OneMinute,
    update =>
    {
        var latest = update.Data.Klines.LastOrDefault();
        Console.WriteLine($"ETH 1m close: {latest?.ClosePrice}");
    });

if (!klineSub.Success)
{
    Console.WriteLine($"Failed to subscribe klines: {klineSub.Error}");
    return;
}
```

## Authenticated Websocket Subscriptions

```csharp
var socket = new LighterSocketClient(options =>
{
    options.ApiCredentials = new LighterCredentials(EthKey.FromPrivateKey("PRIVATE_KEY"), 123, 5, "API_SECRET");
});

var orderSub = await socket.ExchangeApi.Trading.SubscribeToOrderUpdatesAsync(
    accountIndex: null,
    update =>
    {
        foreach (var order in update.Data.Orders.Values.SelectMany(x => x))
            Console.WriteLine($"Order update: {order.OrderIndex} {order.Status}");
    });

if (!orderSub.Success)
{
    Console.WriteLine($"Failed to subscribe order updates: {orderSub.Error}");
    return;
}

await socket.UnsubscribeAsync(orderSub.Data);
```

## Websocket Order Request

Socket request methods return `QueryResult<T>`, not `WebSocketResult<UpdateSubscription>`.

```csharp
var order = await socket.ExchangeApi.Trading.PlaceOrderAsync(
    symbol: "ETH",
    side: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 0.1m,
    price: 2000m,
    timeInForce: TimeInForce.GoodTillTime);

if (!order.Success)
{
    Console.WriteLine($"Socket order failed: {order.Error}");
    return;
}
```

## SharedApis REST

Use this when the user wants exchange-agnostic code.

```csharp
using CryptoExchange.Net.SharedApis;

var lighterRest = new LighterRestClient();
ISpotTickerRestClient tickers = lighterRest.ExchangeApi.SharedClient;
var capabilities = lighterRest.ExchangeApi.SharedClient.Discover();
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USDC");

var result = await tickers.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine($"[{tickers.Exchange}] Failed: {result.Error}");
    return;
}

Console.WriteLine($"[{tickers.Exchange}] {result.Data.Symbol}: {result.Data.LastPrice}");
```

## SharedApis Websocket

```csharp
var socket = new LighterSocketClient();
ITickerSocketClient tickers = socket.ExchangeApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USDC");

var sub = await tickers.SubscribeToTickerUpdatesAsync(
    new SubscribeTickerRequest(symbol),
    update => Console.WriteLine($"[{tickers.Exchange}] {update.Data.Symbol}: {update.Data.LastPrice}"));

if (!sub.Success)
{
    Console.WriteLine($"Subscribe failed: {sub.Error}");
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

## Local Order Book

```csharp
using CryptoExchange.Net.SharedApis;
using Lighter.Net.Interfaces;
using Microsoft.Extensions.DependencyInjection;

var services = new ServiceCollection();
services.AddLighter();
var provider = services.BuildServiceProvider();

var bookFactory = provider.GetRequiredService<ILighterOrderBookFactory>();
var book = bookFactory.Create(new SharedSymbol(TradingMode.Spot, "ETH", "USDC"));

var start = await book.StartAsync();
if (!start.Success)
{
    Console.WriteLine(start.Error);
    return;
}

Console.WriteLine($"Best bid: {book.Book.bids.FirstOrDefault()?.Price}");
```

## Trade Tracker

```csharp
var trackerFactory = provider.GetRequiredService<ILighterTrackerFactory>();
var tracker = trackerFactory.CreateTradeTracker(
    new SharedSymbol(TradingMode.PerpetualLinear, "ETH", "USDC"),
    period: TimeSpan.FromMinutes(10));

var start = await tracker.StartAsync();
if (!start.Success)
{
    Console.WriteLine(start.Error);
    return;
}

var stats = tracker.GetStats(DateTime.UtcNow.AddMinutes(-5));
Console.WriteLine($"Recent volume: {stats.Volume}");
```

## Error Handling And Retry

REST methods return `HttpResult<T>`. Subscription methods return `WebSocketResult<UpdateSubscription>`. Socket request methods return `QueryResult<T>`. Errors are normally returned in `Error`, not thrown.

```csharp
async Task<HttpResult<T>> WithRetry<T>(
    Func<Task<HttpResult<T>>> call,
    int maxAttempts = 3)
{
    HttpResult<T> last = default!;

    for (var attempt = 1; attempt <= maxAttempts; attempt++)
    {
        last = await call();
        if (last.Success)
            return last;

        if (last.Error == null || !last.Error.IsTransient)
            return last;

        await Task.Delay(TimeSpan.FromMilliseconds(250 * Math.Pow(2, attempt)));
    }

    return last;
}

var details = await WithRetry(() => client.ExchangeApi.ExchangeData.GetSymbolDetailsAsync("ETH/USDC"));
```

## Dependency Injection

```csharp
using Lighter.Net;
using Lighter.Net.Interfaces.Clients;

services.AddLighter(options =>
{
    options.Rest.ApiCredentials = new LighterCredentials(EthKey.FromPrivateKey("PRIVATE_KEY"), 123, 5, "API_SECRET");
    options.Socket.ApiCredentials = new LighterCredentials(EthKey.FromPrivateKey("PRIVATE_KEY"), 123, 5, "API_SECRET");
});
```

Inject `ILighterRestClient`, `ILighterSocketClient`, `ILighterOrderBookFactory`, `ILighterTrackerFactory`, or `ILighterUserClientProvider` depending on the workflow.

## Integrator Fee

Lighter.Net uses Lighter's optional integrator-code mechanism by default. Disable it only when explicitly requested:

```csharp
var client = new LighterRestClient(options =>
{
    options.ApiCredentials = new LighterCredentials(EthKey.FromPrivateKey("PRIVATE_KEY"), 123, 5, "API_SECRET");
    options.IntegratorFeePercentage = 0m;
});
```

Use `null` or `0m` to disable the optional fee. Socket options expose the same property for socket transaction requests.

## Local Examples

When available, read:

- `../Lighter.Net/Examples/ai-friendly/01-quickstart.cs`
- `../Lighter.Net/Examples/ai-friendly/02-trading.cs`
- `../Lighter.Net/Examples/ai-friendly/03-websocket.cs`
- `../Lighter.Net/Examples/ai-friendly/04-multi-exchange.cs`
- `../Lighter.Net/Examples/ai-friendly/05-error-handling.cs`
- `../Lighter.Net/Examples/Lighter.Examples.OrderBook/Program.cs`
- `../Lighter.Net/Examples/Lighter.Examples.Tracker/Program.cs`
