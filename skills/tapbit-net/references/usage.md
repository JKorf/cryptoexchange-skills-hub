# Tapbit.Net Usage

Use these snippets as patterns for the anticipated 1.0.0 release. Keep code async, pass cancellation tokens in services, and check every result.

## Base Imports

```csharp
using CryptoExchange.Net.Objects;
using Tapbit.Net;
using Tapbit.Net.Clients;
using Tapbit.Net.Enums;
using Tapbit.Net.Objects.Models;
```

Add SharedApis, tracker, DI, or interface namespaces only when the workflow needs them.

## Public Ticker

```csharp
var client = new TapbitRestClient();
var ticker = await client.SpotApi.ExchangeData
    .GetTickerAsync("BTC/USDT");

if (!ticker.Success)
{
    Console.WriteLine($"Ticker failed: {ticker.Error}");
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

## Symbol Rules Before Trading

```csharp
var symbol = await client.SpotApi.ExchangeData
    .GetSymbolAsync("BTC/USDT");

if (!symbol.Success)
{
    Console.WriteLine($"Symbol lookup failed: {symbol.Error}");
    return;
}

Console.WriteLine($"Price decimals: {symbol.Data.PricePrecision}");
Console.WriteLine($"Quantity decimals: {symbol.Data.QuantityPrecision}");
Console.WriteLine($"Minimum quantity: {symbol.Data.MinQuantity}");
Console.WriteLine($"Minimum notional: {symbol.Data.MinNotional}");
```

Validate precision, minimums, price-fluctuation bounds, current price, and balance before submitting an order.

## Authenticated Balance

```csharp
var client = new TapbitRestClient(options =>
{
    options.ApiCredentials =
        new TapbitCredentials("API_KEY", "API_SECRET");
});

var balance = await client.SpotApi.Account.GetBalanceAsync("USDT");
if (!balance.Success)
{
    Console.WriteLine($"Balance failed: {balance.Error}");
    return;
}

Console.WriteLine($"Available: {balance.Data.Available}");
```

## Limit Order, Lookup, And Cancellation

The following calls can mutate a live account:

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTC/USDT",
    side: OrderSide.Buy,
    quantity: 0.001m,
    price: 50000m);

if (!order.Success)
{
    Console.WriteLine($"Placement failed: {order.Error}");
    return;
}

long orderId = order.Data.OrderId;

var status = await client.SpotApi.Trading.GetOrderAsync(orderId);
if (!status.Success)
{
    Console.WriteLine($"Lookup failed: {status.Error}");
    return;
}

Console.WriteLine($"{status.Data.Status}, filled {status.Data.QuantityFilled}");

var cancel = await client.SpotApi.Trading.CancelOrderAsync(orderId);
if (!cancel.Success)
    Console.WriteLine($"Cancellation failed: {cancel.Error}");
```

Do not add an order-type parameter. The placement API is limit-only.

## Batch Orders

```csharp
var requests = new[]
{
    new TapbitOrderRequest
    {
        Symbol = "BTC/USDT",
        Side = OrderSide.Buy,
        Quantity = 0.001m,
        Price = 50000m
    },
    new TapbitOrderRequest
    {
        Symbol = "ETH/USDT",
        Side = OrderSide.Buy,
        Quantity = 0.01m,
        Price = 2000m
    }
};

var batch = await client.SpotApi.Trading
    .PlaceMultipleOrdersAsync(requests);

if (!batch.Success)
{
    Console.WriteLine($"Batch request failed: {batch.Error}");
    return;
}

foreach (var item in batch.Data)
{
    if (item.Success)
        Console.WriteLine($"Placed {item.Data.OrderId}");
    else
        Console.WriteLine($"Item failed: {item.Error}");
}
```

A failed item does not roll back successful siblings. Record successful IDs and reconcile before cancellation or retry.

## SharedApis Ticker

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers =
    new TapbitRestClient().SpotApi.SharedClient;

var request = new GetTickerRequest(
    new SharedSymbol(TradingMode.Spot, "BTC", "USDT"));

var result = await tickers.GetSpotTickerAsync(request);
if (!result.Success)
{
    Console.WriteLine($"[{tickers.Exchange}] {result.Error}");
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

## Shared Symbol Catalog

```csharp
ISpotSymbolRestClient symbols =
    new TapbitRestClient().SpotApi.SharedClient;

var fetch = await symbols.GetSpotSymbolsAsync(new GetSymbolsRequest());
if (!fetch.Success)
{
    Console.WriteLine(fetch.Error);
    return;
}

var catalog = symbols.SpotSymbolCatalog;
Console.WriteLine(catalog?.Symbols.Count ?? 0);
```

Fetch successfully before reading `SpotSymbolCatalog`.

## Dependency Injection

```csharp
using Microsoft.Extensions.DependencyInjection;
using Tapbit.Net.Interfaces.Clients;

services.AddTapbit(options =>
{
    options.Rest.ApiCredentials =
        new TapbitCredentials("API_KEY", "API_SECRET");
});
```

Inject `ITapbitRestClient` and reuse it.

## Multi-User REST Clients

```csharp
using Tapbit.Net.Interfaces.Clients;

ITapbitUserClientProvider provider = new TapbitUserClientProvider();

provider.InitializeUserClient(
    "user-42",
    new TapbitCredentials("API_KEY", "API_SECRET"));

var userClient = provider.GetRestClient("user-42");
var balances = await userClient.SpotApi.Account.GetBalancesAsync();
```

Clear the user's cached client when credentials change. Never encode credentials in the user identifier.

## REST-Polled User Data Tracker

```csharp
using CryptoExchange.Net.SharedApis;
using CryptoExchange.Net.Trackers.UserData.Objects;
using Tapbit.Net.Interfaces;

ITapbitTrackerFactory trackerFactory = new TapbitTrackerFactory();

var config = new SpotUserDataTrackerConfig
{
    TrackedSymbols = new[]
    {
        new SharedSymbol(TradingMode.Spot, "BTC", "USDT")
    },
    TrackTrades = false,
    OnlyTrackProvidedSymbols = true
};

var tracker = trackerFactory.CreateUserSpotDataTracker(
    "user-42",
    new TapbitCredentials("API_KEY", "API_SECRET"),
    config);
```

This tracker polls REST balances and orders. Tapbit.Net cannot create kline or public-trade trackers, and it cannot track user trades.

## Error Handling And Retry

Tapbit maps common error categories such as unknown symbol/order, order rate limits, invalid quantity or price, insufficient balance, and invalid parameters. Only the system-busy mapping is transient by default.

Use bounded retry logic only when `result.Error?.IsTransient == true`. Authentication, validation, balance, and rejected-order errors require correction or reconciliation rather than blind retry.
