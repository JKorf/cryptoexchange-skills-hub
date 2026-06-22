# Mexc.Net Usage

Use these snippets as compact patterns for MEXC-specific code. They assume:

```csharp
using CryptoExchange.Net.Objects;
using Mexc.Net;
using Mexc.Net.Clients;
using Mexc.Net.Enums;
```

## Public Spot Market Data

```csharp
var client = new MexcRestClient();

var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTCUSDT");
if (!ticker.Success)
{
    Console.WriteLine($"Ticker failed: {ticker.Error}");
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

## Credentials

```csharp
var client = new MexcRestClient(options =>
{
    options.ApiCredentials = new MexcCredentials("API_KEY", "API_SECRET");
});
```

Use HMAC credentials unless the target project already uses RSA credentials:

```csharp
options.ApiCredentials = new MexcCredentials()
    .WithRSAPem("API_KEY", "PRIVATE_KEY_PEM");
```

## Spot Account

```csharp
var account = await client.SpotApi.Account.GetAccountInfoAsync();
if (!account.Success)
{
    Console.WriteLine($"Account failed: {account.Error}");
    return;
}

foreach (var balance in account.Data.Balances.Where(x => x.Total > 0))
    Console.WriteLine($"{balance.Asset}: {balance.Available}");
```

## Spot Order Test

```csharp
var order = await client.SpotApi.Trading.PlaceTestOrderAsync(
    symbol: "BTCUSDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    price: 50000m);

if (!order.Success)
{
    Console.WriteLine($"Order validation failed: {order.Error}");
    return;
}
```

Use `PlaceOrderAsync` only when the user explicitly wants a live spot order.

## Live Spot Order With Cleanup

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTCUSDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    price: 40000m);

if (!order.Success)
{
    Console.WriteLine($"Place order failed: {order.Error}");
    return;
}

try
{
    var status = await client.SpotApi.Trading.GetOrderAsync("BTCUSDT", orderId: order.Data.OrderId);
    if (status.Success)
        Console.WriteLine(status.Data.Status);
}
finally
{
    await client.SpotApi.Trading.CancelOrderAsync("BTCUSDT", orderId: order.Data.OrderId);
}
```

## Futures Market Data

```csharp
const string symbol = "ETH_USDT";

var contract = await client.FuturesApi.ExchangeData.GetSymbolAsync(symbol);
if (!contract.Success)
{
    Console.WriteLine($"Contract failed: {contract.Error}");
    return;
}

var ticker = await client.FuturesApi.ExchangeData.GetTickerAsync(symbol);
if (!ticker.Success)
{
    Console.WriteLine($"Ticker failed: {ticker.Error}");
    return;
}
```

## Futures Leverage And Position

```csharp
var leverage = await client.FuturesApi.Account.SetLeverageAsync(
    leverage: 5,
    marginType: MarginType.Isolated,
    symbol: "ETH_USDT",
    positionSide: PositionSide.Long);

if (!leverage.Success)
{
    Console.WriteLine($"Set leverage failed: {leverage.Error}");
    return;
}

var positions = await client.FuturesApi.Trading.GetPositionsAsync("ETH_USDT");
```

`SetLeverageAsync` changes live account state.

## Futures Order

```csharp
var order = await client.FuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETH_USDT",
    side: FuturesOrderSide.OpenLong,
    type: FuturesOrderType.Market,
    quantity: 1m,
    leverage: 5,
    marginType: MarginType.Isolated);
```

Futures order placement is live. Use `FuturesOrderSide.CloseLong` or `CloseShort` for close-position examples, and prefer read-only futures examples unless trading is requested.

## Spot Websocket Subscription

```csharp
var socket = new MexcSocketClient();

var sub = await socket.SpotApi.SubscribeToMiniTickerUpdatesAsync(
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

## Spot Private Websocket Subscription

Automatic listen-key management:

```csharp
var socket = new MexcSocketClient(options =>
{
    options.ApiCredentials = new MexcCredentials("API_KEY", "API_SECRET");
});

var sub = await socket.SpotApi.SubscribeToOrderUpdatesAsync(
    update => Console.WriteLine($"Spot order {update.Data.OrderId}: {update.Data.Status}"));
```

Manual listen-key management:

```csharp
var rest = new MexcRestClient(options =>
{
    options.ApiCredentials = new MexcCredentials("API_KEY", "API_SECRET");
});

var listenKey = await rest.SpotApi.Account.StartUserStreamAsync();
if (!listenKey.Success)
    return;

var sub = await socket.SpotApi.SubscribeToOrderUpdatesAsync(
    listenKey.Data,
    update => Console.WriteLine(update.Data.OrderId));

await socket.UnsubscribeAsync(sub.Data);
await rest.SpotApi.Account.StopUserStreamAsync(listenKey.Data);
```

## Futures Websocket Subscription

```csharp
var sub = await socket.FuturesApi.SubscribeToTickerUpdatesAsync(
    "ETH_USDT",
    update => Console.WriteLine(update.Data.LastPrice));
```

Authenticated futures user-data stream:

```csharp
var sub = await socket.FuturesApi.SubscribeToUserDataUpdatesAsync(
    orderUpdateHandler: update => Console.WriteLine(update.Data.OrderId),
    positionUpdateHandler: update => Console.WriteLine(update.Data.PositionSize));
```

## SharedApis Spot Ticker

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new MexcRestClient().SpotApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");

var result = await tickers.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

Use native Mexc APIs for subaccount management, spot listen-key management, detailed futures order controls, plan orders, TP/SL, trailing orders, close-all, and reverse-position flows.

## Dependency Injection

```csharp
services.AddMexc(options =>
{
    options.ApiCredentials = new MexcCredentials("API_KEY", "API_SECRET");
});
```

Inject `IMexcRestClient` and `IMexcSocketClient`, or specific registered interfaces already used in the application.

## Error Handling

```csharp
var result = await client.SpotApi.ExchangeData.GetTickerAsync("BTCUSDT");
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

REST failures, API validation errors, and rate limits are returned on `.Error`. Do not rely on exceptions for normal API failures.
