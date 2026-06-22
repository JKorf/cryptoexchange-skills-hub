# Kucoin.Net Usage

Use these snippets as compact patterns for KuCoin-specific code. They assume:

```csharp
using CryptoExchange.Net.Objects;
using Kucoin.Net;
using Kucoin.Net.Clients;
using Kucoin.Net.Enums;
```

## Public Spot Market Data

```csharp
var client = new KucoinRestClient();

var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTC-USDT");
if (!ticker.Success)
{
    Console.WriteLine($"Ticker failed: {ticker.Error}");
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

## Credentials

```csharp
var client = new KucoinRestClient(options =>
{
    options.ApiCredentials = new KucoinCredentials(
        "API_KEY",
        "API_SECRET",
        "API_PASSPHRASE");
});
```

KuCoin credentials require a passphrase. Use `KucoinCredentials(key, secret, pass)` for REST and private socket examples.

## Spot Balances

```csharp
var accounts = await client.SpotApi.Account.GetAccountsAsync();
if (!accounts.Success)
{
    Console.WriteLine($"Accounts failed: {accounts.Error}");
    return;
}

foreach (var account in accounts.Data.Where(x => x.Total > 0))
    Console.WriteLine($"{account.Asset} {account.Type}: {account.Available}");
```

## Spot Order Test

```csharp
var symbolInfo = await client.SpotApi.ExchangeData.GetSymbolAsync("BTC-USDT");
if (!symbolInfo.Success)
{
    Console.WriteLine($"Symbol info failed: {symbolInfo.Error}");
    return;
}

var order = await client.SpotApi.Trading.PlaceTestOrderAsync(
    symbol: "BTC-USDT",
    side: OrderSide.Buy,
    type: NewOrderType.Limit,
    quantity: 0.001m,
    price: 50000m,
    timeInForce: TimeInForce.GoodTillCanceled);

if (!order.Success)
{
    Console.WriteLine($"Order validation failed: {order.Error}");
    return;
}
```

Use `PlaceOrderAsync` only when the user explicitly wants a live order. For live examples, include cancellation cleanup when possible.

## Live Spot Order With Cleanup

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTC-USDT",
    side: OrderSide.Buy,
    type: NewOrderType.Limit,
    quantity: 0.001m,
    price: 40000m,
    timeInForce: TimeInForce.GoodTillCanceled);

if (!order.Success)
{
    Console.WriteLine($"Place order failed: {order.Error}");
    return;
}

try
{
    var status = await client.SpotApi.Trading.GetOrderAsync(order.Data.Id);
    if (status.Success)
        Console.WriteLine($"Order active: {status.Data.IsActive}");
}
finally
{
    await client.SpotApi.Trading.CancelOrderAsync(order.Data.Id);
}
```

## Futures Market Data

```csharp
const string symbol = "ETHUSDTM";

var contract = await client.FuturesApi.ExchangeData.GetContractAsync(symbol);
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

Console.WriteLine(ticker.Data.Price);
```

## Futures Position Read

```csharp
var positions = await client.FuturesApi.Account.GetPositionAsync("ETHUSDTM");
if (!positions.Success)
{
    Console.WriteLine($"Positions failed: {positions.Error}");
    return;
}

foreach (var position in positions.Data.Where(x => x.CurrentQuantity != 0))
    Console.WriteLine($"{position.Symbol}: {position.CurrentQuantity} at {position.AverageEntryPrice}");
```

## Futures Order

```csharp
var order = await client.FuturesApi.Trading.PlaceTestOrderAsync(
    symbol: "ETHUSDTM",
    side: OrderSide.Buy,
    type: NewOrderType.Market,
    leverage: 5m,
    quantity: 1,
    marginMode: FuturesMarginMode.Isolated);
```

Use `PlaceOrderAsync` only when the user explicitly wants a live futures order. For close-position examples, use the opposite side and `reduceOnly: true`.

## Unified Account

```csharp
var overview = await client.UnifiedApi.Account.GetAccountOverviewAsync();
if (!overview.Success)
{
    Console.WriteLine($"Unified account failed: {overview.Error}");
    return;
}

var balances = await client.UnifiedApi.Account.GetBalancesAsync();
```

Use native `UnifiedApi` methods for KuCoin Unified account workflows. SharedApis are exposed from `SpotApi.SharedClient` and `FuturesApi.SharedClient`, not `UnifiedApi`.

## Spot Websocket Subscription

```csharp
var socket = new KucoinSocketClient();

var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "BTC-USDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine($"Subscribe failed: {sub.Error}");
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Subscription methods return `WebSocketResult<UpdateSubscription>`.

## Private Spot Websocket Subscription

```csharp
var socket = new KucoinSocketClient(options =>
{
    options.ApiCredentials = new KucoinCredentials(
        "API_KEY",
        "API_SECRET",
        "API_PASSPHRASE");
});

var sub = await socket.SpotApi.SubscribeToOrderUpdatesAsync(
    onNewOrder: update => Console.WriteLine($"New order {update.Data.OrderId}"),
    onOrderData: update => Console.WriteLine($"Order {update.Data.OrderId}: {update.Data.Status}"),
    onTradeData: update => Console.WriteLine($"Trade {update.Data.TradeId}"));
```

## Futures Websocket Subscription

```csharp
var sub = await socket.FuturesApi.SubscribeToBookTickerUpdatesAsync(
    "ETHUSDTM",
    update => Console.WriteLine(update.Data.Price));
```

Authenticated futures socket streams require credentials with futures permissions.

## Unified Websocket Subscription

```csharp
var sub = await socket.UnifiedApi.SubscribeToOrderUpdatesAsync(
    UnifiedAccountType.Spot,
    update => Console.WriteLine(update.Data.OrderId));
```

Use the correct `UnifiedAccountType` for the account/trading mode.

## SharedApis Spot Ticker

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new KucoinRestClient().SpotApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");

var result = await tickers.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

Use native Kucoin APIs for Unified account, high-frequency spot trading, Earn, margin-specific borrow/lend features, and detailed futures account features.

## Dependency Injection

```csharp
services.AddKucoin(options =>
{
    options.ApiCredentials = new KucoinCredentials(
        "API_KEY",
        "API_SECRET",
        "API_PASSPHRASE");
});
```

Inject `IKucoinRestClient` and `IKucoinSocketClient`, or specific registered interfaces already used in the application.

## Error Handling

```csharp
var result = await client.SpotApi.ExchangeData.GetTickerAsync("BTC-USDT");
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
