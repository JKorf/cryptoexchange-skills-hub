---
name: cryptoclients-net
description: Build C#/.NET multi-exchange integrations with CryptoClients.Net, focusing on aggregate CryptoExchange.Net SharedApis REST and websocket workflows, capability discovery, shared symbols and models, per-exchange results, async fan-out, credentials, dependency injection, cross-exchange order books, trackers, and user client providers. Also use for full native exchange API access through ExchangeRestClient and ExchangeSocketClient properties when shared interfaces do not expose the required endpoint or model.
---

# CryptoClients.Net

## Overview

Use `CryptoClients.Net` when an application needs broad exchange coverage from one package. Prefer its aggregate SharedApis methods for portable workflows, then use the exposed native clients for exchange-specific endpoints or models.

Use an individual exchange skill instead when the application targets only one exchange or needs substantial exchange-specific behavior.

## Setup

```bash
dotnet add package CryptoClients.Net
```

```csharp
using CryptoClients.Net;
using CryptoClients.Net.Enums;
using CryptoClients.Net.Interfaces;
using CryptoClients.Net.Models;
using CryptoExchange.Net.Objects;
using CryptoExchange.Net.SharedApis;
```

## Aggregate Clients

```csharp
var rest = new ExchangeRestClient();
var socket = new ExchangeSocketClient();
```

Use `IExchangeRestClient` and `IExchangeSocketClient` with dependency injection.

## SharedApis First

Describe markets with `SharedSymbol` and use shared request/model types:

```csharp
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");
var request = new GetTickerRequest(symbol);
var exchanges = new[] { Exchange.Binance, Exchange.Bybit, Exchange.Kraken, Exchange.OKX };

var results = await rest.GetSpotTickerAsync(request, exchanges);
foreach (var result in results)
{
    if (!result.Success)
    {
        Console.WriteLine($"{result.Exchange}: {result.Error}");
        continue;
    }

    Console.WriteLine($"{result.Exchange}: {result.Data.LastPrice}");
}
```

Aggregate methods only call shared clients that advertise support for the requested operation. Do not assume every selected exchange implements every SharedApis interface.

## Result Shapes

- One-exchange REST aggregate call: `HttpResult<T>`
- Multi-exchange REST aggregate call: `HttpResult<T>[]`
- Streaming fan-out: `IAsyncEnumerable<HttpResult<T>>`
- One-exchange socket subscription: `WebSocketResult<UpdateSubscription>`
- Multi-exchange socket subscription: `WebSocketResult<UpdateSubscription>[]`
- Some symbol/capability helpers: `ExchangeCallResult<T>`

Check each result independently. One exchange failure must not hide successful results from other exchanges.

## Capability Discovery

Get one nullable shared client when routing dynamically:

```csharp
var tickerClient = rest.GetSpotTickerClient(Exchange.Binance);
if (tickerClient != null)
{
    var result = await tickerClient.GetSpotTickerAsync(
        new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "ETH", "USDT")));
}
```

Use plural getters such as `GetSpotTickerClients()`, `GetOrderBookClients(mode)`, or `GetFuturesOrderClients(mode)` to enumerate supported clients. Use `GetExchangeSharedClients(exchange, tradingMode)` to inspect all shared interfaces exposed by one exchange.

## Full Exchange API Access

The aggregate clients expose the bundled native clients directly:

```csharp
var binance = await rest.Binance.SpotApi.ExchangeData.GetTickerAsync("ETHUSDT");
var kucoin = await rest.Kucoin.SpotApi.ExchangeData.GetTickerAsync("ETH-USDT");
var bitfinex = await rest.Bitfinex.ExchangeApi.ExchangeData.GetTickerAsync("tETHUSD");
```

Use native access when the shared request lacks an option, the shared model omits required fields, or the endpoint has no shared interface. Follow the corresponding exchange skill for roots, credentials, symbols, and safety.

Current source exposes direct REST clients for all bundled libraries, including CoinGecko and Polymarket, and direct socket clients for socket-capable libraries. CoinGecko and Polymarket are not currently part of the aggregate `_sharedClients` list, so use their direct properties.

## Websocket Fan-Out

```csharp
var subscriptions = await socket.SubscribeToTickerUpdatesAsync(
    new SubscribeTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")),
    update => Console.WriteLine($"{update.Exchange}: {update.Data.LastPrice}"),
    new[] { Exchange.Binance, Exchange.Bybit, Exchange.OKX });

foreach (var subscription in subscriptions.Where(x => !x.Success))
    Console.WriteLine($"{subscription.Exchange}: {subscription.Error}");

await socket.UnsubscribeAllAsync();
```

Keep handlers fast. Close individual successful subscriptions with `subscription.Data.CloseAsync()` or tear down all aggregate connections with `UnsubscribeAllAsync()`.

## Credentials

Use typed `ExchangeCredentials` when exchanges are known at compile time. Use `DynamicCredentials` only when the exchange is selected at runtime, after calling `ExchangeCredentials.GetDynamicCredentialInfo(...)` to discover required fields.

Do not use the obsolete `SetApiCredentials(exchange, apiKey, apiSecret, apiPass)` overload; it cannot represent every exchange credential shape.

## Dependency Injection

```csharp
services.AddCryptoClients(options =>
{
    options.RequestTimeout = TimeSpan.FromSeconds(15);
});
```

`AddCryptoClients` registers native exchange clients plus `IExchangeRestClient`, `IExchangeSocketClient`, `IExchangeOrderBookFactory`, `IExchangeTrackerFactory`, and `IExchangeUserClientProvider`. It also accepts per-exchange option delegates and an `IConfiguration` overload.

## Order Books And Trackers

Use `IExchangeOrderBookFactory` for individual books or `CreateCrossExchange(...)` for a combined cross-exchange book. Use `IExchangeTrackerFactory` for kline, trade, spot-user-data, and futures-user-data trackers. Factory methods can return `null` when an exchange or trading mode is unsupported.

## References

- Read `references/api-surfaces.md` for aggregate methods, direct clients, discovery, DI, credentials, factories, and result types.
- Read `references/usage.md` for practical aggregate, native, websocket, credentials, and DI patterns.
- Read `references/safety.md` before credentials, private data, orders, leverage, transfers, withdrawals, or production fan-out.
- In a maintainer checkout, read `../CryptoClients.Net/Examples/ai-friendly/` for current examples. When an example and a public interface differ, follow the current interface signature and result type.
