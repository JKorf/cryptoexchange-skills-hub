# BitMEX.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../BitMEX.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using BitMEX.Net;
using BitMEX.Net.Clients;
using BitMEX.Net.Enums;
using BitMEX.Net.ExtensionMethods;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Active Symbols

```csharp
var client = new BitMEXRestClient();
var symbols = await client.ExchangeApi.ExchangeData.GetActiveSymbolsAsync();

if (!symbols.Success)
{
    Console.WriteLine($"Failed to get active symbols: {symbols.Error}");
    return;
}

foreach (var symbol in symbols.Data.Take(5))
{
    Console.WriteLine($"{symbol.Symbol}: {symbol.SymbolType}, last={symbol.LastPrice}, mark={symbol.MarkPrice}");
}
```

Prefer active-symbol discovery over guessing names.

## Order Book And Klines

```csharp
const string symbol = "XBTUSD";

var orderBook = await client.ExchangeApi.ExchangeData.GetOrderBookAsync(symbol, 25);
if (!orderBook.Success)
{
    Console.WriteLine($"Failed to get order book: {orderBook.Error}");
    return;
}

var candles = await client.ExchangeApi.ExchangeData.GetKlinesAsync(
    symbol,
    BinPeriod.OneMinute,
    limit: 5,
    reverse: true);
```

## BitMEX Quantities

BitMEX symbols and balances can use exchange-specific units. Load conversion metadata before using conversion extension methods.

```csharp
var conversionInfo = await BitMEXUtils.UpdateSymbolInfoAsync();
if (!conversionInfo.Success)
{
    Console.WriteLine($"Could not load BitMEX conversion metadata: {conversionInfo.Error}");
    return;
}

var balances = await client.ExchangeApi.Account.GetBalancesAsync();
if (!balances.Success)
{
    Console.WriteLine($"Failed to get balances: {balances.Error}");
    return;
}

foreach (var balance in balances.Data.Take(5))
{
    var sharedQuantity = balance.Quantity.ToSharedAssetQuantity(balance.Currency);
    Console.WriteLine($"{balance.Currency}: bitmex={balance.Quantity}, shared={sharedQuantity}");
}
```

## Authenticated Account Info

```csharp
var client = new BitMEXRestClient(options =>
{
    options.ApiCredentials = new BitMEXCredentials("API_KEY", "API_SECRET");
});

var account = await client.ExchangeApi.Account.GetAccountInfoAsync();
if (!account.Success)
{
    Console.WriteLine($"Failed to get account info: {account.Error}");
    return;
}

Console.WriteLine($"Account id: {account.Data.Id}");
```

## Limit Order

Prefer post-only limit examples because they make execution intent explicit.

```csharp
var order = await client.ExchangeApi.Trading.PlaceOrderAsync(
    symbol: "XBTUSD",
    orderSide: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 100,
    price: 1m,
    timeInForce: TimeInForce.GoodTillCancel,
    executionInstruction: ExecutionInstruction.PostOnly,
    clientOrderId: $"ai-example-{DateTimeOffset.UtcNow.ToUnixTimeMilliseconds()}");

if (!order.Success)
{
    Console.WriteLine($"Failed to place order: {order.Error}");
    return;
}

Console.WriteLine($"Placed order {order.Data.OrderId}");
```

## Order Lookup And Cleanup

```csharp
var orders = await client.ExchangeApi.Trading.GetOrdersAsync(
    symbol: "XBTUSD",
    filter: new Dictionary<string, object>
    {
        ["orderID"] = order.Data.OrderId
    });

if (!orders.Success)
{
    Console.WriteLine($"Failed to get orders: {orders.Error}");
    return;
}

var cancel = await client.ExchangeApi.Trading.CancelOrderAsync(orderId: order.Data.OrderId);
if (!cancel.Success)
{
    Console.WriteLine($"Failed to cancel order: {cancel.Error}");
    return;
}
```

## Positions And Leverage

Changing leverage affects account risk settings. Only generate leverage writes when explicitly requested.

```csharp
var positions = await client.ExchangeApi.Trading.GetPositionsAsync(
    filter: new Dictionary<string, object> { ["symbol"] = "XBTUSD" });

if (positions.Success)
{
    foreach (var position in positions.Data)
        Console.WriteLine($"{position.Symbol}: quantity={position.CurrentQuantity}, leverage={position.Leverage}, open={position.IsOpen}");
}

var leverage = await client.ExchangeApi.Trading.SetCrossMarginLeverageAsync("XBTUSD", leverage: 2m);
```

## Public Websocket Subscription

```csharp
var socket = new BitMEXSocketClient();

var tradeSub = await socket.ExchangeApi.SubscribeToTradeUpdatesAsync(
    "XBTUSD",
    update =>
    {
        foreach (var trade in update.Data)
            Console.WriteLine($"{trade.Symbol}: {trade.Quantity} @ {trade.Price}");
    });

if (!tradeSub.Success)
{
    Console.WriteLine($"Failed to subscribe trades: {tradeSub.Error}");
    return;
}

await socket.UnsubscribeAsync(tradeSub.Data);
```

Keep handlers fast; push expensive processing to a queue or channel.

## Kline Websocket Subscription

```csharp
var klineSub = await socket.ExchangeApi.SubscribeToKlineUpdatesAsync(
    "XBTUSD",
    BinPeriod.OneMinute,
    update =>
    {
        foreach (var candle in update.Data)
            Console.WriteLine($"{candle.Symbol} 1m close: {candle.ClosePrice}");
    });

if (!klineSub.Success)
{
    Console.WriteLine($"Failed to subscribe klines: {klineSub.Error}");
    return;
}

await socket.UnsubscribeAsync(klineSub.Data);
```

## Authenticated Websocket Updates

Private streams require credentials on the socket client.

```csharp
var socket = new BitMEXSocketClient(options =>
{
    options.ApiCredentials = new BitMEXCredentials("API_KEY", "API_SECRET");
});

var orderSub = await socket.ExchangeApi.SubscribeToOrderUpdatesAsync(update =>
{
    foreach (var order in update.Data)
        Console.WriteLine($"Order update: {order.OrderId} {order.Status}");
});

if (!orderSub.Success)
{
    Console.WriteLine($"Failed to subscribe order updates: {orderSub.Error}");
    return;
}

await socket.UnsubscribeAsync(orderSub.Data);
```

## SharedApis REST

Use this when the user wants exchange-agnostic code. Call discovery before relying on optional features.

```csharp
using BitMEX.Net.Clients;
using CryptoExchange.Net.SharedApis;

var shared = new BitMEXRestClient().ExchangeApi.SharedClient;
var info = shared.Discover();

Console.WriteLine($"Exchange: {shared.Exchange}");
Console.WriteLine($"Trading modes: {string.Join(", ", shared.SupportedTradingModes)}");
Console.WriteLine($"{info.Exchange} {info.TypeName}");

var assets = await shared.GetAssetsAsync(new GetAssetsRequest());
if (!assets.Success)
{
    Console.WriteLine($"Shared asset request failed: {assets.Error}");
    return;
}
```

## SharedApis Websocket

```csharp
var socket = new BitMEXSocketClient();
ITradeSocketClient trades = socket.ExchangeApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.PerpetualInverse, "XBT", "USD");

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

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared non-I/O helpers can return `ExchangeCallResult<T>`. Errors are normally returned in `Error`, not thrown.

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

var book = await WithRetry(() => client.ExchangeApi.ExchangeData.GetOrderBookAsync("XBTUSD", 25));
```

Common handling guidance:

- Retry only transient network, server, or rate-limit errors.
- Do not retry validation errors, permission errors, insufficient balance, or rejected orders blindly.
- Use `ct: cancellationToken` on REST methods when the calling application has cancellation.
- Use `options.RequestTimeout = TimeSpan.FromSeconds(10)` for per-request timeout configuration.

## Dependency Injection

```csharp
using BitMEX.Net;

services.AddBitMEX(options =>
{
    options.Rest.ApiCredentials = new BitMEXCredentials("API_KEY", "API_SECRET");
    options.Socket.ApiCredentials = new BitMEXCredentials("API_KEY", "API_SECRET");
});
```

Inject `IBitMEXRestClient` and `IBitMEXSocketClient`, or follow the consuming project's existing interface pattern. `AddBitMEX` registers shared REST and socket interfaces from `ExchangeApi.SharedClient`.

## Local Examples

When available, read:

- `../BitMEX.Net/Examples/ai-friendly/01-market-and-account.cs`
- `../BitMEX.Net/Examples/ai-friendly/02-trading-and-positions.cs`
- `../BitMEX.Net/Examples/ai-friendly/03-websocket.cs`
- `../BitMEX.Net/Examples/ai-friendly/04-shared-client.cs`
- `../BitMEX.Net/Examples/ai-friendly/05-error-handling.cs`

