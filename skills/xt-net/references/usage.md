# XT.Net Usage

```csharp
using XT.Net;
using XT.Net.Clients;
using XT.Net.Enums;
```

## Public Spot Ticker

```csharp
var client = new XTRestClient();
var result = await client.SpotApi.ExchangeData.GetTickersAsync("eth_usdt");
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

var ticker = result.Data.First();
```

## Credentials And Balance

```csharp
var client = new XTRestClient(options =>
{
    options.ApiCredentials = new XTCredentials("API_KEY", "API_SECRET");
});

var balances = await client.SpotApi.Account.GetBalancesAsync();
if (!balances.Success)
    Console.WriteLine(balances.Error);
```

## Validate Spot Symbol

```csharp
var symbols = await client.SpotApi.ExchangeData.GetSymbolsAsync(symbol: "eth_usdt");
if (!symbols.Success || !symbols.Data.Any())
    return;
```

Inspect the returned symbol's trading status, quantity/price precision, and order limits before placing an order.

## Spot Limit Order With Cleanup

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    "eth_usdt",
    OrderSide.Buy,
    OrderType.Limit,
    TimeInForce.GoodTillCanceled,
    BusinessType.Spot,
    quantity: 0.01m,
    price: 1000m);

if (!order.Success)
    return;

try
{
    var status = await client.SpotApi.Trading.GetOrderAsync(order.Data.OrderId);
}
finally
{
    await client.SpotApi.Trading.CancelOrderAsync(order.Data.OrderId);
}
```

This places a live order. For a market buy, pass `quoteQuantity` and omit `quantity`; for a market sell, pass `quantity`.

## USDT-M Futures

```csharp
const string symbol = "ETH_USDT";

var leverage = await client.UsdtFuturesApi.Account.SetLeverageAsync(
    symbol, PositionSide.Long, leverage: 5);
if (!leverage.Success)
    return;

var order = await client.UsdtFuturesApi.Trading.PlaceOrderAsync(
    symbol,
    OrderSide.Buy,
    OrderType.Market,
    quantity: 0.01m,
    positionSide: PositionSide.Long);

var positions = await client.UsdtFuturesApi.Trading.GetPositionsAsync(symbol);
```

Use `client.CoinFuturesApi` for Coin-M REST operations. Do not change the socket root: futures subscriptions still use `socket.FuturesApi`.

## Public Websockets

```csharp
var socket = new XTSocketClient();

var spotSub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "eth_usdt", update => Console.WriteLine(update.Data));

var futuresSub = await socket.FuturesApi.SubscribeToTickerUpdatesAsync(
    "ETH_USDT", update => Console.WriteLine(update.Data));

if (spotSub.Success)
    await socket.UnsubscribeAsync(spotSub.Data);
if (futuresSub.Success)
    await socket.UnsubscribeAsync(futuresSub.Data);
```

## Private Spot Stream

```csharp
var token = await client.SpotApi.Account.GetWebsocketTokenAsync();
if (!token.Success)
    return;

var socket = new XTSocketClient();
var sub = await socket.SpotApi.SubscribeToOrderUpdatesAsync(
    token.Data,
    update => Console.WriteLine(update.Data));
```

Alternatively configure credentials on `XTSocketClient` and use the overload without a token.

## Private Futures Stream

```csharp
var listenKey = await client.UsdtFuturesApi.Account.GetListenKeyAsync();
if (!listenKey.Success)
    return;

var socket = new XTSocketClient();
var sub = await socket.FuturesApi.SubscribeToPositionUpdatesAsync(
    listenKey.Data,
    update => Console.WriteLine(update.Data));
```

## Shared API V2

Use a narrow capability interface when the operation is known at compile time:

```csharp
using CryptoExchange.Net.SharedApis;

using var client = new XTRestClient();
IGetTickerRest ticker = client.SpotApi.SharedApi;
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USDT");

var result = await ticker.GetTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

Inject `IXTSharedApiClient` when a service needs multiple XT Shared API surfaces. It exposes `SpotRest`, `UsdtFuturesRest`, `CoinFuturesRest`, `SpotSocket`, `FuturesSocket`. When the operation or transport is selected dynamically, use a typed capability descriptor:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is null)
    return;

Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

For WebSocket workflows, use the narrow subscription interface exposed by the applicable socket surface, such as `ISubscribeTickerSocket` or `ISubscribeTradesSocket`. Prefer an aggregate property or direct capability injection when the required surface is known at compile time.

## Dependency Injection

```csharp
services.AddXT(options =>
{
    options.ApiCredentials = new XTCredentials("API_KEY", "API_SECRET");
});
```

## Error Handling

```csharp
var result = await client.SpotApi.ExchangeData.GetTickersAsync("eth_usdt");
if (!result.Success)
{
    if (result.Error?.IsTransient == true)
    {
        // Retry with bounded backoff.
    }

    Console.WriteLine($"{result.Error?.Code}: {result.Error?.Message}");
}
```
