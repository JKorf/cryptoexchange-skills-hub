# Toobit.Net Usage

Use these snippets as compact patterns for Toobit-specific code. They assume:

```csharp
using CryptoExchange.Net.Objects;
using Toobit.Net;
using Toobit.Net.Clients;
using Toobit.Net.Enums;
```

## Public Spot Market Data

```csharp
var client = new ToobitRestClient();

var ticker = await client.SpotApi.ExchangeData.GetTickersAsync("BTCUSDT");
if (!ticker.Success)
{
    Console.WriteLine($"Ticker failed: {ticker.Error}");
    return;
}

Console.WriteLine(ticker.Data.Single().LastPrice);
```

## Credentials

```csharp
var client = new ToobitRestClient(options =>
{
    options.ApiCredentials = new ToobitCredentials("API_KEY", "API_SECRET");
});
```

Use `ToobitCredentials(key, secret)` for account, trading, withdrawal, transfer, and private socket examples.

## Spot Balances

```csharp
var balances = await client.SpotApi.Account.GetBalancesAsync();
if (!balances.Success)
{
    Console.WriteLine($"Balance failed: {balances.Error}");
    return;
}

foreach (var balance in balances.Data.Balances.Where(x => x.Total > 0))
    Console.WriteLine($"{balance.Asset}: {balance.Free}");
```

## Spot Test Order

```csharp
var test = await client.SpotApi.Trading.PlaceTestOrderAsync(
    symbol: "BTCUSDT",
    orderSide: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 0.001m,
    timeInForce: TimeInForce.GoodTillCanceled,
    price: 50000m);

if (!test.Success)
{
    Console.WriteLine($"Test order failed: {test.Error}");
    return;
}
```

Prefer `PlaceTestOrderAsync` for examples that should not place live orders.

## Live Spot Order With Cleanup

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTCUSDT",
    orderSide: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 0.001m,
    timeInForce: TimeInForce.GoodTillCanceled,
    price: 50000m);

if (!order.Success)
{
    Console.WriteLine($"Place order failed: {order.Error}");
    return;
}

try
{
    var details = await client.SpotApi.Trading.GetOrderAsync(orderId: order.Data.OrderId);
    if (details.Success)
        Console.WriteLine(details.Data.Status);
}
finally
{
    await client.SpotApi.Trading.CancelOrderAsync(orderId: order.Data.OrderId);
}
```

## USDT Futures Leverage And Order

```csharp
const string symbol = "ETH-SWAP-USDT";

var leverage = await client.UsdtFuturesApi.Account.SetLeverageAsync(symbol, 10);
if (!leverage.Success)
{
    Console.WriteLine($"Set leverage failed: {leverage.Error}");
    return;
}

var order = await client.UsdtFuturesApi.Trading.PlaceOrderAsync(
    symbol: symbol,
    orderSide: FuturesOrderSide.BuyOpen,
    orderType: FuturesNewOrderType.Limit,
    quantity: 10,
    price: 2000m,
    timeInForce: TimeInForce.GoodTillCanceled);

if (!order.Success)
{
    Console.WriteLine($"Futures order failed: {order.Error}");
    return;
}
```

Use `FuturesOrderSide.SellClose` to close long exposure and `FuturesOrderSide.BuyClose` to close short exposure.

## USDT Futures Positions

```csharp
var positions = await client.UsdtFuturesApi.Trading.GetPositionsAsync("ETH-SWAP-USDT");
if (!positions.Success)
{
    Console.WriteLine($"Positions failed: {positions.Error}");
    return;
}

foreach (var position in positions.Data)
    Console.WriteLine($"{position.Symbol} {position.PositionSide}: {position.Position}");
```

## Public Websocket Subscription

```csharp
var socket = new ToobitSocketClient();

var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "BTCUSDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine($"Subscribe failed: {sub.Error}");
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Subscription methods return `WebSocketResult<UpdateSubscription>`.

## Spot User Stream With Listen Key

```csharp
var listenKey = await client.SpotApi.Account.StartUserStreamAsync();
if (!listenKey.Success)
{
    Console.WriteLine($"Listen key failed: {listenKey.Error}");
    return;
}

var sub = await socket.SpotApi.SubscribeToUserDataUpdatesAsync(
    listenKey.Data,
    onAccountMessage: update => Console.WriteLine("Account update"),
    onOrderMessage: update => Console.WriteLine(update.Data.Length),
    onUserTradeMessage: update => Console.WriteLine(update.Data.Length));

if (sub.Success)
    await socket.UnsubscribeAsync(sub.Data);

await client.SpotApi.Account.StopUserStreamAsync(listenKey.Data);
```

For services, keep the listen key alive with `KeepAliveUserStreamAsync`.

## Auto-Managed User Stream

```csharp
var socket = new ToobitSocketClient(options =>
{
    options.ApiCredentials = new ToobitCredentials("API_KEY", "API_SECRET");
});

var sub = await socket.SpotApi.SubscribeToUserDataUpdatesAsync(
    onAccountMessage: update => Console.WriteLine("Account update"),
    onOrderMessage: update => Console.WriteLine(update.Data.Length),
    onUserTradeMessage: update => Console.WriteLine(update.Data.Length));
```

Credentialed overloads without a listen key acquire and renew the listen key automatically.

## SharedApis Spot Ticker

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new ToobitRestClient().SpotApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");

var result = await tickers.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

Use native Toobit APIs for listen-key lifecycle, Toobit-specific futures symbols, leverage, margin type, trading stops, and detailed exchange-specific models.

## Dependency Injection

```csharp
services.AddToobit(options =>
{
    options.ApiCredentials = new ToobitCredentials("API_KEY", "API_SECRET");
});
```

Inject `IToobitRestClient`, `IToobitSocketClient`, `IToobitOrderBookFactory`, `IToobitTrackerFactory`, or `IToobitUserClientProvider`.

## Error Handling

```csharp
var result = await client.SpotApi.ExchangeData.GetTickersAsync("BTCUSDT");
if (!result.Success)
{
    if (result.Error?.IsTransient == true)
    {
        // Retry with bounded backoff.
    }

    Console.WriteLine($"{result.Error?.Code}: {result.Error?.Message}");
    return;
}
```

REST failures, API validation errors, socket subscription failures, and rate limits are returned on `.Error`. Do not rely on exceptions for normal API failures.
