# CoinEx.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../CoinEx.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using CoinEx.Net;
using CoinEx.Net.Clients;
using CoinEx.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Spot Ticker

```csharp
var client = new CoinExRestClient();

var tickers = await client.SpotApiV2.ExchangeData.GetTickersAsync(new[] { "BTCUSDT" });
if (!tickers.Success)
{
    Console.WriteLine($"Failed to get ticker: {tickers.Error}");
    return;
}

var ticker = tickers.Data.First();
Console.WriteLine($"{ticker.Symbol} last price: {ticker.LastPrice}");
```

## CoinEx Symbols

Use CoinEx-native symbols for exchange-specific clients:

- Spot: `BTCUSDT`, `ETHUSDT`
- Futures: `BTCUSDT`, `ETHUSDT`

Use `SharedSymbol` values only for `CryptoExchange.Net.SharedApis`:

```csharp
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");
```

## Order Book And Klines

```csharp
var orderBook = await client.SpotApiV2.ExchangeData.GetOrderBookAsync(
    "BTCUSDT",
    limit: 20,
    mergeLevel: "0");

var candles = await client.SpotApiV2.ExchangeData.GetKlinesAsync(
    "BTCUSDT",
    KlineInterval.OneMinute,
    limit: 5);
```

## Authenticated Balances

```csharp
var client = new CoinExRestClient(options =>
{
    options.ApiCredentials = new CoinExCredentials("API_KEY", "API_SECRET");
});

var balances = await client.SpotApiV2.Account.GetBalancesAsync();
if (!balances.Success)
{
    Console.WriteLine($"Failed to get balances: {balances.Error}");
    return;
}

foreach (var balance in balances.Data.Where(b => b.Total > 0))
    Console.WriteLine($"{balance.Asset}: {balance.Available} available, {balance.Frozen} frozen");
```

## Spot Limit Order

```csharp
var order = await client.SpotApiV2.Trading.PlaceOrderAsync(
    symbol: "BTCUSDT",
    accountType: AccountType.Spot,
    side: OrderSide.Buy,
    type: OrderTypeV2.Limit,
    quantity: 0.001m,
    price: 50000m);

if (!order.Success)
{
    Console.WriteLine($"Failed to place order: {order.Error}");
    return;
}

var cancel = await client.SpotApiV2.Trading.CancelOrderAsync(
    "BTCUSDT",
    AccountType.Spot,
    order.Data.Id);
```

## Precision-Aware Order Inputs

```csharp
var symbols = await client.SpotApiV2.ExchangeData.GetSymbolsAsync();
if (!symbols.Success)
    return;

var btcusdt = symbols.Data.FirstOrDefault(s => s.Name == "BTCUSDT");
if (btcusdt == null)
    return;

var validQuantity = Math.Round(0.00123456m, btcusdt.QuantityPrecision, MidpointRounding.ToZero);
var validPrice = Math.Round(50000.123456m, btcusdt.PricePrecision, MidpointRounding.ToZero);
```

## Futures Leverage And Position

Futures position and leverage calls affect account risk. Only generate write operations when explicitly requested.

```csharp
var leverage = await client.FuturesApi.Account.SetLeverageAsync("ETHUSDT", MarginMode.Cross, 5);
if (!leverage.Success)
{
    Console.WriteLine($"Failed to set leverage: {leverage.Error}");
    return;
}

var positions = await client.FuturesApi.Trading.GetPositionsAsync("ETHUSDT");
if (!positions.Success)
{
    Console.WriteLine($"Failed to get positions: {positions.Error}");
    return;
}
```

## Futures Close Position

```csharp
var position = positions.Data.Items.FirstOrDefault(p => p.OpenInterest != 0);
if (position != null)
{
    var closeOrder = await client.FuturesApi.Trading.ClosePositionAsync(
        symbol: "ETHUSDT",
        orderType: OrderTypeV2.Market,
        quantity: position.CloseAvailable);
}
```

## Public Websocket Subscriptions

```csharp
var socket = new CoinExSocketClient();

var tickerSub = await socket.SpotApiV2.SubscribeToTickerUpdatesAsync(
    new[] { "BTCUSDT" },
    update =>
    {
        foreach (var ticker in update.Data)
            Console.WriteLine($"{ticker.Symbol}: {ticker.LastPrice}");
    });

if (!tickerSub.Success)
{
    Console.WriteLine($"Failed to subscribe ticker: {tickerSub.Error}");
    return;
}

await socket.UnsubscribeAsync(tickerSub.Data);
```

Keep handlers fast; push expensive processing to a queue or channel.

## Order Book Websocket Subscription

```csharp
var bookSub = await socket.SpotApiV2.SubscribeToOrderBookUpdatesAsync(
    symbol: "BTCUSDT",
    depth: 20,
    mergeLevel: "0",
    fullBookUpdates: true,
    onMessage: update =>
    {
        var bestBid = update.Data.Data.Bids.FirstOrDefault();
        var bestAsk = update.Data.Data.Asks.FirstOrDefault();
        Console.WriteLine($"Book {update.Data.Symbol}: bid={bestBid?.Price} ask={bestAsk?.Price}");
    });

if (!bookSub.Success)
{
    Console.WriteLine($"Failed to subscribe order book: {bookSub.Error}");
    return;
}

await socket.UnsubscribeAsync(bookSub.Data);
```

## Authenticated Websocket Updates

Private streams require credentials on the socket client.

```csharp
var socket = new CoinExSocketClient(options =>
{
    options.ApiCredentials = new CoinExCredentials("API_KEY", "API_SECRET");
});

var orderSub = await socket.SpotApiV2.SubscribeToOrderUpdatesAsync(update =>
{
    var order = update.Data.Order;
    Console.WriteLine($"Order {order.Id} {order.Symbol}: filled {order.QuantityFilled}/{order.Quantity}");
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
using CoinEx.Net.Clients;
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient spot = new CoinExRestClient().SpotApiV2.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");

var result = await spot.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (result.Success)
    Console.WriteLine($"[{spot.Exchange}] {result.Data.Symbol}: {result.Data.LastPrice}");
```

## SharedApis Websocket

```csharp
var socket = new CoinExSocketClient();
ITickerSocketClient tickers = socket.SpotApiV2.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");

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

Shared socket interfaces do not expose `UnsubscribeAsync`; keep the concrete socket client and call `socket.UnsubscribeAsync(sub.Data)`.

## Error Handling And Retry

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared non-I/O helpers can return `ExchangeCallResult<T>`. Batch operations can return `HttpResult<CallResult<T>[]>` or `HttpResult<CoinExBatchResult<T>[]>`; inspect each item after the outer result succeeds.

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

        if (last.Error?.IsTransient != true)
            return last;

        await Task.Delay(TimeSpan.FromMilliseconds(250 * Math.Pow(2, attempt)));
    }

    return last;
}

var ticker = await WithRetry(() => client.SpotApiV2.ExchangeData.GetTickersAsync(new[] { "BTCUSDT" }));
```

Common handling guidance:

- Retry only transient network, server, or rate-limit errors.
- Do not retry validation errors, permission errors, insufficient balance, or rejected orders blindly.
- Use `ct: cancellationToken` on REST methods when the calling application has cancellation.
- Use `options.RequestTimeout = TimeSpan.FromSeconds(10)` for per-request timeout configuration.

## Dependency Injection

```csharp
using CoinEx.Net;

services.AddCoinEx(options =>
{
    options.ApiCredentials = new CoinExCredentials("API_KEY", "API_SECRET");
});
```

Inject `ICoinExRestClient` and `ICoinExSocketClient`, or follow the consuming project's existing interface pattern. `AddCoinEx` registers shared REST and socket interfaces from `SpotApiV2.SharedClient` and `FuturesApi.SharedClient`.

## Local Examples

When available, read:

- `../CoinEx.Net/Examples/ai-friendly/01-spot-quickstart.cs`
- `../CoinEx.Net/Examples/ai-friendly/02-futures.cs`
- `../CoinEx.Net/Examples/ai-friendly/03-websocket.cs`
- `../CoinEx.Net/Examples/ai-friendly/04-multi-exchange.cs`
- `../CoinEx.Net/Examples/ai-friendly/05-error-handling.cs`
