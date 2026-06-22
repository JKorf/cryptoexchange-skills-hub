# Aster.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../Aster.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using Aster.Net;
using Aster.Net.Clients;
using Aster.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Spot V3 Ticker

```csharp
var client = new AsterRestClient();
var ticker = await client.SpotV3Api.ExchangeData.GetTickerAsync("BTCUSDT");

if (!ticker.Success)
{
    Console.WriteLine($"Failed to get ticker: {ticker.Error}");
    return;
}

Console.WriteLine($"BTC/USDT last price: {ticker.Data.LastPrice}");
Console.WriteLine($"24h volume: {ticker.Data.Volume}");
```

## Authenticated V3 Client

```csharp
var client = new AsterRestClient(options =>
{
    options.ApiCredentials = new AsterCredentials()
        .WithV3("USER_PRIVATE_KEY", "SIGNER_PRIVATE_KEY");
});
```

Use environment variables or secret storage in real applications.

## Authenticated Spot Account

```csharp
var account = await client.SpotV3Api.Account.GetAccountInfoAsync();
if (!account.Success)
{
    Console.WriteLine($"Failed to get account: {account.Error}");
    return;
}

foreach (var balance in account.Data.Balances.Where(x => x.Free + x.Locked > 0))
{
    Console.WriteLine($"{balance.Asset}: {balance.Free} free, {balance.Locked} locked");
}
```

## Spot Limit Order

Prefer limit orders in examples because they make price intent explicit.

```csharp
var order = await client.SpotV3Api.Trading.PlaceOrderAsync(
    symbol: "BTCUSDT",
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

Console.WriteLine($"Placed order {order.Data.Id}");
```

Let the library generate `clientOrderId` unless the user explicitly needs an external correlation id.

## Spot Order Lookup And Cleanup

```csharp
var status = await client.SpotV3Api.Trading.GetOrderAsync("BTCUSDT", order.Data.Id);
if (!status.Success)
{
    Console.WriteLine($"Failed to get order status: {status.Error}");
    return;
}

Console.WriteLine($"Order status: {status.Data.Status}, filled: {status.Data.QuantityFilled}");

var cancel = await client.SpotV3Api.Trading.CancelOrderAsync("BTCUSDT", order.Data.Id);
if (!cancel.Success)
{
    Console.WriteLine($"Failed to cancel order: {cancel.Error}");
    return;
}
```

## Futures V3

Futures examples can change leverage and create real exposure. Only generate futures trading code when the user asks for it.

```csharp
const string symbol = "ETHUSDT";

var leverage = await client.FuturesV3Api.Account.SetLeverageAsync(symbol, 5);
if (!leverage.Success)
{
    Console.WriteLine($"Failed to set leverage: {leverage.Error}");
    return;
}

var openOrder = await client.FuturesV3Api.Trading.PlaceOrderAsync(
    symbol: symbol,
    side: OrderSide.Buy,
    type: OrderType.Market,
    quantity: 0.01m);

if (!openOrder.Success)
{
    Console.WriteLine($"Failed to open position: {openOrder.Error}");
    return;
}
```

Common futures variations:

- Limit order: use `OrderType.Limit` with `price` and `timeInForce`
- Stop-market: use `OrderType.StopMarket` with `stopPrice`
- Take-profit market: use `OrderType.TakeProfitMarket` with `stopPrice`
- Hedge mode: pass `positionSide: PositionSide.Long` or `PositionSide.Short`
- Margin type: `client.FuturesV3Api.Account.SetMarginTypeAsync(symbol, MarginType.Isolated)`
- Safe close: place the opposite side with `reduceOnly: true`

## Futures Position Lookup And Reduce-Only Close

```csharp
var positions = await client.FuturesV3Api.Trading.GetPositionsAsync("ETHUSDT");
if (!positions.Success)
{
    Console.WriteLine($"Failed to get positions: {positions.Error}");
    return;
}

var position = positions.Data.FirstOrDefault(x => x.PositionAmount != 0);
if (position == null)
{
    Console.WriteLine("No open position found.");
    return;
}

var closeOrder = await client.FuturesV3Api.Trading.PlaceOrderAsync(
    symbol: "ETHUSDT",
    side: position.PositionAmount > 0 ? OrderSide.Sell : OrderSide.Buy,
    type: OrderType.Market,
    quantity: Math.Abs(position.PositionAmount),
    reduceOnly: true);
```

## Public Websocket Subscription

```csharp
var socket = new AsterSocketClient();

var tickerSub = await socket.SpotV3Api.SubscribeToTickerUpdatesAsync(
    "BTCUSDT",
    update =>
    {
        Console.WriteLine($"BTCUSDT: {update.Data.LastPrice}");
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
var klineSub = await socket.SpotV3Api.SubscribeToKlineUpdatesAsync(
    "ETHUSDT",
    KlineInterval.OneMinute,
    update =>
    {
        var kline = update.Data.Data;
        if (kline.Final)
        {
            Console.WriteLine($"ETH 1m close: {kline.ClosePrice}");
        }
    });

if (!klineSub.Success)
{
    Console.WriteLine($"Failed to subscribe klines: {klineSub.Error}");
    return;
}
```

## Futures Websocket Variations

```csharp
var futuresTicker = await socket.FuturesV3Api.SubscribeToTickerUpdatesAsync(
    "ETHUSDT",
    update => Console.WriteLine(update.Data.LastPrice));

var markPrice = await socket.FuturesV3Api.SubscribeToMarkPriceUpdatesAsync(
    "ETHUSDT",
    1000,
    update => Console.WriteLine(update.Data.MarkPrice));
```

Confirm exact overloads in the target package before using less common streams.

## SharedApis REST

Use this when the user wants exchange-agnostic code.

```csharp
using Aster.Net.Clients;
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new AsterRestClient().SpotV3Api.SharedClient;
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
var socket = new AsterSocketClient();
ITickerSocketClient tickers = socket.SpotV3Api.SharedClient;
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

var ticker = await WithRetry(() => client.SpotV3Api.ExchangeData.GetTickerAsync("BTCUSDT"));
```

## Dependency Injection

```csharp
using Aster.Net;

services.AddAster(options =>
{
    options.ApiCredentials = new AsterCredentials()
        .WithV3("USER_PRIVATE_KEY", "SIGNER_PRIVATE_KEY");
});
```

Inject `IAsterRestClient` and `IAsterSocketClient`, or follow the consuming project's existing interface pattern.

## Local Examples

When available, read:

- `../Aster.Net/Examples/ai-friendly/01-spot-quickstart.cs`
- `../Aster.Net/Examples/ai-friendly/02-futures.cs`
- `../Aster.Net/Examples/ai-friendly/03-websocket.cs`
- `../Aster.Net/Examples/ai-friendly/04-multi-exchange.cs`
- `../Aster.Net/Examples/ai-friendly/05-error-handling.cs`
