# Bitget.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../Bitget.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using Bitget.Net;
using Bitget.Net.Clients;
using Bitget.Net.Enums;
using Bitget.Net.Enums.V2;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Spot Ticker

```csharp
var client = new BitgetRestClient();
var ticker = await client.SpotApiV2.ExchangeData.GetTickersAsync("BTCUSDT");

if (!ticker.Success)
{
    Console.WriteLine($"Failed to get ticker: {ticker.Error}");
    return;
}

var btc = ticker.Data.Single();
Console.WriteLine($"{btc.Symbol} last price: {btc.LastPrice}");
Console.WriteLine($"{btc.Symbol} 24h base volume: {btc.Volume}");
```

## Bitget Symbols And Product Types

Use compact symbols for exchange-specific clients:

- Spot: `BTCUSDT`, `ETHUSDT`, `BGBUSDT`
- Futures: `BTCUSDT`, `ETHUSDT`

Futures calls generally use:

```csharp
var productType = BitgetProductTypeV2.UsdtFutures;
const string symbol = "BTCUSDT";
const string marginAsset = "USDT";
```

Use unprefixed `SharedSymbol` values only for `CryptoExchange.Net.SharedApis`:

```csharp
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");
```

## Authenticated Spot Balances

```csharp
var client = new BitgetRestClient(options =>
{
    options.ApiCredentials = new BitgetCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
});

var balances = await client.SpotApiV2.Account.GetSpotBalancesAsync();
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
var order = await client.SpotApiV2.Trading.PlaceOrderAsync(
    symbol: "BTCUSDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    timeInForce: TimeInForce.GoodTillCanceled,
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
var openOrders = await client.SpotApiV2.Trading.GetOpenOrdersAsync("BTCUSDT");
if (!openOrders.Success)
{
    Console.WriteLine($"Failed to get open orders: {openOrders.Error}");
    return;
}

var cancel = await client.SpotApiV2.Trading.CancelOrderAsync(
    symbol: "BTCUSDT",
    orderId: order.Data.OrderId);

if (!cancel.Success)
{
    Console.WriteLine($"Failed to cancel order: {cancel.Error}");
    return;
}
```

## Futures Market Data And Account

```csharp
var productType = BitgetProductTypeV2.UsdtFutures;
const string symbol = "BTCUSDT";
const string marginAsset = "USDT";

var ticker = await client.FuturesApiV2.ExchangeData.GetTickerAsync(productType, symbol);
if (!ticker.Success)
{
    Console.WriteLine($"Failed to get futures ticker: {ticker.Error}");
    return;
}

Console.WriteLine($"{ticker.Data.Symbol} last={ticker.Data.LastPrice}, mark={ticker.Data.MarkPrice}");

var balances = await client.FuturesApiV2.Account.GetBalancesAsync(productType);
if (!balances.Success)
{
    Console.WriteLine($"Failed to get futures balances: {balances.Error}");
    return;
}

var positions = await client.FuturesApiV2.Trading.GetPositionsAsync(productType, marginAsset);
```

## Futures Limit Order

Futures examples can create real exposure. Only generate futures trading code when the user asks for it.

```csharp
var order = await client.FuturesApiV2.Trading.PlaceOrderAsync(
    productType: productType,
    symbol: symbol,
    marginAsset: marginAsset,
    side: OrderSide.Buy,
    type: OrderType.Limit,
    marginMode: MarginMode.CrossMargin,
    quantity: 0.001m,
    price: 1m,
    timeInForce: TimeInForce.GoodTillCanceled,
    tradeSide: TradeSide.Open);

if (!order.Success)
{
    Console.WriteLine($"Failed to place futures order: {order.Error}");
    return;
}

var cancel = await client.FuturesApiV2.Trading.CancelOrderAsync(
    productType: productType,
    symbol: symbol,
    orderId: order.Data.OrderId,
    marginAsset: marginAsset);
```

Common futures variations:

- USDT futures: `BitgetProductTypeV2.UsdtFutures`
- Coin-margined futures: use the matching `BitgetProductTypeV2` value from source
- Set leverage: `client.FuturesApiV2.Account.SetLeverageAsync(...)`
- Set margin mode: `client.FuturesApiV2.Account.SetMarginModeAsync(...)`
- Set position mode: `client.FuturesApiV2.Account.SetPositionModeAsync(...)`
- Close positions: `client.FuturesApiV2.Trading.ClosePositionsAsync(...)`

## Spot Margin

Spot margin endpoints live under `client.SpotApiV2.Margin`. Generate borrow, repay, margin order, and liquidation-history code only when the user explicitly asks for margin behavior.

```csharp
var crossBalances = await client.SpotApiV2.Margin.GetCrossBalancesAsync();
if (!crossBalances.Success)
{
    Console.WriteLine($"Failed to get cross margin balances: {crossBalances.Error}");
    return;
}
```

Check exact cross/isolated method overloads in `IBitgetRestClientSpotApiMargin` before generating write operations.

## Copy Trading

Copy trading endpoints live under `client.CopyTradingFuturesV2.Trader` and `client.CopyTradingFuturesV2.Follower`.

```csharp
var traders = await client.CopyTradingFuturesV2.Follower.GetMyTradersAsync();
if (!traders.Success)
{
    Console.WriteLine($"Failed to get copy traders: {traders.Error}");
    return;
}
```

## UnifiedApi

Use `client.UnifiedApi` only for UTA/unified-account workflows. It has `Account`, `ExchangeData`, and `Trading` groups.

```csharp
var account = await client.UnifiedApi.Account.GetAccountInfoAsync();
if (!account.Success)
{
    Console.WriteLine($"Failed to get unified account info: {account.Error}");
    return;
}
```

Do not assume UnifiedApi is a replacement for V2 spot/futures examples unless the user asks for UTA/unified account behavior.

## Public Websocket Subscription

```csharp
var socket = new BitgetSocketClient();

var tickerSub = await socket.SpotApiV2.SubscribeToTickerUpdatesAsync(
    "BTCUSDT",
    update =>
    {
        var ticker = update.Data.FirstOrDefault();
        if (ticker != null)
            Console.WriteLine($"BTCUSDT ticker: {ticker.LastPrice}");
    });

if (!tickerSub.Success)
{
    Console.WriteLine($"Failed to subscribe: {tickerSub.Error}");
    return;
}

await socket.UnsubscribeAsync(tickerSub.Data);
```

Keep handlers fast; push expensive processing to a queue or channel.

## Futures Kline Websocket Subscription

```csharp
var klineSub = await socket.FuturesApiV2.SubscribeToKlineUpdatesAsync(
    BitgetProductTypeV2.UsdtFutures,
    "ETHUSDT",
    BitgetStreamKlineIntervalV2.OneMinute,
    update =>
    {
        var candle = update.Data.FirstOrDefault();
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
var socket = new BitgetSocketClient(options =>
{
    options.ApiCredentials = new BitgetCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
});

var orderSub = await socket.SpotApiV2.SubscribeToOrderUpdatesAsync(
    update =>
    {
        foreach (var order in update.Data)
            Console.WriteLine($"{order.Symbol} {order.OrderId}: {order.Status}");
    });

if (!orderSub.Success)
{
    Console.WriteLine($"Failed to subscribe order updates: {orderSub.Error}");
    return;
}

await socket.UnsubscribeAsync(orderSub.Data);
```

## SharedApis REST

Use this when the user wants exchange-agnostic code.

```csharp
using Bitget.Net.Clients;
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new BitgetRestClient().SpotApiV2.SharedClient;
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
var socket = new BitgetSocketClient();
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

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Errors are normally returned in `Error`, not thrown.

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

var ticker = await WithRetry(() => client.SpotApiV2.ExchangeData.GetTickersAsync("BTCUSDT"));
```

Common handling guidance:

- Retry only transient network, server, or rate-limit errors.
- Do not retry validation errors, permission errors, insufficient balance, or rejected orders blindly.
- Use `ct: cancellationToken` on REST methods when the calling application has cancellation.
- Use `options.RequestTimeout = TimeSpan.FromSeconds(10)` for per-request timeout configuration.

## Dependency Injection

```csharp
using Bitget.Net;

services.AddBitget(options =>
{
    options.Rest.ApiCredentials = new BitgetCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
    options.Socket.ApiCredentials = new BitgetCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
});
```

Inject `IBitgetRestClient` and `IBitgetSocketClient`, or follow the consuming project's existing interface pattern. `AddBitget` registers shared REST and socket interfaces for both `SpotApiV2.SharedClient` and `FuturesApiV2.SharedClient`.

## Local Examples

When available, read:

- `../Bitget.Net/Examples/ai-friendly/01-spot-quickstart.cs`
- `../Bitget.Net/Examples/ai-friendly/02-futures-v2.cs`
- `../Bitget.Net/Examples/ai-friendly/03-websocket.cs`
- `../Bitget.Net/Examples/ai-friendly/04-multi-exchange.cs`
- `../Bitget.Net/Examples/ai-friendly/05-error-handling.cs`

