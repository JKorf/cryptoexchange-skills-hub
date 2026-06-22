# OKX.Net Usage

Use these snippets as compact patterns for OKX-specific code. They assume:

```csharp
using CryptoExchange.Net.Objects;
using OKX.Net;
using OKX.Net.Clients;
using OKX.Net.Enums;
```

## Public Spot Market Data

```csharp
var client = new OKXRestClient();

var ticker = await client.UnifiedApi.ExchangeData.GetTickerAsync("ETH-USDT");
if (!ticker.Success)
{
    Console.WriteLine($"Ticker failed: {ticker.Error}");
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

## Credentials

```csharp
var client = new OKXRestClient(options =>
{
    options.ApiCredentials = new OKXCredentials(
        "API_KEY",
        "API_SECRET",
        "PASSPHRASE");
});
```

OKX credentials require a passphrase. Use `OKXCredentials(key, secret, pass)` for REST and private socket examples.

## Demo Environment

```csharp
var client = new OKXRestClient(options =>
{
    options.Environment = OKXEnvironment.Demo;
    options.ApiCredentials = new OKXCredentials(
        "DEMO_KEY",
        "DEMO_SECRET",
        "DEMO_PASSPHRASE");
});
```

Use demo only when the user explicitly wants sandbox/demo behavior.

## Account Balance

```csharp
var balance = await client.UnifiedApi.Account.GetAccountBalanceAsync("USDT");
if (!balance.Success)
{
    Console.WriteLine($"Balance failed: {balance.Error}");
    return;
}

foreach (var asset in balance.Data.Details)
    Console.WriteLine($"{asset.Asset}: {asset.AvailableBalance}");
```

## Spot Order Check

```csharp
var check = await client.UnifiedApi.Trading.CheckOrderAsync(
    symbol: "ETH-USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.01m,
    price: 2000m,
    tradeMode: TradeMode.Cash);

if (!check.Success)
{
    Console.WriteLine($"Order check failed: {check.Error}");
    return;
}
```

Prefer `CheckOrderAsync` for examples that should not place live orders.

## Live Spot Order With Cleanup

```csharp
var order = await client.UnifiedApi.Trading.PlaceOrderAsync(
    symbol: "ETH-USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.01m,
    price: 2000m,
    tradeMode: TradeMode.Cash);

if (!order.Success)
{
    Console.WriteLine($"Place order failed: {order.Error}");
    return;
}

try
{
    var details = await client.UnifiedApi.Trading.GetOrderDetailsAsync(
        "ETH-USDT",
        orderId: order.Data.OrderId);

    if (details.Success)
        Console.WriteLine(details.Data.OrderState);
}
finally
{
    await client.UnifiedApi.Trading.CancelOrderAsync(
        "ETH-USDT",
        orderId: order.Data.OrderId);
}
```

## Swap Market Data

```csharp
const string symbol = "ETH-USDT-SWAP";

var instruments = await client.UnifiedApi.ExchangeData.GetSymbolsAsync(
    InstrumentType.Swap,
    symbol: symbol);

var ticker = await client.UnifiedApi.ExchangeData.GetTickerAsync(symbol);
```

## Swap Leverage And Position

```csharp
var leverage = await client.UnifiedApi.Account.SetLeverageAsync(
    leverage: 5,
    marginMode: MarginMode.Cross,
    symbol: "ETH-USDT-SWAP",
    positionSide: PositionSide.Long);

if (!leverage.Success)
{
    Console.WriteLine($"Set leverage failed: {leverage.Error}");
    return;
}

var positions = await client.UnifiedApi.Account.GetPositionsAsync(
    InstrumentType.Swap,
    symbol: "ETH-USDT-SWAP");
```

`SetLeverageAsync` changes live account state.

## Swap Order And Close

```csharp
var order = await client.UnifiedApi.Trading.PlaceOrderAsync(
    symbol: "ETH-USDT-SWAP",
    side: OrderSide.Buy,
    type: OrderType.Market,
    quantity: 1m,
    positionSide: PositionSide.Long,
    tradeMode: TradeMode.Cross);
```

Close a position explicitly:

```csharp
var close = await client.UnifiedApi.Trading.ClosePositionAsync(
    symbol: "ETH-USDT-SWAP",
    marginMode: MarginMode.Cross,
    positionSide: PositionSide.Long);
```

Both calls are live account actions.

## Public Websocket Subscription

```csharp
var socket = new OKXSocketClient();

var sub = await socket.UnifiedApi.ExchangeData.SubscribeToTickerUpdatesAsync(
    "ETH-USDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine($"Subscribe failed: {sub.Error}");
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Subscription methods return `WebSocketResult<UpdateSubscription>`.

## Private Websocket Subscription

```csharp
var socket = new OKXSocketClient(options =>
{
    options.ApiCredentials = new OKXCredentials(
        "API_KEY",
        "API_SECRET",
        "PASSPHRASE");
});

var sub = await socket.UnifiedApi.Trading.SubscribeToOrderUpdatesAsync(
    InstrumentType.Spot,
    symbol: "ETH-USDT",
    instrumentFamily: null,
    onData: update => Console.WriteLine(update.Data.OrderState));
```

## Websocket Order Request

```csharp
var symbols = await client.UnifiedApi.ExchangeData.GetSymbolsAsync(
    InstrumentType.Spot,
    symbol: "ETH-USDT");

if (!symbols.Success || symbols.Data.Length == 0 || symbols.Data[0].SymbolCode == null)
{
    Console.WriteLine($"Symbol lookup failed: {symbols.Error}");
    return;
}

var order = await socket.UnifiedApi.Trading.PlaceOrderAsync(
    symbolCode: symbols.Data[0].SymbolCode.Value,
    side: OrderSide.Buy,
    type: OrderType.Limit,
    tradeMode: TradeMode.Cash,
    quantity: 0.01m,
    price: 2000m);

if (!order.Success)
{
    Console.WriteLine($"Socket order failed: {order.Error}");
    return;
}
```

Socket order request methods return `QueryResult<T>` and are live trading requests. They use `OKXInstrument.SymbolCode` from instrument metadata, not the display symbol string.

## SharedApis Spot Ticker

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new OKXRestClient().UnifiedApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USDT");

var result = await tickers.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

Use native OKX APIs for account mode, copy trading, subaccounts, detailed options data, websocket order requests, and OKX-specific algo order controls.

## Dependency Injection

```csharp
services.AddOKX(options =>
{
    options.ApiCredentials = new OKXCredentials(
        "API_KEY",
        "API_SECRET",
        "PASSPHRASE");
});
```

Inject `IOKXRestClient` and `IOKXSocketClient`, or specific registered interfaces already used in the application.

## Error Handling

```csharp
var result = await client.UnifiedApi.ExchangeData.GetTickerAsync("ETH-USDT");
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

REST failures, API validation errors, socket request failures, and rate limits are returned on `.Error`. Do not rely on exceptions for normal API failures.
