# CoinW.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../CoinW.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using CoinW.Net;
using CoinW.Net.Clients;
using CoinW.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Spot Ticker

CoinW spot exposes all tickers in one call. Filter the result array for a specific symbol.

```csharp
var client = new CoinWRestClient();
const string symbol = "BTC_USDT";

var tickers = await client.SpotApi.ExchangeData.GetTickersAsync();
if (!tickers.Success)
{
    Console.WriteLine($"Failed to get tickers: {tickers.Error}");
    return;
}

var ticker = tickers.Data.SingleOrDefault(x => x.Symbol == symbol);
if (ticker == null)
{
    Console.WriteLine($"{symbol} was not returned by CoinW.");
    return;
}

Console.WriteLine($"{symbol} last price: {ticker.LastPrice}");
```

## CoinW Symbols

Use CoinW-native symbols for exchange-specific clients:

- Spot: `BTC_USDT`, `ETH_USDT`
- Futures: `BTC`, `ETH`

Use `SharedSymbol` values only for `CryptoExchange.Net.SharedApis`:

```csharp
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");
```

Use `CoinWExchange.FormatSymbol` when native code receives base/quote input:

```csharp
var spotSymbol = CoinWExchange.FormatSymbol("BTC", "USDT", TradingMode.Spot);
var futuresSymbol = CoinWExchange.FormatSymbol("BTC", "USDT", TradingMode.PerpetualLinear);
```

## Order Book, Symbols, And Klines

```csharp
var symbols = await client.SpotApi.ExchangeData.GetSymbolsAsync();
var orderBook = await client.SpotApi.ExchangeData.GetOrderBookAsync("BTC_USDT", limit: 20);
var candles = await client.SpotApi.ExchangeData.GetKlinesAsync(
    "BTC_USDT",
    KlineInterval.OneMinute,
    startTime: DateTime.UtcNow.AddHours(-1));
```

## Authenticated Spot Balances

```csharp
var client = new CoinWRestClient(options =>
{
    options.ApiCredentials = new CoinWCredentials("API_KEY", "API_SECRET");
});

var balances = await client.SpotApi.Account.GetBalancesDetailsAsync();
if (!balances.Success)
{
    Console.WriteLine($"Failed to get balances: {balances.Error}");
    return;
}

foreach (var balance in balances.Data.Where(x => x.Value.Available + x.Value.OnOrders > 0))
    Console.WriteLine($"{balance.Key}: {balance.Value.Available} available, {balance.Value.OnOrders} on orders");
```

## Spot Limit Order

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTC_USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    price: 50000m);

if (!order.Success)
{
    Console.WriteLine($"Failed to place order: {order.Error}");
    return;
}

Console.WriteLine($"Placed order {order.Data.OrderId}");

var cancel = await client.SpotApi.Trading.CancelOrderAsync(order.Data.OrderId);
```

For market buys where the exchange expects quote sizing, use `quoteQuantity` instead of `quantity` when that matches the intended behavior.

## Spot Account Actions

Withdrawals and transfers move funds. Only generate write calls when the user explicitly asks for them.

```csharp
var deposits = await client.SpotApi.Account.GetDepositWithdrawalHistoryAsync("USDT");
var addresses = await client.SpotApi.Account.GetDepositAddressesAsync("USDT", "TRC20");

var transfer = await client.SpotApi.Account.TransferAsync(
    AccountType.Funding,
    AccountType.Spot,
    "USDT",
    10m);
```

## Futures Ticker And Margin Mode

CoinW futures methods commonly use a base/instrument symbol such as `ETH`.

```csharp
var ticker = await client.FuturesApi.ExchangeData.GetTickerAsync("ETH");
if (!ticker.Success)
{
    Console.WriteLine($"Failed to get futures ticker: {ticker.Error}");
    return;
}

var marginMode = await client.FuturesApi.Account.SetMarginModeAsync(
    MarginType.IsolatedMargin,
    PositionCombineType.Split);
```

## Futures Market Order And Position

Futures position and margin calls affect account risk. Only generate write operations when explicitly requested.

```csharp
var openOrder = await client.FuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETH",
    side: PositionSide.Long,
    orderType: FuturesOrderType.Market,
    quantity: 1m,
    leverage: 5,
    quantityUnit: QuantityUnit.Contracts,
    marginType: MarginType.IsolatedMargin);

if (!openOrder.Success)
{
    Console.WriteLine($"Failed to open position: {openOrder.Error}");
    return;
}

var positions = await client.FuturesApi.Trading.GetPositionsAsync("ETH");
if (!positions.Success)
{
    Console.WriteLine($"Failed to get positions: {positions.Error}");
    return;
}
```

## Futures Close Position

Use the position id returned by position queries. `PlaceOrderAsync` is for opening positions; `ClosePositionAsync` is for closing an existing position.

```csharp
var position = positions.Data.FirstOrDefault(p => p.Status == OpenStatus.Open && p.TotalQuantity != 0);
if (position != null)
{
    var close = await client.FuturesApi.Trading.ClosePositionAsync(
        position.Id,
        FuturesOrderType.Market);
}
```

## Public Websocket Subscriptions

```csharp
var socket = new CoinWSocketClient();

var tickerSub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "BTC_USDT",
    update => Console.WriteLine($"{update.Data.Symbol}: {update.Data.LastPrice}"));

if (!tickerSub.Success)
{
    Console.WriteLine($"Failed to subscribe ticker: {tickerSub.Error}");
    return;
}

await socket.UnsubscribeAsync(tickerSub.Data);
```

Keep handlers fast; push expensive processing to a queue or channel.

## Futures Websocket Subscriptions

```csharp
var futuresSub = await socket.FuturesApi.SubscribeToTickerUpdatesAsync(
    "BTC",
    update => Console.WriteLine($"BTC futures: {update.Data.LastPrice}"));

if (!futuresSub.Success)
{
    Console.WriteLine($"Failed to subscribe futures ticker: {futuresSub.Error}");
    return;
}

await socket.UnsubscribeAsync(futuresSub.Data);
```

## Authenticated Websocket Updates

Private streams require credentials on the socket client.

```csharp
var socket = new CoinWSocketClient(options =>
{
    options.ApiCredentials = new CoinWCredentials("API_KEY", "API_SECRET");
});

var orderSub = await socket.SpotApi.SubscribeToOrderUpdatesAsync(update =>
{
    Console.WriteLine($"Spot order {update.Data.OrderId} {update.Data.Symbol}: {update.Data.EventType}");
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
using CoinW.Net.Clients;
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient spot = new CoinWRestClient().SpotApi.SharedClient;
var info = spot.Discover();
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");

var result = await spot.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (result.Success)
    Console.WriteLine($"[{spot.Exchange}] {result.Data.Symbol}: {result.Data.LastPrice}");
```

## SharedApis Websocket

```csharp
var socket = new CoinWSocketClient();
ITickerSocketClient tickers = socket.SpotApi.SharedClient;
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

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared non-I/O helpers can return `ExchangeCallResult<T>`. Batch futures orders return `HttpResult<CallResult<CoinWBatchResult>[]>`; inspect each item after the outer result succeeds.

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

var tickers = await WithRetry(() => client.SpotApi.ExchangeData.GetTickersAsync());
```

Common handling guidance:

- Retry only transient network, server, or rate-limit errors.
- Do not retry validation errors, permission errors, insufficient balance, or rejected orders blindly.
- Use `ct: cancellationToken` on REST methods when the calling application has cancellation.
- Use `options.RequestTimeout = TimeSpan.FromSeconds(10)` for per-request timeout configuration.

## Dependency Injection

```csharp
using CoinW.Net;

services.AddCoinW(options =>
{
    options.ApiCredentials = new CoinWCredentials("API_KEY", "API_SECRET");
});
```

Inject `ICoinWRestClient` and `ICoinWSocketClient`, or follow the consuming project's existing interface pattern. `AddCoinW` registers shared REST and socket interfaces from `SpotApi.SharedClient` and `FuturesApi.SharedClient`.

## Local Examples

When available, read:

- `../CoinW.Net/Examples/ai-friendly/01-spot-quickstart.cs`
- `../CoinW.Net/Examples/ai-friendly/02-futures.cs`
- `../CoinW.Net/Examples/ai-friendly/03-websocket.cs`
- `../CoinW.Net/Examples/ai-friendly/04-multi-exchange.cs`
- `../CoinW.Net/Examples/ai-friendly/05-error-handling.cs`
