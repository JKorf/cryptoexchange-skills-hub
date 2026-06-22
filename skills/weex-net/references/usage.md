# Weex.Net Usage

```csharp
using CryptoExchange.Net.Objects;
using Weex.Net;
using Weex.Net.Clients;
using Weex.Net.Enums;
```

## Public Spot Ticker

```csharp
var client = new WeexRestClient();
var ticker = await client.SpotApi.ExchangeData.GetTickersAsync(new[] { "ETHUSDT" });
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.Single().LastPrice);
```

## Credentials And Account

```csharp
var client = new WeexRestClient(options =>
{
    options.ApiCredentials = new WeexCredentials(
        "API_KEY", "API_SECRET", "API_PASSPHRASE");
});

var account = await client.SpotApi.Account.GetAccountInfoAsync();
if (!account.Success)
{
    Console.WriteLine(account.Error);
    return;
}
```

## Validate Precision

```csharp
var info = await client.SpotApi.ExchangeData.GetExchangeInfoAsync(new[] { "ETHUSDT" });
if (!info.Success || info.Data.Symbols.Length == 0)
    return;

var symbol = info.Data.Symbols[0];
var quantity = Math.Floor(0.123456m / symbol.StepSize) * symbol.StepSize;
var price = Math.Floor(2000.123m / symbol.TickSize) * symbol.TickSize;
```

## Live Spot Order With Cleanup

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    "ETHUSDT", OrderSide.Buy, OrderType.Limit,
    quantity: 0.1m, price: 2000m,
    timeInForce: TimeInForce.GoodTillCanceled);

if (!order.Success)
{
    Console.WriteLine(order.Error);
    return;
}

try
{
    var status = await client.SpotApi.Trading.GetOrderAsync(orderId: order.Data.OrderId);
    if (status.Success)
        Console.WriteLine(status.Data.Status);
}
finally
{
    await client.SpotApi.Trading.CancelOrderAsync(orderId: order.Data.OrderId);
}
```

## Futures Leverage And Order

```csharp
var leverage = await client.FuturesApi.Account.SetLeverageAsync(
    "ETHUSDT", MarginType.Isolated,
    isolatedLongLeverage: 5,
    isolatedShortLeverage: 5);

if (!leverage.Success)
    return;

var order = await client.FuturesApi.Trading.PlaceOrderAsync(
    "ETHUSDT", OrderSide.Buy, PositionSide.Long,
    OrderType.Market, quantity: 0.01m);
```

Regular futures orders use `OrderType`.

## Conditional Futures Order

```csharp
var order = await client.FuturesApi.Trading.PlaceConditionalOrderAsync(
    symbol: "ETHUSDT",
    side: OrderSide.Sell,
    positionSide: PositionSide.Long,
    type: FuturesOrderType.StopMarket,
    quantity: 0.01m,
    triggerPrice: 1800m);
```

Conditional futures orders use `FuturesOrderType` and are live orders.

## Positions And Close

```csharp
var positions = await client.FuturesApi.Trading.GetPositionAsync("ETHUSDT");
if (positions.Success)
{
    var close = await client.FuturesApi.Trading.ClosePositionsAsync("ETHUSDT");
}
```

`ClosePositionsAsync` changes live exposure.

## Public Websocket

```csharp
var socket = new WeexSocketClient();
var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "ETHUSDT", update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
    return;

await socket.UnsubscribeAsync(sub.Data);
```

## Private Websocket

```csharp
var socket = new WeexSocketClient(options =>
{
    options.ApiCredentials = new WeexCredentials(
        "API_KEY", "API_SECRET", "API_PASSPHRASE");
});

var sub = await socket.FuturesApi.SubscribeToOrderUpdatesAsync(
    update => Console.WriteLine(update.Data.Orders.Length));
```

## SharedApis

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new WeexRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "ETH", "USDT")));
```

## Dependency Injection

```csharp
services.AddWeex(options =>
{
    options.ApiCredentials = new WeexCredentials(
        "API_KEY", "API_SECRET", "API_PASSPHRASE");
});
```

## Error Handling

```csharp
var result = await client.SpotApi.ExchangeData.GetTickersAsync(new[] { "ETHUSDT" });
if (!result.Success)
{
    if (result.Error?.IsTransient == true)
    {
        // Retry with bounded backoff.
    }

    Console.WriteLine($"{result.Error?.Code}: {result.Error?.Message}");
}
```
