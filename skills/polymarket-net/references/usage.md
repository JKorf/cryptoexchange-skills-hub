# Polymarket.Net Usage

Use these snippets as compact patterns for Polymarket-specific code. They assume:

```csharp
using CryptoExchange.Net.Authentication;
using CryptoExchange.Net.Objects;
using Polymarket.Net;
using Polymarket.Net.Clients;
using Polymarket.Net.Enums;
using Polymarket.Net.Objects.Models;
```

## Gamma Market Discovery

```csharp
var client = new PolymarketRestClient();

var markets = await client.GammaApi.GetMarketsAsync(closed: false, limit: 5);
if (!markets.Success)
{
    Console.WriteLine($"Could not load markets: {markets.Error}");
    return;
}

foreach (var market in markets.Data)
    Console.WriteLine($"{market.Question} | condition id: {market.MarketId}");
```

Use `GammaApi` when the user needs human-readable questions, events, tags, sports metadata, slugs, or search.

## Search

```csharp
var search = await client.GammaApi.SearchAsync("election", limitPerType: 3);
if (!search.Success)
{
    Console.WriteLine($"Search failed: {search.Error}");
    return;
}
```

## CLOB Token Order Book

```csharp
var tokenId = "TOKEN_ID";

var book = await client.ClobApi.ExchangeData.GetOrderBookAsync(tokenId);
if (!book.Success)
{
    Console.WriteLine($"Could not load order book: {book.Error}");
    return;
}

Console.WriteLine($"Bids: {book.Data.Bids.Length}, asks: {book.Data.Asks.Length}");
```

## Token Prices

```csharp
var midpoint = await client.ClobApi.ExchangeData.GetMidpointPriceAsync(tokenId);
if (!midpoint.Success)
{
    Console.WriteLine($"Could not load midpoint: {midpoint.Error}");
    return;
}

var buyPrice = await client.ClobApi.ExchangeData.GetPriceAsync(tokenId, OrderSide.Buy);
if (!buyPrice.Success)
{
    Console.WriteLine($"Could not load buy price: {buyPrice.Error}");
    return;
}
```

## Credentials

```csharp
var credentials = new PolymarketCredentials(
    new PolymarketL1Credential(
        SignType.Poly1271,
        "PRIVATE_KEY",
        "POLYMARKET_FUNDING_OR_DEPOSIT_ADDRESS"),
    new HMACPassCredential(
        "L2_API_KEY",
        "L2_API_SECRET",
        "L2_API_PASSPHRASE"));

var client = new PolymarketRestClient(options =>
{
    options.ApiCredentials = credentials;
});
```

Use `PolymarketCredentials` with L1 wallet credentials and L2 API key/secret/passphrase for private account, trading, and user stream examples.

## Derive L2 Credentials

```csharp
var client = new PolymarketRestClient(options =>
{
    options.ApiCredentials = new PolymarketCredentials()
        .WithL1(SignType.Poly1271, "PRIVATE_KEY", "POLYMARKET_FUNDING_OR_DEPOSIT_ADDRESS");
});

var l2 = await client.ClobApi.Account.GetOrCreateApiCredentialsAsync();
if (!l2.Success)
{
    Console.WriteLine($"Could not create L2 credentials: {l2.Error}");
    return;
}

client.UpdateL2Credentials(l2.Data);
```

## Allowance Check

```csharp
var allowance = await client.ClobApi.Account.GetBalanceAllowanceAsync(AssetType.Collateral);
if (!allowance.Success)
{
    Console.WriteLine($"Could not load allowance: {allowance.Error}");
    return;
}
```

For conditional token allowance, pass `AssetType.Conditional` and the `tokenId`.

## Live Order With Acceptance Check And Cleanup

```csharp
var order = await client.ClobApi.Trading.PlaceOrderAsync(
    tokenId: tokenId,
    side: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 10m,
    price: 0.42m,
    timeInForce: TimeInForce.GoodTillCanceled);

if (!order.Success)
{
    Console.WriteLine($"Order request failed: {order.Error}");
    return;
}

if (!order.Data.Success)
{
    Console.WriteLine($"Order rejected: {order.Data.Error}");
    return;
}

try
{
    var openOrders = await client.ClobApi.Trading.GetOpenOrdersAsync(tokenId: tokenId);
    if (openOrders.Success)
        Console.WriteLine($"Open orders: {openOrders.Data.Data.Length}");
}
finally
{
    await client.ClobApi.Trading.CancelOrderAsync(order.Data.OrderId);
}
```

`PlaceOrderAsync` places a live order. A successful `HttpResult` can still contain a rejected Polymarket order payload.

## Public Websocket Token Stream

```csharp
var socket = new PolymarketSocketClient();

var sub = await socket.ClobApi.SubscribeToTokenUpdatesAsync(
    new[] { tokenId },
    onPriceChangeUpdate: update => Console.WriteLine(update.Data),
    onBookUpdate: update => Console.WriteLine(update.Data),
    onLastTradePriceUpdate: update => Console.WriteLine(update.Data),
    onTickSizeUpdate: update => Console.WriteLine(update.Data),
    onBestBidAskUpdate: update => Console.WriteLine(update.Data));

if (!sub.Success)
{
    Console.WriteLine($"Token subscription failed: {sub.Error}");
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Subscription methods return `WebSocketResult<UpdateSubscription>`.

## Platform And Sports Streams

```csharp
var platform = await socket.ClobApi.SubscribeToPlatformUpdatesAsync(
    onNewMarketUpdate: update => Console.WriteLine(update.Data),
    onMarketResolvedUpdate: update => Console.WriteLine(update.Data));

var sports = await socket.ClobApi.SubscribeToSportsUpdatesAsync(
    update => Console.WriteLine(update.Data));
```

Check each subscription result before assuming the stream is live.

## Authenticated User Stream

```csharp
var socket = new PolymarketSocketClient(options =>
{
    options.ApiCredentials = credentials;
});

var sub = await socket.ClobApi.SubscribeToUserUpdatesAsync(
    onOrderUpdate: update => Console.WriteLine(update.Data),
    onTradeUpdate: update => Console.WriteLine(update.Data));

if (!sub.Success)
{
    Console.WriteLine($"User subscription failed: {sub.Error}");
    return;
}
```

## Dependency Injection

```csharp
using Microsoft.Extensions.DependencyInjection;
using Polymarket.Net.Interfaces;
using Polymarket.Net.Interfaces.Clients;

var services = new ServiceCollection();

services.AddPolymarket(options =>
{
    options.ApiCredentials = credentials;
});

using var provider = services.BuildServiceProvider();

var restClient = provider.GetRequiredService<IPolymarketRestClient>();
var socketClient = provider.GetRequiredService<IPolymarketSocketClient>();
var orderBookFactory = provider.GetRequiredService<IPolymarketOrderBookFactory>();
var userClientProvider = provider.GetRequiredService<IPolymarketUserClientProvider>();
```

## Local Order Book

```csharp
var orderBook = orderBookFactory.CreateClob(tokenId);

var start = await orderBook.StartAsync();
if (!start.Success)
{
    Console.WriteLine(start.Error);
    return;
}

await orderBook.StopAsync();
```

Local order books use CLOB token ids.

## User Client Provider

```csharp
userClientProvider.InitializeUserClient("alice", credentials);

var aliceRestClient = userClientProvider.GetRestClient("alice");
var aliceSocketClient = userClientProvider.GetSocketClient("alice");
```

Use the user client provider when an application manages separate Polymarket credentials per user.

## Error Handling

```csharp
var result = await client.ClobApi.ExchangeData.GetOrderBookAsync(tokenId);
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

REST failures, API validation errors, socket subscription failures, auth failures, and rate limits are returned on `.Error`. Do not rely on exceptions for normal API failures.
