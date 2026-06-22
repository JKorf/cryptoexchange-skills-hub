# Bitstamp.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../Bitstamp.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using Bitstamp.Net;
using Bitstamp.Net.Clients;
using Bitstamp.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Symbols And Ticker

```csharp
var client = new BitstampRestClient();
const string symbol = "ETH/USD";

var symbols = await client.ExchangeApi.ExchangeData.GetSymbolsAsync();
if (!symbols.Success)
{
    Console.WriteLine($"Failed to get symbols: {symbols.Error}");
    return;
}

var ticker = await client.ExchangeApi.ExchangeData.GetTickerAsync(symbol);
if (!ticker.Success)
{
    Console.WriteLine($"Failed to get ticker: {ticker.Error}");
    return;
}

Console.WriteLine($"{symbol} last={ticker.Data.LastPrice}, bid={ticker.Data.BestBidPrice}, ask={ticker.Data.BestAskPrice}");
```

## Bitstamp Symbols

Use Bitstamp-native symbols for exchange-specific clients:

- Spot: `ETH/USD`, `BTC/USD`
- Derivatives: `ETH/USD-PERP`

Use `SharedSymbol` values only for `CryptoExchange.Net.SharedApis`:

```csharp
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USD");
```

## Klines And Order Book

```csharp
var orderBook = await client.ExchangeApi.ExchangeData.GetOrderBookAsync("ETH/USD");
if (!orderBook.Success)
{
    Console.WriteLine($"Failed to get order book: {orderBook.Error}");
    return;
}

var candles = await client.ExchangeApi.ExchangeData.GetKlinesAsync(
    "ETH/USD",
    KlineInterval.OneMinute,
    limit: 5,
    excludeCurrentCandle: true);
```

## Authenticated Balances

```csharp
var client = new BitstampRestClient(options =>
{
    options.ApiCredentials = new BitstampCredentials("API_KEY", "API_SECRET");
});

var balances = await client.ExchangeApi.Account.GetAccountBalancesAsync();
if (!balances.Success)
{
    Console.WriteLine($"Failed to get balances: {balances.Error}");
    return;
}

foreach (var balance in balances.Data.Where(x => x.Asset is "ETH" or "USD"))
{
    Console.WriteLine($"{balance.Asset}: total={balance.Total}, available={balance.Available}, reserved={balance.Reserved}");
}
```

## Spot Limit Order

```csharp
var order = await client.ExchangeApi.Trading.PlaceLimitOrderAsync(
    symbol: "ETH/USD",
    side: OrderSide.Buy,
    price: 1m,
    orderType: OrderType.Limit,
    quantity: 0.01m,
    clientOrderId: $"ai-example-{DateTimeOffset.UtcNow.ToUnixTimeMilliseconds()}");

if (!order.Success)
{
    Console.WriteLine($"Failed to place order: {order.Error}");
    return;
}

Console.WriteLine($"Placed order {order.Data.Id}");
```

## Order Lookup And Cleanup

```csharp
var openOrders = await client.ExchangeApi.Trading.GetOpenOrdersAsync("ETH/USD");
if (!openOrders.Success)
{
    Console.WriteLine($"Failed to get open orders: {openOrders.Error}");
    return;
}

var cancel = await client.ExchangeApi.Trading.CancelOrderAsync(orderId: order.Data.Id);
if (!cancel.Success)
{
    Console.WriteLine($"Failed to cancel order: {cancel.Error}");
    return;
}
```

## Derivative Positions And Leverage

Derivative position and leverage calls affect account risk. Only generate write operations when explicitly requested.

```csharp
const string derivativeSymbol = "ETH/USD-PERP";

var positions = await client.ExchangeApi.Trading.GetOpenPositionsAsync(derivativeSymbol);
if (positions.Success)
{
    foreach (var position in positions.Data)
        Console.WriteLine($"{position.Symbol}: side={position.Side}, quantity={position.Quantity}, leverage={position.Leverage}");
}

var leverageSettings = await client.ExchangeApi.Account.GetLeverageSettingsAsync(
    MarginMode.Cross,
    derivativeSymbol);
```

## Public Websocket Subscription

```csharp
var socket = new BitstampSocketClient();

var tradeSub = await socket.ExchangeApi.SubscribeToTradeUpdatesAsync(
    "ETH/USD",
    update => Console.WriteLine($"{update.Data.Quantity} @ {update.Data.Price}"));

if (!tradeSub.Success)
{
    Console.WriteLine($"Failed to subscribe trades: {tradeSub.Error}");
    return;
}

await socket.UnsubscribeAsync(tradeSub.Data);
```

Keep handlers fast; push expensive processing to a queue or channel.

## Order Book Websocket Subscription

```csharp
var bookSub = await socket.ExchangeApi.SubscribeToOrderBookSnapshotUpdatesAsync(
    "ETH/USD",
    update =>
    {
        var bestBid = update.Data.Bids.FirstOrDefault();
        var bestAsk = update.Data.Asks.FirstOrDefault();
        Console.WriteLine($"ETH/USD book: bid={bestBid?.Price}, ask={bestAsk?.Price}");
    });

if (!bookSub.Success)
{
    Console.WriteLine($"Failed to subscribe book: {bookSub.Error}");
    return;
}

await socket.UnsubscribeAsync(bookSub.Data);
```

## Funding Rate Websocket Subscription

```csharp
var fundingSub = await socket.ExchangeApi.SubscribeToFundingRateUpdatesAsync(
    "ETH/USD-PERP",
    update => Console.WriteLine($"{update.Data.Symbol}: funding={update.Data.FundingRate}, mark={update.Data.MarkPrice}"));

if (!fundingSub.Success)
{
    Console.WriteLine($"Failed to subscribe funding rate: {fundingSub.Error}");
    return;
}

await socket.UnsubscribeAsync(fundingSub.Data);
```

## Authenticated Websocket Updates

Private streams require credentials on the socket client.

```csharp
var socket = new BitstampSocketClient(options =>
{
    options.ApiCredentials = new BitstampCredentials("API_KEY", "API_SECRET");
});

var orderSub = await socket.ExchangeApi.SubscribeToOrderUpdatesAsync(
    "ETH/USD",
    update => Console.WriteLine($"Order update: {update.Data.Id} {update.Data.OrderEvent}"));

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
using Bitstamp.Net.Clients;
using CryptoExchange.Net.SharedApis;

var shared = new BitstampRestClient().ExchangeApi.SharedClient;
var info = shared.Discover();

Console.WriteLine($"Exchange: {shared.Exchange}");
Console.WriteLine($"Trading modes: {string.Join(", ", shared.SupportedTradingModes)}");
Console.WriteLine($"{info.Exchange} {info.TypeName}");

var symbols = await shared.GetSpotSymbolsAsync(new GetSymbolsRequest());
if (!symbols.Success)
{
    Console.WriteLine($"Shared spot symbol request failed: {symbols.Error}");
    return;
}
```

## SharedApis Websocket

Bitstamp shared websocket support is spot-only.

```csharp
var socket = new BitstampSocketClient();
ITradeSocketClient trades = socket.ExchangeApi.SharedClient;
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

var book = await WithRetry(() => client.ExchangeApi.ExchangeData.GetOrderBookAsync("ETH/USD"));
```

Common handling guidance:

- Retry only transient network, server, or rate-limit errors.
- Do not retry validation errors, permission errors, insufficient balance, or rejected orders blindly.
- Use `ct: cancellationToken` on REST methods when the calling application has cancellation.
- Use `options.RequestTimeout = TimeSpan.FromSeconds(10)` for per-request timeout configuration.

## Dependency Injection

```csharp
using Bitstamp.Net;

services.AddBitstamp(options =>
{
    options.Rest.ApiCredentials = new BitstampCredentials("API_KEY", "API_SECRET");
    options.Socket.ApiCredentials = new BitstampCredentials("API_KEY", "API_SECRET");
});
```

Inject `IBitstampRestClient` and `IBitstampSocketClient`, or follow the consuming project's existing interface pattern. `AddBitstamp` registers shared REST and socket interfaces from `ExchangeApi.SharedClient`.

## Local Examples

When available, read:

- `../Bitstamp.Net/Examples/ai-friendly/01-market-and-account.cs`
- `../Bitstamp.Net/Examples/ai-friendly/02-trading-and-positions.cs`
- `../Bitstamp.Net/Examples/ai-friendly/03-websocket.cs`
- `../Bitstamp.Net/Examples/ai-friendly/04-shared-client.cs`
- `../Bitstamp.Net/Examples/ai-friendly/05-error-handling.cs`
