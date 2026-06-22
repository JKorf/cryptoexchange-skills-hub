# Coinbase.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../Coinbase.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using Coinbase.Net;
using Coinbase.Net.Clients;
using Coinbase.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Advanced Trade Product

```csharp
var client = new CoinbaseRestClient();
const string symbol = "ETH-USD";

var product = await client.AdvancedTradeApi.ExchangeData.GetSymbolAsync(symbol);
if (!product.Success)
{
    Console.WriteLine($"Product request failed: {product.Error}");
    return;
}

Console.WriteLine($"{product.Data.Symbol} last price: {product.Data.LastPrice}");
```

## Coinbase Product Ids

Use Coinbase-native product ids for exchange-specific clients:

- Spot: `ETH-USD`, `BTC-USD`
- Coinbase futures/perpetual product ids should be inspected from `GetSymbolsAsync` before use.

Use `SharedSymbol` values only for `CryptoExchange.Net.SharedApis`:

```csharp
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USD");
```

## Klines And Order Book

```csharp
var candles = await client.AdvancedTradeApi.ExchangeData.GetKlinesAsync(
    "ETH-USD",
    KlineInterval.OneMinute,
    limit: 5);

if (candles.Success)
{
    foreach (var candle in candles.Data)
        Console.WriteLine($"{candle.OpenTime:u} close={candle.ClosePrice} volume={candle.Volume}");
}

var orderBook = await client.AdvancedTradeApi.ExchangeData.GetOrderBookAsync("ETH-USD", limit: 50);
```

## Coinbase Exchange Market Data

```csharp
var exchangeProducts = await client.ExchangeApi.ExchangeData.GetSymbolsAsync();
if (exchangeProducts.Success)
    Console.WriteLine($"Exchange API products returned: {exchangeProducts.Data.Length}");
```

Use `ExchangeApi` for Coinbase Exchange market data. Prefer `AdvancedTradeApi` for Coinbase Advanced Trade account and trading workflows.

## Authenticated Accounts

```csharp
var keyName = Environment.GetEnvironmentVariable("COINBASE_API_KEY_NAME");
var privateKey = Environment.GetEnvironmentVariable("COINBASE_API_PRIVATE_KEY")?.Replace("\\n", "\n");
if (string.IsNullOrWhiteSpace(keyName) || string.IsNullOrWhiteSpace(privateKey))
{
    Console.WriteLine("Skipping private account request; COINBASE_API_KEY_NAME and COINBASE_API_PRIVATE_KEY are not set.");
    return;
}

var client = new CoinbaseRestClient(options =>
{
    options.ApiCredentials = new CoinbaseCredentials(keyName, privateKey);
});

var accounts = await client.AdvancedTradeApi.Account.GetAccountsAsync(limit: 100);
if (!accounts.Success)
{
    Console.WriteLine($"Account request failed: {accounts.Error}");
    return;
}

foreach (var account in accounts.Data.Accounts.Take(10))
    Console.WriteLine($"{account.Asset}: available={account.AvailableBalance.Value}, hold={account.HoldBalance.Value}");
```

## Dry-Run Trading Pattern

```csharp
var openOrders = await client.AdvancedTradeApi.Trading.GetOrdersAsync(
    symbols: new[] { "ETH-USD" },
    orderStatus: new[] { OrderStatus.Open },
    limit: 20);

if (openOrders.Success)
    Console.WriteLine($"Open orders returned: {openOrders.Data.Length}");
else
    Console.WriteLine($"Open-order request failed: {openOrders.Error}");

var fills = await client.AdvancedTradeApi.Trading.GetUserTradesAsync(
    symbols: new[] { "ETH-USD" },
    limit: 20);
```

## Live Limit Order And Cleanup

Only generate live order placement when explicitly requested. The local ai-friendly example gates this behind `COINBASE_EXAMPLE_PLACE_ORDER=true`.

```csharp
var order = await client.AdvancedTradeApi.Trading.PlaceOrderAsync(
    "ETH-USD",
    OrderSide.Buy,
    NewOrderType.Limit,
    quantity: 0.01m,
    price: 1000m);

if (!order.Success)
{
    Console.WriteLine($"Order request failed: {order.Error}");
    return;
}

if (!order.Data.Success)
{
    Console.WriteLine($"Order rejected: {order.Data.ErrorResponse.Message}");
    return;
}

var orderId = order.Data.SuccessResponse.OrderId;

var cancel = await client.AdvancedTradeApi.Trading.CancelOrderAsync(orderId);
Console.WriteLine(cancel.Success
    ? $"Cancel success={cancel.Data.Success}, order={cancel.Data.OrderId}, error={cancel.Data.ErrorMessage}"
    : $"Cancel request failed: {cancel.Error}");
```

## Public Websocket Subscriptions

```csharp
var socket = new CoinbaseSocketClient();

var tickerSub = await socket.AdvancedTradeApi.SubscribeToTickerUpdatesAsync(
    "ETH-USD",
    update => Console.WriteLine($"{update.Data.Symbol}: {update.Data.LastPrice}"));

if (!tickerSub.Success)
{
    Console.WriteLine($"Ticker subscription failed: {tickerSub.Error}");
    return;
}

var bookSub = await socket.AdvancedTradeApi.SubscribeToOrderBookUpdatesAsync(
    "ETH-USD",
    update => Console.WriteLine($"Book update: bids={update.Data.Bids.Length}, asks={update.Data.Asks.Length}"));

if (!bookSub.Success)
{
    Console.WriteLine($"Order book subscription failed: {bookSub.Error}");
    await socket.UnsubscribeAsync(tickerSub.Data);
    return;
}

await socket.UnsubscribeAsync(tickerSub.Data);
await socket.UnsubscribeAsync(bookSub.Data);
```

Keep handlers fast; push expensive processing to a queue or channel.

## Authenticated Websocket Updates

Private streams require credentials. Use a separate authenticated socket client when examples also include public subscriptions.

```csharp
var socket = new CoinbaseSocketClient(options =>
{
    options.ApiCredentials = new CoinbaseCredentials(keyName, privateKey);
});

var userSub = await socket.AdvancedTradeApi.SubscribeToUserUpdatesAsync(
    update => Console.WriteLine($"User update orders={update.Data.Orders.Length}"));

if (!userSub.Success)
{
    Console.WriteLine($"User subscription failed: {userSub.Error}");
    return;
}

await socket.UnsubscribeAsync(userSub.Data);
```

## SharedApis REST

Use this when the user wants exchange-agnostic code. Call discovery before relying on optional features.

```csharp
using Coinbase.Net.Clients;
using CryptoExchange.Net.SharedApis;

var shared = new CoinbaseRestClient().AdvancedTradeApi.SharedClient;
var info = shared.Discover();

Console.WriteLine($"Shared exchange: {shared.Exchange}");
Console.WriteLine($"Supported trading modes: {string.Join(", ", shared.SupportedTradingModes)}");
Console.WriteLine($"{info.Exchange} {info.TypeName}");
```

## SharedApis Websocket

```csharp
var socket = new CoinbaseSocketClient();
ITradeSocketClient trades = socket.AdvancedTradeApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USD");

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

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared non-I/O helpers can return `ExchangeCallResult<T>`.

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

var product = await WithRetry(() => client.AdvancedTradeApi.ExchangeData.GetSymbolAsync("ETH-USD"));
```

Common handling guidance:

- Retry only transient network, server, or rate-limit errors.
- Do not retry validation errors, permission errors, insufficient balance, or rejected orders blindly.
- Use `ct: cancellationToken` on REST methods when the calling application has cancellation.
- Use `options.RequestTimeout = TimeSpan.FromSeconds(10)` for per-request timeout configuration.

## Dependency Injection

```csharp
using Coinbase.Net;

services.AddCoinbase(options =>
{
    options.ApiCredentials = new CoinbaseCredentials("API_KEY_NAME", "PRIVATE_KEY");
});
```

Inject `ICoinbaseRestClient` and `ICoinbaseSocketClient`, or follow the consuming project's existing interface pattern. `AddCoinbase` registers shared REST and socket interfaces from `AdvancedTradeApi.SharedClient`.

## Local Examples

When available, read:

- `../Coinbase.Net/Examples/ai-friendly/01-advanced-trade-market-and-account.cs`
- `../Coinbase.Net/Examples/ai-friendly/02-advanced-trade-trading.cs`
- `../Coinbase.Net/Examples/ai-friendly/03-websocket.cs`
- `../Coinbase.Net/Examples/ai-friendly/04-shared-client.cs`
- `../Coinbase.Net/Examples/ai-friendly/05-error-handling.cs`
