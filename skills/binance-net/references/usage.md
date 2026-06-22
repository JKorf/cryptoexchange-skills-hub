# Binance.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../Binance.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using Binance.Net;
using Binance.Net.Clients;
using Binance.Net.Enums;
using Binance.Net.Objects;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Spot Ticker

```csharp
var client = new BinanceRestClient();
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTCUSDT");

if (!ticker.Success)
{
    Console.WriteLine($"Failed to get ticker: {ticker.Error}");
    return;
}

Console.WriteLine($"BTC/USDT last price: {ticker.Data.LastPrice}");
Console.WriteLine($"24h volume: {ticker.Data.Volume}");
```

## Exchange Info And Filters

Use exchange info before production order placement. Binance rejects invalid quantity/price with filter errors such as `LOT_SIZE`, `PRICE_FILTER`, or `MIN_NOTIONAL`.

```csharp
var exchangeInfo = await client.SpotApi.ExchangeData.GetExchangeInfoAsync("BTCUSDT");
if (!exchangeInfo.Success || exchangeInfo.Data.Symbols.Length == 0)
{
    Console.WriteLine($"Failed to get exchange info: {exchangeInfo.Error}");
    return;
}

var symbolInfo = exchangeInfo.Data.Symbols.First();
var lotSize = symbolInfo.LotSizeFilter;

decimal rawQuantity = 0.00123456m;
decimal quantity = lotSize == null
    ? rawQuantity
    : CryptoExchange.Net.ExchangeHelpers.AdjustValueStep(
        lotSize.MinQuantity,
        lotSize.MaxQuantity,
        lotSize.StepSize,
        RoundingType.Down,
        rawQuantity);
```

## Authenticated Spot Account

```csharp
var client = new BinanceRestClient(options =>
{
    options.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
});

var account = await client.SpotApi.Account.GetAccountInfoAsync();
if (!account.Success)
{
    Console.WriteLine($"Failed to get account: {account.Error}");
    return;
}

foreach (var balance in account.Data.Balances.Where(x => x.Total > 0))
{
    Console.WriteLine($"{balance.Asset}: {balance.Available} free, {balance.Locked} locked");
}
```

## Spot Limit Order

Prefer limit orders in examples because they make price intent explicit.

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTCUSDT",
    side: OrderSide.Buy,
    type: SpotOrderType.Limit,
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
var status = await client.SpotApi.Trading.GetOrderAsync("BTCUSDT", order.Data.Id);
if (!status.Success)
{
    Console.WriteLine($"Failed to get order status: {status.Error}");
    return;
}

Console.WriteLine($"Order status: {status.Data.Status}, filled: {status.Data.QuantityFilled}");

var cancel = await client.SpotApi.Trading.CancelOrderAsync("BTCUSDT", order.Data.Id);
if (!cancel.Success)
{
    Console.WriteLine($"Failed to cancel order: {cancel.Error}");
    return;
}
```

## USD-M Futures

Futures examples can change leverage and create real exposure. Only generate futures trading code when the user asks for it.

```csharp
var client = new BinanceRestClient(options =>
{
    options.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
});

const string symbol = "ETHUSDT";

var leverage = await client.UsdFuturesApi.Account.ChangeInitialLeverageAsync(symbol, 5);
if (!leverage.Success)
{
    Console.WriteLine($"Failed to set leverage: {leverage.Error}");
    return;
}

var openOrder = await client.UsdFuturesApi.Trading.PlaceOrderAsync(
    symbol: symbol,
    side: OrderSide.Buy,
    type: FuturesOrderType.Market,
    quantity: 0.01m);

if (!openOrder.Success)
{
    Console.WriteLine($"Failed to open position: {openOrder.Error}");
    return;
}
```

Common futures variations:

- COIN-M futures: use `client.CoinFuturesApi.*`
- Limit order: use `FuturesOrderType.Limit` with `price` and `timeInForce`
- Hedge mode: pass `positionSide: PositionSide.Long` or `PositionSide.Short`
- Margin type: `client.UsdFuturesApi.Account.ChangeMarginTypeAsync(symbol, FuturesMarginType.Isolated)`
- Safe close: place the opposite side with `reduceOnly: true`

## Futures Position Lookup And Reduce-Only Close

```csharp
var positions = await client.UsdFuturesApi.Account.GetPositionInformationAsync("ETHUSDT");
if (!positions.Success)
{
    Console.WriteLine($"Failed to get positions: {positions.Error}");
    return;
}

var position = positions.Data.FirstOrDefault(x => x.Quantity != 0);
if (position == null)
{
    Console.WriteLine("No open position found.");
    return;
}

var closeOrder = await client.UsdFuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETHUSDT",
    side: position.Quantity > 0 ? OrderSide.Sell : OrderSide.Buy,
    type: FuturesOrderType.Market,
    quantity: Math.Abs(position.Quantity),
    reduceOnly: true);
```

## Public Websocket Subscription

```csharp
var socket = new BinanceSocketClient();

var tickerSub = await socket.SpotApi.ExchangeData.SubscribeToTickerUpdatesAsync(
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
var klineSub = await socket.SpotApi.ExchangeData.SubscribeToKlineUpdatesAsync(
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

## Authenticated User Data Stream

```csharp
var socket = new BinanceSocketClient(options =>
{
    options.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
});

var userSub = await socket.SpotApi.Account.SubscribeToUserDataUpdatesAsync(
    onOrderUpdateMessage: update =>
    {
        var order = update.Data;
        Console.WriteLine($"Order {order.Id} {order.Symbol}: {order.Status}");
    },
    onAccountPositionMessage: update =>
    {
        foreach (var balance in update.Data.Balances)
            Console.WriteLine($"{balance.Asset}: {balance.Available} free, {balance.Locked} locked");
    },
    onAccountBalanceUpdate: update =>
    {
        Console.WriteLine($"{update.Data.Asset} delta: {update.Data.BalanceDelta}");
    });

if (!userSub.Success)
{
    Console.WriteLine($"Failed to subscribe user data: {userSub.Error}");
    return;
}

await socket.UnsubscribeAsync(userSub.Data);
```

For futures user data streams, use `socket.UsdFuturesApi.Account.SubscribeToUserDataUpdatesAsync(...)`.

## SharedApis REST

Use this when the user wants exchange-agnostic code.

```csharp
using Binance.Net.Clients;
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new BinanceRestClient().SpotApi.SharedClient;
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
var socket = new BinanceSocketClient();
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

        if (last.Error == null || !last.Error.IsTransient)
            return last;

        await Task.Delay(TimeSpan.FromMilliseconds(250 * Math.Pow(2, attempt)));
    }

    return last;
}

var ticker = await WithRetry(() => client.SpotApi.ExchangeData.GetTickerAsync("BTCUSDT"));
```

Common Binance errors:

- `-1003`: too many requests. Transient; retry with backoff.
- `-1021`: timestamp outside `recvWindow`. Check clock sync and `AutoTimestamp`.
- `-1013`: filter failure. Use exchange info filters to adjust price/quantity.
- `-1022`: invalid signature. Fix credentials.
- `-2010`: insufficient balance or rejected order. Do not blindly retry.
- `-2011`: unknown order. May be expected if already filled/cancelled.

## Dependency Injection

```csharp
using Binance.Net;

services.AddBinance(options =>
{
    options.Rest.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
    options.Socket.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
});
```

Inject `IBinanceRestClient` and `IBinanceSocketClient`, or follow the consuming project's existing interface pattern.

## Local Examples

When available, read:

- `../Binance.Net/Examples/ai-friendly/01-spot-quickstart.cs`
- `../Binance.Net/Examples/ai-friendly/02-futures.cs`
- `../Binance.Net/Examples/ai-friendly/03-websocket.cs`
- `../Binance.Net/Examples/ai-friendly/04-multi-exchange.cs`
- `../Binance.Net/Examples/ai-friendly/05-error-handling.cs`
