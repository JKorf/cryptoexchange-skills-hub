# Kraken.Net Usage

Use these snippets as compact patterns for Kraken-specific code. They assume:

```csharp
using CryptoExchange.Net.Authentication;
using CryptoExchange.Net.Objects;
using Kraken.Net;
using Kraken.Net.Clients;
using Kraken.Net.Enums;
```

## Public Spot Market Data

```csharp
var client = new KrakenRestClient();

var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("ETHUSDT");
if (!ticker.Success)
{
    Console.WriteLine($"Ticker failed: {ticker.Error}");
    return;
}

Console.WriteLine(ticker.Data.Values.First().LastTrade.Price);
```

## Credentials

Spot-only credentials:

```csharp
var client = new KrakenRestClient(options =>
{
    options.ApiCredentials = new KrakenCredentials(
        new HMACCredential("SPOT_KEY", "SPOT_SECRET"),
        null);
});
```

Futures-only credentials:

```csharp
var client = new KrakenRestClient(options =>
{
    options.ApiCredentials = new KrakenCredentials(
        null,
        new HMACCredential("FUTURES_KEY", "FUTURES_SECRET"));
});
```

Use `new KrakenCredentials(spotCredential, futuresCredential)` or `.WithSpot(...)` / `.WithFutures(...)`. Avoid the obsolete string constructor.

## Spot Balances

```csharp
var balances = await client.SpotApi.Account.GetBalancesAsync();
if (!balances.Success)
{
    Console.WriteLine($"Balances failed: {balances.Error}");
    return;
}
```

## Spot Order Validation

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "ETHUSDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.01m,
    price: 1000m,
    validateOnly: true);

if (!order.Success)
{
    Console.WriteLine($"Order validation failed: {order.Error}");
    return;
}
```

Set `validateOnly: false` or omit it only when the user wants a live spot order.

## Spot Websocket Symbol

```csharp
var symbols = await client.SpotApi.ExchangeData.GetSymbolsAsync(new[] { "ETHUSDT" });
if (!symbols.Success || symbols.Data.Count == 0)
    return;

var wsSymbol = symbols.Data.Values.First().WebsocketName; // ETH/USDT
```

REST spot methods use `ETHUSDT`; spot websocket subscriptions usually use `ETH/USDT`.

## Futures Market Data

```csharp
var ticker = await client.FuturesApi.ExchangeData.GetTickerAsync("PF_ETHUSD");
if (!ticker.Success)
{
    Console.WriteLine($"Futures ticker failed: {ticker.Error}");
    return;
}

var klines = await client.FuturesApi.ExchangeData.GetKlinesAsync(
    TickType.Trade,
    "PF_ETHUSD",
    FuturesKlineInterval.OneMinute,
    limit: 10);
```

## Futures Leverage And Order

```csharp
var leverage = await client.FuturesApi.Trading.SetLeverageAsync("PF_ETHUSD", 5);
if (!leverage.Success)
{
    Console.WriteLine($"Set leverage failed: {leverage.Error}");
    return;
}

var order = await client.FuturesApi.Trading.PlaceOrderAsync(
    symbol: "PF_ETHUSD",
    side: OrderSide.Buy,
    type: FuturesOrderType.Limit,
    quantity: 0.1m,
    price: 1000m);
```

Futures order placement is live. Use read-only examples unless the user asks to trade.

## Spot Websocket Subscription

```csharp
var socket = new KrakenSocketClient();

var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "ETH/USDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine($"Subscribe failed: {sub.Error}");
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Subscription methods return `WebSocketResult<UpdateSubscription>`.

## Spot Websocket Order Request

```csharp
var socket = new KrakenSocketClient(options =>
{
    options.ApiCredentials = new KrakenCredentials(
        new HMACCredential("SPOT_KEY", "SPOT_SECRET"),
        null);
});

var order = await socket.SpotApi.PlaceOrderAsync(
    symbol: "ETH/USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.01m,
    limitPrice: 1000m,
    validateOnly: true);

if (!order.Success)
{
    Console.WriteLine($"Socket order validation failed: {order.Error}");
    return;
}
```

Spot websocket order request methods return `QueryResult<T>`.

## Futures Websocket Subscription

```csharp
var sub = await socket.FuturesApi.SubscribeToTickerUpdatesAsync(
    "PF_ETHUSD",
    update => Console.WriteLine(update.Data));
```

Authenticated futures socket streams require futures credentials.

## Earn

```csharp
var strategies = await client.SpotApi.Earn.GetStrategiesAsync(asset: "ETH");
if (!strategies.Success)
{
    Console.WriteLine($"Earn strategies failed: {strategies.Error}");
    return;
}
```

Generate `AllocateEarnFundsAsync` or `DeallocateEarnFundsAsync` only when explicitly requested.

## SharedApis Spot Ticker

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new KrakenRestClient().SpotApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USDT");

var result = await tickers.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

Use native Kraken APIs for Kraken-specific Earn, wallet transfer, 2FA, websocket v2 order, and futures-specific details.

## Dependency Injection

```csharp
services.AddKraken(options =>
{
    options.ApiCredentials = new KrakenCredentials(
        new HMACCredential("SPOT_KEY", "SPOT_SECRET"),
        new HMACCredential("FUTURES_KEY", "FUTURES_SECRET"));
});
```

Inject `IKrakenRestClient` and `IKrakenSocketClient`, or specific registered interfaces already used in the application.

## Error Handling

```csharp
var result = await client.SpotApi.ExchangeData.GetTickerAsync("ETHUSDT");
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
