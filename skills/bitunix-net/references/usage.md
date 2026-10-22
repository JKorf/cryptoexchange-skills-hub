# Bitunix.Net Usage Patterns

Use this reference for practical snippets after selecting the correct surface in `api-surfaces.md`. All private examples use placeholders and represent live operations when sent to Bitunix.

## Public Spot Price

```csharp
using Bitunix.Net.Clients;

using var client = new BitunixRestClient();
var result = await client.SpotApi.ExchangeData
    .GetLastPriceAsync("ETHUSDT");

if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data);
```

## Public Futures Data

```csharp
var tickers = await client.FuturesApi.ExchangeData
    .GetTickersAsync(new[] { "ETHUSDT" });

var funding = await client.FuturesApi.ExchangeData
    .GetFundingRateAsync("ETHUSDT");
```

Check each result independently before accessing data.

## Credentials And Balances

```csharp
using Bitunix.Net;
using Bitunix.Net.Clients;

using var client = new BitunixRestClient(options =>
{
    options.ApiCredentials =
        new BitunixCredentials("API_KEY", "API_SECRET");
});

var spotBalances = await client.SpotApi.Account.GetBalancesAsync();
var futuresBalance = await client.FuturesApi.Account
    .GetBalancesAsync("USDT");
```

Bitunix Futures balance lookup requires the margin asset.

## Spot Limit Order

```csharp
using Bitunix.Net.Enums;

var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "ETHUSDT",
    orderSide: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 0.01m,
    price: 2000m);

if (!order.Success)
{
    Console.WriteLine(order.Error);
    return;
}

var cancel = await client.SpotApi.Trading.CancelOrderAsync(
    "ETHUSDT",
    order.Data.OrderId);
```

There is no test-order endpoint. A valid placement request can create a live order.

## Futures Settings And Order

```csharp
var mode = await client.FuturesApi.Account
    .SetMarginModeAsync("ETHUSDT", "USDT", MarginMode.Isolated);

if (!mode.Success)
{
    Console.WriteLine(mode.Error);
    return;
}

var leverage = await client.FuturesApi.Account
    .SetLeverageAsync("ETHUSDT", "USDT", 5);

if (!leverage.Success)
{
    Console.WriteLine(leverage.Error);
    return;
}

var order = await client.FuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETHUSDT",
    side: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 0.1m,
    price: 2000m,
    tradeSide: TradeSide.Open,
    timeInForce: TimeInForce.GoodTillCanceled);

if (!order.Success)
    Console.WriteLine(order.Error);
```

Setting margin mode/leverage and placing the order are separate live mutations. In one-way mode, omit `tradeSide` unless the endpoint semantics require it and use `reduceOnly` deliberately when reducing exposure.

## Futures Ticker Subscription

```csharp
using var socket = new BitunixSocketClient();

var sub = await socket.FuturesApi.SubscribeToTickerUpdatesAsync(
    "ETHUSDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

try
{
    await Task.Delay(Timeout.InfiniteTimeSpan);
}
finally
{
    await socket.UnsubscribeAsync(sub.Data);
}
```

Production services should tie shutdown to a cancellation token instead of an infinite delay.

## Futures User Updates

```csharp
using var socket = new BitunixSocketClient(options =>
{
    options.ApiCredentials =
        new BitunixCredentials("API_KEY", "API_SECRET");
});

var sub = await socket.FuturesApi.SubscribeToUserUpdatesAsync(
    onBalanceMessage: update => Console.WriteLine("Balance update"),
    onOrderMessage: update => Console.WriteLine("Order update"),
    onPositionMessage: update => Console.WriteLine("Position update"),
    onTpSlMessage: update => Console.WriteLine("TP/SL update"));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

No REST listen key is created or renewed.

## Spot Socket Query

```csharp
using var socket = new BitunixSocketClient(options =>
{
    options.ApiCredentials =
        new BitunixCredentials("API_KEY", "API_SECRET");
});

var order = await socket.SpotApi.PlaceOrderAsync(
    "ETHUSDT",
    OrderSide.Buy,
    OrderType.Limit,
    0.01m,
    price: 2000m);

if (!order.Success)
{
    Console.WriteLine(order.Error);
    return;
}

Console.WriteLine(order.Data.OrderId);
```

This returns `QueryResult<BitunixOrderResult>`, not a subscription.

## Shared Futures Ticker

```csharp
using CryptoExchange.Net.SharedApis;

using var client = new BitunixRestClient();
IGetTickerRest ticker = client.FuturesApi.SharedApi;

var request = new GetTickerRequest(
    new SharedSymbol(
        TradingMode.PerpetualLinear,
        "ETH",
        "USDT"));

var result = await ticker.GetTickerAsync(request);
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

For Spot prices, call native `GetLastPriceAsync`; Spot SharedApi has no ticker capability.

## Dependency Injection

```csharp
services.AddBitunix(options =>
{
    options.ApiCredentials =
        new BitunixCredentials("API_KEY", "API_SECRET");
});
```

```csharp
public sealed class PriceService
{
    private readonly IBitunixRestClient _client;

    public PriceService(IBitunixRestClient client)
    {
        _client = client;
    }

    public async Task<decimal?> GetPriceAsync(CancellationToken ct)
    {
        var result = await _client.SpotApi.ExchangeData
            .GetLastPriceAsync("ETHUSDT", ct);
        return result.Success ? result.Data : null;
    }
}
```

## Error Handling

```csharp
var result = await client.SpotApi.ExchangeData
    .GetLastPriceAsync("ETHUSDT");

if (!result.Success)
{
    if (result.Error?.IsTransient == true)
    {
        // Apply bounded backoff and cancellation.
    }

    Console.WriteLine(
        $"{result.Error?.Code}: {result.Error?.Message}");
    return;
}
```

API failures are represented in result errors. Do not rely on exceptions for normal API rejections.
