# BitMart.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../BitMart.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using BitMart.Net;
using BitMart.Net.Clients;
using BitMart.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Spot Ticker

```csharp
var client = new BitMartRestClient();
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTC_USDT");

if (!ticker.Success)
{
    Console.WriteLine($"Failed to get ticker: {ticker.Error}");
    return;
}

Console.WriteLine($"{ticker.Data.Symbol} last price: {ticker.Data.LastPrice}");
Console.WriteLine($"{ticker.Data.Symbol} 24h base volume: {ticker.Data.Volume24h}");
```

## BitMart Symbols

Use BitMart-native symbols for exchange-specific clients:

- Spot: `BTC_USDT`, `ETH_USDT`
- USD futures: `BTCUSDT`, `ETHUSDT`

Use unformatted `SharedSymbol` values only for `CryptoExchange.Net.SharedApis`:

```csharp
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");
```

## Authenticated Spot Balances

```csharp
var client = new BitMartRestClient(options =>
{
    options.ApiCredentials = new BitMartCredentials("API_KEY", "API_SECRET", "MEMO_OR_PASS");
});

var balances = await client.SpotApi.Account.GetSpotBalancesAsync();
if (!balances.Success)
{
    Console.WriteLine($"Failed to get balances: {balances.Error}");
    return;
}

foreach (var balance in balances.Data.Where(x => x.Asset is "BTC" or "USDT"))
{
    Console.WriteLine($"{balance.Asset}: available={balance.Available}, frozen={balance.Frozen}");
}
```

## Spot Limit Order

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTC_USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    price: 1m);

if (!order.Success)
{
    Console.WriteLine($"Failed to place order: {order.Error}");
    return;
}

Console.WriteLine($"Placed spot order {order.Data.OrderId}");
```

## Spot Order Lookup And Cleanup

```csharp
var openOrders = await client.SpotApi.Trading.GetOpenOrdersAsync("BTC_USDT");
if (!openOrders.Success)
{
    Console.WriteLine($"Failed to get open orders: {openOrders.Error}");
    return;
}

var cancel = await client.SpotApi.Trading.CancelOrderAsync(
    symbol: "BTC_USDT",
    orderId: order.Data.OrderId);

if (!cancel.Success)
{
    Console.WriteLine($"Failed to cancel order: {cancel.Error}");
    return;
}
```

## Spot Margin

Spot margin borrow and repay endpoints live under `client.SpotApi.Margin`. Margin order placement lives under `client.SpotApi.Trading.PlaceMarginOrderAsync`.

```csharp
var marginAccounts = await client.SpotApi.Account.GetIsolatedMarginAccountsAsync("BTC_USDT");
if (!marginAccounts.Success)
{
    Console.WriteLine($"Failed to get margin accounts: {marginAccounts.Error}");
    return;
}
```

Generate borrow, repay, and margin-order code only when the user explicitly asks for margin behavior.

## USD Futures Market Data And Account

```csharp
const string symbol = "BTCUSDT";

var funding = await client.UsdFuturesApi.ExchangeData.GetCurrentFundingRateAsync(symbol);
if (!funding.Success)
{
    Console.WriteLine($"Failed to get funding rate: {funding.Error}");
    return;
}

Console.WriteLine($"{funding.Data.Symbol} expected funding rate: {funding.Data.ExpectedFundingRate}");

var balances = await client.UsdFuturesApi.Account.GetBalancesAsync();
if (!balances.Success)
{
    Console.WriteLine($"Failed to get futures balances: {balances.Error}");
    return;
}

var positions = await client.UsdFuturesApi.Trading.GetPositionsAsync(symbol);
```

## USD Futures Limit Order

USD futures examples can create real exposure. Only generate futures trading code when the user asks for it.

```csharp
var order = await client.UsdFuturesApi.Trading.PlaceOrderAsync(
    symbol: "BTCUSDT",
    side: FuturesSide.BuyOpenLong,
    type: FuturesOrderType.Limit,
    quantity: 1,
    price: 1m,
    leverage: 5m,
    marginType: MarginType.CrossMargin,
    orderMode: OrderMode.GoodTilCancel);

if (!order.Success)
{
    Console.WriteLine($"Failed to place futures order: {order.Error}");
    return;
}

var cancel = await client.UsdFuturesApi.Trading.CancelOrderAsync(
    symbol: "BTCUSDT",
    orderId: order.Data.OrderId.ToString());
```

Common futures variations:

- Set leverage: `client.UsdFuturesApi.Account.SetLeverageAsync(...)`
- Set position mode: `client.UsdFuturesApi.Account.SetPositionModeAsync(...)`
- Trigger order: `client.UsdFuturesApi.Trading.PlaceTriggerOrderAsync(...)`
- TP/SL order: `client.UsdFuturesApi.Trading.PlaceTpSlOrderAsync(...)`
- Cancel all after: `client.UsdFuturesApi.Trading.CancelAllAfterAsync(...)`

## Public Websocket Subscription

```csharp
var socket = new BitMartSocketClient();

var tickerSub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "BTC_USDT",
    update => Console.WriteLine($"BTC_USDT ticker: {update.Data.LastPrice}"));

if (!tickerSub.Success)
{
    Console.WriteLine($"Failed to subscribe: {tickerSub.Error}");
    return;
}

await socket.UnsubscribeAsync(tickerSub.Data);
```

Keep handlers fast; push expensive processing to a queue or channel.

## USD Futures Kline Websocket Subscription

```csharp
var klineSub = await socket.UsdFuturesApi.SubscribeToKlineUpdatesAsync(
    "ETHUSDT",
    FuturesStreamKlineInterval.OneMinute,
    update =>
    {
        var candle = update.Data.Klines.FirstOrDefault();
        if (candle != null)
            Console.WriteLine($"ETHUSDT 1m close: {candle.ClosePrice}");
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
var socket = new BitMartSocketClient(options =>
{
    options.ApiCredentials = new BitMartCredentials("API_KEY", "API_SECRET", "MEMO_OR_PASS");
});

var balanceSub = await socket.SpotApi.SubscribeToBalanceUpdatesAsync(
    update =>
    {
        foreach (var balance in update.Data.Balances)
            Console.WriteLine($"{balance.Asset}: available={balance.Available}, frozen={balance.Frozen}");
    });

if (!balanceSub.Success)
{
    Console.WriteLine($"Failed to subscribe balance updates: {balanceSub.Error}");
    return;
}

await socket.UnsubscribeAsync(balanceSub.Data);
```

## SharedApis REST

Use this when the user wants exchange-agnostic code.

```csharp
using BitMart.Net.Clients;
using CryptoExchange.Net.SharedApis;

var client = new BitMartRestClient();
ISpotTickerRestClient tickers = client.SpotApi.SharedClient;

var discovery = client.SpotApi.SharedClient.Discover();
Console.WriteLine($"{discovery.Exchange} {discovery.TypeName}");

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
var socket = new BitMartSocketClient();
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

var ticker = await WithRetry(() => client.SpotApi.ExchangeData.GetTickerAsync("BTC_USDT"));
```

Common handling guidance:

- Retry only transient network, server, or rate-limit errors.
- Do not retry validation errors, permission errors, insufficient balance, or rejected orders blindly.
- Use `ct: cancellationToken` on REST methods when the calling application has cancellation.
- Use `options.RequestTimeout = TimeSpan.FromSeconds(10)` for per-request timeout configuration.

## Dependency Injection

```csharp
using BitMart.Net;

services.AddBitMart(options =>
{
    options.Rest.ApiCredentials = new BitMartCredentials("API_KEY", "API_SECRET", "MEMO_OR_PASS");
    options.Socket.ApiCredentials = new BitMartCredentials("API_KEY", "API_SECRET", "MEMO_OR_PASS");
});
```

Inject `IBitMartRestClient` and `IBitMartSocketClient`, or follow the consuming project's existing interface pattern. `AddBitMart` registers shared REST and socket interfaces for both `SpotApi.SharedClient` and `UsdFuturesApi.SharedClient`.

## Local Examples

When available, read:

- `../BitMart.Net/Examples/ai-friendly/01-spot-quickstart.cs`
- `../BitMart.Net/Examples/ai-friendly/02-usd-futures.cs`
- `../BitMart.Net/Examples/ai-friendly/03-websocket.cs`
- `../BitMart.Net/Examples/ai-friendly/04-multi-exchange.cs`
- `../BitMart.Net/Examples/ai-friendly/05-error-handling.cs`
