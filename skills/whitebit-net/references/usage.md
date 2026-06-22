# WhiteBit.Net Usage

```csharp
using CryptoExchange.Net.Objects;
using WhiteBit.Net;
using WhiteBit.Net.Clients;
using WhiteBit.Net.Enums;
```

## Public Ticker

```csharp
var client = new WhiteBitRestClient();
var tickers = await client.V4Api.ExchangeData.GetTickersAsync();
if (!tickers.Success)
{
    Console.WriteLine(tickers.Error);
    return;
}

var ticker = tickers.Data.SingleOrDefault(x => x.Symbol == "ETH_USDT");
```

## Credentials And Balances

```csharp
var client = new WhiteBitRestClient(options =>
{
    options.ApiCredentials = new WhiteBitCredentials("API_KEY", "API_SECRET");
});

var balances = await client.V4Api.Account.GetSpotBalancesAsync();
```

## Validate Symbol Metadata

```csharp
var symbols = await client.V4Api.ExchangeData.GetSymbolsAsync();
if (!symbols.Success)
    return;

var symbol = symbols.Data.SingleOrDefault(x => x.Name == "ETH_USDT");
if (symbol == null || !symbol.TradingEnabled)
    return;
```

Check `MinOrderQuantity` and `MinOrderValue` before trading.

## Live Spot Order With Cleanup

```csharp
var order = await client.V4Api.Trading.PlaceSpotOrderAsync(
    "ETH_USDT", OrderSide.Buy, NewOrderType.Limit,
    quantity: 0.01m, price: 2000m);

if (!order.Success)
    return;

try
{
    var open = await client.V4Api.Trading.GetOpenOrdersAsync(
        symbol: "ETH_USDT", orderId: order.Data.OrderId);
}
finally
{
    await client.V4Api.Trading.CancelOrderAsync(
        "ETH_USDT", order.Data.OrderId);
}
```

## Collateral Position And Close

```csharp
await client.V4Api.Account.SetAccountLeverageAsync(5);

var open = await client.V4Api.CollateralTrading.PlaceOrderAsync(
    "ETH_PERP", OrderSide.Buy, NewOrderType.Market,
    quantity: 0.1m);

var positions = await client.V4Api.CollateralTrading.GetOpenPositionsAsync("ETH_PERP");
var position = positions.Success
    ? positions.Data.FirstOrDefault(x => x.Quantity != 0)
    : null;

if (position != null)
{
    await client.V4Api.CollateralTrading.PlaceOrderAsync(
        "ETH_PERP", OrderSide.Sell, NewOrderType.Market,
        quantity: Math.Abs(position.Quantity),
        positionSide: position.PositionSide,
        reduceOnly: true);
}
```

## Public Websocket

```csharp
var socket = new WhiteBitSocketClient();
var sub = await socket.V4Api.SubscribeToTickerUpdatesAsync(
    "ETH_USDT",
    update => Console.WriteLine(update.Data.Ticker.LastPrice));

if (sub.Success)
    await socket.UnsubscribeAsync(sub.Data);
```

## Private Websocket

```csharp
var socket = new WhiteBitSocketClient(options =>
{
    options.ApiCredentials = new WhiteBitCredentials("API_KEY", "API_SECRET");
});

var sub = await socket.V4Api.SubscribeToSpotBalanceUpdatesAsync(
    new[] { "ETH", "USDT" },
    update => Console.WriteLine(update.Data.Count));
```

## Socket Request

```csharp
var result = await socket.V4Api.GetTickerAsync("ETH_USDT");
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}
```

Socket requests return `QueryResult<T>`; subscriptions return `WebSocketResult<UpdateSubscription>`.

## SharedApis

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new WhiteBitRestClient().V4Api.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "ETH", "USDT")));
```

## Dependency Injection

```csharp
services.AddWhiteBit(options =>
{
    options.ApiCredentials = new WhiteBitCredentials("API_KEY", "API_SECRET");
});
```

## Error Handling

```csharp
var result = await client.V4Api.ExchangeData.GetTickersAsync();
if (!result.Success)
{
    if (result.Error?.IsTransient == true)
    {
        // Retry with bounded backoff.
    }

    Console.WriteLine($"{result.Error?.Code}: {result.Error?.Message}");
}
```
