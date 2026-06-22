# BingX.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../BingX.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using BingX.Net;
using BingX.Net.Clients;
using BingX.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Spot Ticker

```csharp
var client = new BingXRestClient();
var tickers = await client.SpotApi.ExchangeData.GetTickersAsync("BTC-USDT");

if (!tickers.Success)
{
    Console.WriteLine($"Failed to get ticker: {tickers.Error}");
    return;
}

var btcTicker = tickers.Data.Single();
Console.WriteLine($"BTC/USDT last price: {btcTicker.LastPrice}");
Console.WriteLine($"24h volume: {btcTicker.Volume}");
```

## Authenticated Client

```csharp
var client = new BingXRestClient(options =>
{
    options.ApiCredentials = new BingXCredentials("API_KEY", "API_SECRET");
});
```

Use environment variables or secret storage in real applications.

## Authenticated Spot Balances

```csharp
var balances = await client.SpotApi.Account.GetBalancesAsync();
if (!balances.Success)
{
    Console.WriteLine($"Failed to get balances: {balances.Error}");
    return;
}

foreach (var balance in balances.Data.Where(x => x.Total > 0))
{
    Console.WriteLine($"{balance.Asset}: {balance.Free} free, {balance.Locked} locked");
}
```

## Spot Limit Order

Prefer limit orders in examples because they make price intent explicit.

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTC-USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    price: 50000m,
    timeInForce: TimeInForce.GoodTillCanceled);

if (!order.Success)
{
    Console.WriteLine($"Failed to place order: {order.Error}");
    return;
}

Console.WriteLine($"Placed order {order.Data.OrderId}");
```

Let the library generate `clientOrderId` unless the user explicitly needs an external correlation id.

## Spot Order Lookup And Cleanup

```csharp
var status = await client.SpotApi.Trading.GetOrderAsync("BTC-USDT", order.Data.OrderId);
if (!status.Success)
{
    Console.WriteLine($"Failed to get order status: {status.Error}");
    return;
}

Console.WriteLine($"Order status: {status.Data.Status}, filled: {status.Data.QuantityFilled}");

var cancel = await client.SpotApi.Trading.CancelOrderAsync("BTC-USDT", order.Data.OrderId);
if (!cancel.Success)
{
    Console.WriteLine($"Failed to cancel order: {cancel.Error}");
    return;
}
```

## Perpetual Futures

Perpetual futures examples can change leverage and create real exposure. Only generate futures trading code when the user asks for it.

```csharp
const string symbol = "ETH-USDT";

var leverage = await client.PerpetualFuturesApi.Account.SetLeverageAsync(symbol, PositionSide.Long, 5);
if (!leverage.Success)
{
    Console.WriteLine($"Failed to set leverage: {leverage.Error}");
    return;
}

var openOrder = await client.PerpetualFuturesApi.Trading.PlaceOrderAsync(
    symbol: symbol,
    side: OrderSide.Buy,
    type: FuturesOrderType.Market,
    positionSide: PositionSide.Long,
    quantity: 0.01m);

if (!openOrder.Success)
{
    Console.WriteLine($"Failed to open position: {openOrder.Error}");
    return;
}
```

Common perpetual futures variations:

- Limit order: use `FuturesOrderType.Limit` with `price` and `timeInForce`
- Stop-market: use `FuturesOrderType.StopMarket` with `stopPrice`
- One-way mode: use `PositionSide.Both`
- Hedge mode: use `PositionSide.Long` or `PositionSide.Short`
- Margin mode: `client.PerpetualFuturesApi.Account.SetMarginModeAsync(symbol, MarginMode.Isolated)`
- Safe close: place the opposite side with `reduceOnly: true`

## Perpetual Position Lookup And Reduce-Only Close

```csharp
var positions = await client.PerpetualFuturesApi.Trading.GetPositionsAsync("ETH-USDT");
if (!positions.Success)
{
    Console.WriteLine($"Failed to get positions: {positions.Error}");
    return;
}

var position = positions.Data.FirstOrDefault(x => x.Size != 0);
if (position == null)
{
    Console.WriteLine("No open position found.");
    return;
}

var closeOrder = await client.PerpetualFuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETH-USDT",
    side: position.Size > 0 ? OrderSide.Sell : OrderSide.Buy,
    type: FuturesOrderType.Market,
    positionSide: PositionSide.Long,
    quantity: Math.Abs(position.Size),
    reduceOnly: true);
```

Adjust `positionSide` for one-way mode or short hedge-mode positions.

## Public Websocket Subscription

```csharp
var socket = new BingXSocketClient();

var tickerSub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "BTC-USDT",
    update =>
    {
        Console.WriteLine($"BTC-USDT: {update.Data.LastPrice}");
    });

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
var klineSub = await socket.SpotApi.SubscribeToKlineUpdatesAsync(
    "ETH-USDT",
    KlineInterval.OneMinute,
    update =>
    {
        var kline = update.Data.Kline;
        Console.WriteLine($"ETH 1m close: {kline.ClosePrice}");
    });

if (!klineSub.Success)
{
    Console.WriteLine($"Failed to subscribe klines: {klineSub.Error}");
    return;
}
```

## Perpetual Websocket Variations

```csharp
var futuresTicker = await socket.PerpetualFuturesApi.SubscribeToTickerUpdatesAsync(
    "ETH-USDT",
    update => Console.WriteLine(update.Data.LastPrice));

var markPrice = await socket.PerpetualFuturesApi.SubscribeToMarkPriceUpdatesAsync(
    "ETH-USDT",
    update => Console.WriteLine(update.Data.MarkPrice));
```

Confirm exact overloads in the target package before using less common streams.

## SharedApis REST

Use this when the user wants exchange-agnostic code.

```csharp
using BingX.Net.Clients;
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new BingXRestClient().SpotApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");

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
var socket = new BingXSocketClient();
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

## Error Handling And Retry

REST methods return `HttpResult<T>`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Errors are normally returned in `Error`, not thrown.

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

var ticker = await WithRetry(() => client.SpotApi.ExchangeData.GetTickersAsync("BTC-USDT"));
```

## Dependency Injection

```csharp
using BingX.Net;

services.AddBingX(options =>
{
    options.ApiCredentials = new BingXCredentials("API_KEY", "API_SECRET");
});
```

Inject `IBingXRestClient` and `IBingXSocketClient`, or follow the consuming project's existing interface pattern.

## Local Examples

When available, read:

- `../BingX.Net/Examples/ai-friendly/01-spot-quickstart.cs`
- `../BingX.Net/Examples/ai-friendly/02-perpetual-futures.cs`
- `../BingX.Net/Examples/ai-friendly/03-websocket.cs`
- `../BingX.Net/Examples/ai-friendly/04-multi-exchange.cs`
- `../BingX.Net/Examples/ai-friendly/05-error-handling.cs`
