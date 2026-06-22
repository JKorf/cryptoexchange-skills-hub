# BloFin.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../BloFin.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using BloFin.Net;
using BloFin.Net.Clients;
using BloFin.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Futures Symbols And Ticker

```csharp
var client = new BloFinRestClient();
const string symbol = "ETH-USDT";

var symbols = await client.FuturesApi.ExchangeData.GetSymbolsAsync(symbol);
if (!symbols.Success)
{
    Console.WriteLine($"Symbols failed: {symbols.Error}");
    return;
}

var ticker = await client.FuturesApi.ExchangeData.GetTickersAsync(symbol);
if (!ticker.Success)
{
    Console.WriteLine($"Ticker failed: {ticker.Error}");
    return;
}

var firstTicker = ticker.Data.FirstOrDefault();
Console.WriteLine($"{firstTicker?.Symbol} last={firstTicker?.LastPrice}, bid={firstTicker?.BestBidPrice}, ask={firstTicker?.BestAskPrice}");
```

## BloFin Symbols

Use BloFin-native symbols for exchange-specific clients:

- Futures: `ETH-USDT`, `BTC-USDT`

Use `SharedSymbol` values only for `CryptoExchange.Net.SharedApis`:

```csharp
var symbol = new SharedSymbol(TradingMode.PerpetualLinear, "ETH", "USDT");
```

## Klines And Order Book

```csharp
var book = await client.FuturesApi.ExchangeData.GetOrderBookAsync("ETH-USDT", depth: 20);
if (!book.Success)
{
    Console.WriteLine($"Order book failed: {book.Error}");
    return;
}

Console.WriteLine($"Book levels: bids={book.Data.Bids.Length}, asks={book.Data.Asks.Length}");

var candles = await client.FuturesApi.ExchangeData.GetKlinesAsync(
    "ETH-USDT",
    KlineInterval.OneMinute,
    limit: 5);
```

## Authenticated Account And Futures Balances

```csharp
var client = new BloFinRestClient(options =>
{
    options.ApiCredentials = new BloFinCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
});

var accountBalances = await client.AccountApi.GetAccountBalancesAsync(AccountType.Futures, "USDT");
if (accountBalances.Success)
{
    foreach (var balance in accountBalances.Data)
        Console.WriteLine($"{balance.Asset}: total={balance.TotalBalance}, available={balance.AvailableBalance}");
}

var futuresBalances = await client.FuturesApi.Account.GetBalancesAsync(ProductType.UsdtFutures);
if (futuresBalances.Success)
    Console.WriteLine($"Futures total equity: {futuresBalances.Data.TotalEquity}");
```

## Futures Limit Order

```csharp
var order = await client.FuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETH-USDT",
    side: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 1m,
    marginMode: MarginMode.Cross,
    price: 1m,
    positionSide: PositionSide.Long,
    clientOrderId: $"ai-example-{DateTimeOffset.UtcNow.ToUnixTimeMilliseconds()}");

if (!order.Success)
{
    Console.WriteLine($"Order rejected: {order.Error}");
    return;
}

Console.WriteLine($"Placed order {order.Data.OrderId}");
```

## Order Lookup And Cleanup

```csharp
var openOrders = await client.FuturesApi.Trading.GetOpenOrdersAsync("ETH-USDT");
if (openOrders.Success)
    Console.WriteLine($"Open orders on ETH-USDT: {openOrders.Data.Length}");

var cancel = await client.FuturesApi.Trading.CancelOrderAsync(orderId: order.Data.OrderId);
Console.WriteLine(cancel.Success
    ? $"Canceled order {cancel.Data.OrderId}"
    : $"Cancel failed: {cancel.Error}");
```

## Positions And Leverage

Position, leverage, margin mode, and position mode calls affect account risk. Only generate write operations when explicitly requested.

```csharp
var positions = await client.FuturesApi.Trading.GetPositionsAsync("ETH-USDT");
if (positions.Success)
{
    foreach (var position in positions.Data)
        Console.WriteLine($"{position.Symbol}: side={position.PositionSide}, size={position.PositionSize}, leverage={position.Leverage}");
}

var leverage = await client.FuturesApi.Account.GetLeverageAsync("ETH-USDT", MarginMode.Cross);
if (leverage.Success)
    Console.WriteLine($"Leverage rows: {leverage.Data.Length}");
```

## Public Websocket Subscriptions

```csharp
var socket = new BloFinSocketClient();

var tickerSub = await socket.FuturesApi.SubscribeToTickerUpdatesAsync(
    "ETH-USDT",
    update => Console.WriteLine($"{update.Data.Symbol} ticker: {update.Data.LastPrice}"));

if (!tickerSub.Success)
{
    Console.WriteLine($"Ticker subscription failed: {tickerSub.Error}");
    return;
}

await socket.UnsubscribeAsync(tickerSub.Data);
```

Keep handlers fast; push expensive processing to a queue or channel.

## Order Book Websocket Subscription

```csharp
var bookSub = await socket.FuturesApi.SubscribeToOrderBookUpdatesAsync(
    "ETH-USDT",
    5,
    update => Console.WriteLine($"Book update: bids={update.Data.Bids.Length}, asks={update.Data.Asks.Length}"));

if (!bookSub.Success)
{
    Console.WriteLine($"Order book subscription failed: {bookSub.Error}");
    return;
}

await socket.UnsubscribeAsync(bookSub.Data);
```

## Authenticated Websocket Updates

Private streams require credentials on the socket client.

```csharp
var socket = new BloFinSocketClient(options =>
{
    options.ApiCredentials = new BloFinCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
});

var orderSub = await socket.FuturesApi.SubscribeToOrderUpdatesAsync(update =>
{
    foreach (var order in update.Data)
        Console.WriteLine($"Order update: {order.OrderId} {order.Status}");
});

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
using BloFin.Net.Clients;
using CryptoExchange.Net.SharedApis;

var client = new BloFinRestClient();
var accountShared = client.AccountApi.SharedClient;
var futuresShared = client.FuturesApi.SharedClient;

var accountInfo = accountShared.Discover();
var futuresInfo = futuresShared.Discover();

Console.WriteLine($"Account shared exchange: {accountInfo.Exchange}");
Console.WriteLine($"Futures shared exchange: {futuresInfo.Exchange}");
Console.WriteLine($"Futures trading modes: {string.Join(", ", futuresInfo.SupportedTradingModes)}");
```

## SharedApis Websocket

```csharp
var socket = new BloFinSocketClient();
ITradeSocketClient trades = socket.FuturesApi.SharedClient;
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

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Batch operations can return `HttpResult<CallResult<T>[]>`; after the outer result succeeds, inspect each item.

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

var book = await WithRetry(() => client.FuturesApi.ExchangeData.GetOrderBookAsync("ETH-USDT", 20));
```

Common handling guidance:

- Retry only transient network, server, or rate-limit errors.
- Do not retry validation errors, permission errors, insufficient balance, or rejected orders blindly.
- Use `ct: cancellationToken` on REST methods when the calling application has cancellation.
- Use `options.RequestTimeout = TimeSpan.FromSeconds(10)` for per-request timeout configuration.

## Dependency Injection

```csharp
using BloFin.Net;

services.AddBloFin(options =>
{
    options.ApiCredentials = new BloFinCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
});
```

Inject `IBloFinRestClient` and `IBloFinSocketClient`, or follow the consuming project's existing interface pattern. `AddBloFin` registers shared REST interfaces from `AccountApi.SharedClient` and `FuturesApi.SharedClient`, and shared socket interfaces from `FuturesApi.SharedClient`.

## Local Examples

When available, read:

- `../BloFin.Net/Examples/ai-friendly/01-futures-market-and-account.cs`
- `../BloFin.Net/Examples/ai-friendly/02-futures-trading.cs`
- `../BloFin.Net/Examples/ai-friendly/03-websocket.cs`
- `../BloFin.Net/Examples/ai-friendly/04-shared-client.cs`
- `../BloFin.Net/Examples/ai-friendly/05-error-handling.cs`
