# Bybit.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../Bybit.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using Bybit.Net;
using Bybit.Net.Clients;
using Bybit.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public V5 Market Data

```csharp
var client = new BybitRestClient();
const string symbol = "ETHUSDT";

var spotTicker = await client.V5Api.ExchangeData.GetSpotTickersAsync(symbol);
if (!spotTicker.Success)
{
    Console.WriteLine($"Spot ticker request failed: {spotTicker.Error}");
    return;
}

var firstSpotTicker = spotTicker.Data.List.FirstOrDefault();
Console.WriteLine($"{firstSpotTicker?.Symbol} spot last price: {firstSpotTicker?.LastPrice}");
```

## Bybit Categories And Symbols

Use Bybit-native symbols for exchange-specific clients:

- Spot: `ETHUSDT`, `BTCUSDT`
- Linear derivatives: `ETHUSDT`, `BTCUSDT`
- Inverse and delivery symbols can differ by contract; inspect exchange data first.

Many V5 methods require `Category`:

```csharp
var category = Category.Linear;
```

Use `SharedSymbol` values only for `CryptoExchange.Net.SharedApis`:

```csharp
var symbol = new SharedSymbol(TradingMode.PerpetualLinear, "ETH", "USDT");
```

## Klines And Order Book

```csharp
var klines = await client.V5Api.ExchangeData.GetKlinesAsync(
    Category.Linear,
    "ETHUSDT",
    KlineInterval.OneMinute,
    limit: 5);

if (!klines.Success)
{
    Console.WriteLine($"Kline request failed: {klines.Error}");
    return;
}

var orderBook = await client.V5Api.ExchangeData.GetOrderbookAsync(
    Category.Linear,
    "ETHUSDT",
    limit: 25);
```

## Authenticated Wallet Balance

```csharp
var apiKey = Environment.GetEnvironmentVariable("BYBIT_API_KEY");
var apiSecret = Environment.GetEnvironmentVariable("BYBIT_API_SECRET");
if (string.IsNullOrWhiteSpace(apiKey) || string.IsNullOrWhiteSpace(apiSecret))
{
    Console.WriteLine("Skipping private balance request; BYBIT_API_KEY and BYBIT_API_SECRET are not set.");
    return;
}

var client = new BybitRestClient(options =>
{
    options.ApiCredentials = new BybitCredentials(apiKey, apiSecret);
});

var balances = await client.V5Api.Account.GetBalancesAsync(AccountType.Unified, "USDT");
if (!balances.Success)
{
    Console.WriteLine($"Balance request failed: {balances.Error}");
    return;
}

foreach (var account in balances.Data.List)
{
    foreach (var asset in account.Assets)
        Console.WriteLine($"{asset.Asset}: wallet={asset.WalletBalance}, free={asset.Free}");
}
```

## Dry-Run Trading Pattern

```csharp
var category = Category.Linear;
var symbol = "ETHUSDT";

var openOrders = await client.V5Api.Trading.GetOrdersAsync(
    category,
    symbol: symbol,
    limit: 20);

if (openOrders.Success)
    Console.WriteLine($"Open order count returned: {openOrders.Data.List.Length}");
else
    Console.WriteLine($"Open-order request failed: {openOrders.Error}");

var positions = await client.V5Api.Trading.GetPositionsAsync(category, symbol: symbol);
if (positions.Success)
    Console.WriteLine($"Position rows returned: {positions.Data.List.Length}");
else
    Console.WriteLine($"Position request failed: {positions.Error}");
```

## Live Limit Order And Cleanup

Only generate live order placement when explicitly requested. The local ai-friendly example gates this behind `BYBIT_EXAMPLE_PLACE_ORDER=true`.

```csharp
var order = await client.V5Api.Trading.PlaceOrderAsync(
    Category.Linear,
    "ETHUSDT",
    OrderSide.Buy,
    NewOrderType.Limit,
    quantity: 0.01m,
    price: 1000m,
    timeInForce: TimeInForce.GoodTillCanceled,
    positionIdx: PositionIdx.OneWayMode);

if (!order.Success)
{
    Console.WriteLine($"Order rejected: {order.Error}");
    return;
}

var cancel = await client.V5Api.Trading.CancelOrderAsync(
    Category.Linear,
    "ETHUSDT",
    orderId: order.Data.OrderId);
```

## Public Websocket Subscriptions

```csharp
var socket = new BybitSocketClient();

var spotTickerSub = await socket.V5SpotApi.SubscribeToTickerUpdatesAsync(
    "ETHUSDT",
    update => Console.WriteLine($"Spot {update.Data.Symbol}: {update.Data.LastPrice}"));

if (!spotTickerSub.Success)
{
    Console.WriteLine($"Spot ticker subscription failed: {spotTickerSub.Error}");
    return;
}

var linearTickerSub = await socket.V5LinearApi.SubscribeToTickerUpdatesAsync(
    "ETHUSDT",
    update => Console.WriteLine($"Linear {update.Data.Symbol}: mark={update.Data.MarkPrice}, last={update.Data.LastPrice}"));

if (!linearTickerSub.Success)
{
    Console.WriteLine($"Linear ticker subscription failed: {linearTickerSub.Error}");
    await socket.UnsubscribeAsync(spotTickerSub.Data);
    return;
}

await socket.UnsubscribeAsync(spotTickerSub.Data);
await socket.UnsubscribeAsync(linearTickerSub.Data);
```

Keep handlers fast; push expensive processing to a queue or channel.

## Authenticated Websocket Updates

Private streams require credentials. Use a separate authenticated socket client when examples also include public subscriptions.

```csharp
var socket = new BybitSocketClient(options =>
{
    options.ApiCredentials = new BybitCredentials(apiKey, apiSecret);
});

var orderSub = await socket.V5PrivateApi.SubscribeToOrderUpdatesAsync(
    update => Console.WriteLine($"Private order updates received: {update.Data.Length}"));

if (!orderSub.Success)
{
    Console.WriteLine($"Private order subscription failed: {orderSub.Error}");
    return;
}

await socket.UnsubscribeAsync(orderSub.Data);
```

## SharedApis REST

Use this when the user wants exchange-agnostic code. Call discovery before relying on optional features.

```csharp
using Bybit.Net.Clients;
using CryptoExchange.Net.SharedApis;

var shared = new BybitRestClient().V5Api.SharedClient;
var info = shared.Discover();

Console.WriteLine($"Shared exchange: {shared.Exchange}");
Console.WriteLine($"Supported trading modes: {string.Join(", ", shared.SupportedTradingModes)}");
Console.WriteLine($"{info.Exchange} {info.TypeName}");
```

## SharedApis Websocket

```csharp
var socket = new BybitSocketClient();
ITradeSocketClient trades = socket.V5LinearApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.PerpetualLinear, "ETH", "USDT");

var sub = await trades.SubscribeToTradeUpdatesAsync(
    new SubscribeTradeRequest(symbol),
    update => Console.WriteLine($"[{trades.Exchange}] trades: {update.Data.Length}"));

if (!sub.Success)
{
    Console.WriteLine($"Subscribe failed: {sub.Error}");
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Shared socket interfaces do not expose `UnsubscribeAsync`; keep the concrete socket client and call `socket.UnsubscribeAsync(sub.Data)`.

## Error Handling And Retry

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared non-I/O helpers can return `ExchangeCallResult<T>`. Batch operations can return `HttpResult<CallResult<T>[]>`; after the outer result succeeds, inspect each item.

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

var book = await WithRetry(() => client.V5Api.ExchangeData.GetOrderbookAsync(Category.Linear, "ETHUSDT", 25));
```

Common handling guidance:

- Retry only transient network, server, or rate-limit errors.
- Do not retry validation errors, permission errors, insufficient balance, or rejected orders blindly.
- Use `ct: cancellationToken` on REST methods when the calling application has cancellation.
- Use `options.RequestTimeout = TimeSpan.FromSeconds(10)` for per-request timeout configuration.

## Dependency Injection

```csharp
using Bybit.Net;

services.AddBybit(options =>
{
    options.ApiCredentials = new BybitCredentials("API_KEY", "API_SECRET");
});
```

Inject `IBybitRestClient` and `IBybitSocketClient`, or follow the consuming project's existing interface pattern. `AddBybit` registers shared REST interfaces from `V5Api.SharedClient`, and shared socket interfaces from `V5SpotApi.SharedClient`, `V5LinearApi.SharedClient`, `V5InverseApi.SharedClient`, and `V5PrivateApi.SharedClient`.

## Local Examples

When available, read:

- `../Bybit.Net/Examples/ai-friendly/01-v5-market-and-account.cs`
- `../Bybit.Net/Examples/ai-friendly/02-v5-trading.cs`
- `../Bybit.Net/Examples/ai-friendly/03-websocket.cs`
- `../Bybit.Net/Examples/ai-friendly/04-shared-client.cs`
- `../Bybit.Net/Examples/ai-friendly/05-error-handling.cs`
