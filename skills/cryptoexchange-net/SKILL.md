---
name: cryptoexchange-net
description: Build C#/.NET applications with the CryptoExchange.Net ecosystem, especially exchange-agnostic workflows using CryptoExchange.Net.SharedApis, CryptoClients.Net, HttpResult REST handling, WebSocketResult subscription handling, symbol normalization, DI setup, websocket lifecycle, package selection, and multi-exchange code. Use when the user asks to integrate multiple crypto exchanges, write code that can swap between Binance/Kucoin/OKX/Bybit/Kraken/etc., use shared clients, inspect balances/tickers/orders through common interfaces, build arbitrage/best-execution/portfolio tools, or choose the right CryptoExchange.Net-based package.
---

# CryptoExchange.Net

## Overview

Use this skill for application code that consumes the CryptoExchange.Net library ecosystem. `CryptoExchange.Net` itself is the base library; install exchange-specific packages such as `Binance.Net`, `JK.OKX.Net`, `Bybit.Net`, or the bundle package `CryptoClients.Net` for actual exchange access.

The main value of this skill is helping agents choose the right package and write exchange-agnostic code through `CryptoExchange.Net.SharedApis`.

## Choose The Right Approach

Use this decision order:

1. Single exchange with exchange-specific features: use that exchange package directly, for example `Binance.Net`.
2. Multiple exchanges with common workflows: use `CryptoExchange.Net.SharedApis`.
3. Broad exchange coverage from one dependency: use `CryptoClients.Net`.
4. Base-library or maintainer work: inspect `CryptoExchange.Net` source and existing exchange library patterns.

Do not install `CryptoExchange.Net` alone and expect to call exchange APIs. It provides the shared base abstractions used by the exchange packages.

Read `references/ecosystem.md` when choosing repository names, NuGet package IDs, or whether `CryptoClients.Net` is the better dependency.

## Package Examples

Install one or more exchange packages:

```bash
dotnet add package Binance.Net
dotnet add package JK.OKX.Net
dotnet add package Bybit.Net
```

Install the all-exchanges bundle when broad coverage is the explicit goal:

```bash
dotnet add package CryptoClients.Net
```

Use the exact NuGet package IDs from `references/ecosystem.md`; some packages do not match the repository name exactly, such as `JK.OKX.Net` and `KrakenExchange.Net`.

## SharedApis Quick Start

Use `.SharedClient` from exchange-specific REST or socket API surfaces:

```csharp
using Binance.Net.Clients;
using OKX.Net.Clients;
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient binance = new BinanceRestClient().SpotApi.SharedClient;
ISpotTickerRestClient okx = new OKXRestClient().UnifiedApi.SharedClient;

var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");

var result = await binance.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine($"[{binance.Exchange}] {result.Error}");
    return;
}

Console.WriteLine($"[{binance.Exchange}] {result.Data.Symbol}: {result.Data.LastPrice}");
```

Read `references/shared-apis.md` for interface families, websocket patterns, pagination, capability checks, and multi-exchange examples.

## Result Handling

REST calls return `HttpResult<T>`. Websocket subscription calls return `WebSocketResult<UpdateSubscription>`.

Rules:

- Always check `Success` before reading `Data`.
- Use `Error` for exchange/API/network/rate-limit failures.
- Use `Error.IsTransient` when deciding whether retry makes sense.
- Do not use exceptions for normal API errors; most API failures are result errors.
- Preserve cancellation tokens in production workflows.

```csharp
var result = await client.GetSpotTickerAsync(new GetTickerRequest(symbol), ct);
if (!result.Success)
{
    logger.LogWarning("{Exchange} failed: {Error}", client.Exchange, result.Error);
    return;
}
```

## Symbol Normalization

Use `SharedSymbol` for shared API calls:

```csharp
var spot = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");
var linearPerp = new SharedSymbol(TradingMode.PerpetualLinear, "BTC", "USDT");
```

Do not pass native symbols such as `BTCUSDT`, `BTC-USDT`, or exchange-specific contract names into shared request types. Let the shared client translate symbols to native format.

When using native exchange clients instead of SharedApis, use that exchange's native symbol format.

## Multi-Exchange Concurrency

Use `Task.WhenAll` for independent calls across exchanges:

```csharp
var clients = new List<ISpotTickerRestClient>
{
    new BinanceRestClient().SpotApi.SharedClient,
    new OKXRestClient().UnifiedApi.SharedClient,
    new BybitRestClient().V5Api.SharedClient,
};

var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");
var tasks = clients.Select(c => c.GetSpotTickerAsync(new GetTickerRequest(symbol))).ToArray();
var results = await Task.WhenAll(tasks);

for (var i = 0; i < clients.Count; i++)
{
    if (!results[i].Success)
    {
        Console.WriteLine($"[{clients[i].Exchange}] {results[i].Error}");
        continue;
    }

    Console.WriteLine($"[{clients[i].Exchange}] {results[i].Data.LastPrice}");
}
```

For repeated market data, prefer websocket shared interfaces where supported instead of tight REST polling loops.

## Websocket Lifecycle

Always close subscriptions on shutdown:

```csharp
ITickerSocketClient tickers = new BinanceSocketClient().SpotApi.SharedClient;

var sub = await tickers.SubscribeToTickerUpdatesAsync(
    new SubscribeTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")),
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await sub.Data.CloseAsync();
```

Keep websocket handlers fast. Push expensive work to a queue, channel, or background processor.

## Common Shared Interfaces

REST:

- `ISpotTickerRestClient`, `IFuturesTickerRestClient`
- `IBookTickerRestClient`, `IOrderBookRestClient`
- `ISpotOrderRestClient`, `IFuturesOrderRestClient`
- `IBalanceRestClient`, `IPositionRestClient`
- `ISpotSymbolRestClient`, `IFuturesSymbolRestClient`
- `IFeeRestClient`, `IDepositRestClient`, `IWithdrawalRestClient`, `ITransferRestClient`

Websocket:

- `ITickerSocketClient`, `ITickersSocketClient`
- `IBookTickerSocketClient`, `IOrderBookSocketClient`
- `ITradeSocketClient`, `IKlineSocketClient`
- `ISpotOrderSocketClient`, `IFuturesOrderSocketClient`
- `IBalanceSocketClient`, `IPositionSocketClient`, `IUserTradeSocketClient`

Not every exchange implements every shared interface. Code against the narrowest interface required and handle unsupported-operation failures.

## When To Use Native Clients

Switch from SharedApis to native clients when:

- The user asks for a specific exchange endpoint.
- The required option is missing from the shared request type.
- The response needs exchange-specific fields.
- The workflow depends on exchange-specific margin, account, listen-key, order, or position mode behavior.
- Production trading requires exact exchange semantics.

It is normal for an application to use SharedApis for common cross-exchange flows and native clients for exchange-specific features. Do not mix shared request/model types with native endpoint methods.

## Dependency Injection

Each exchange package has its own DI registration extension. Prefer the package's current documented pattern.

For example, Binance uses a single options delegate:

```csharp
services.AddBinance(options =>
{
    options.Rest.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
    options.Socket.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
});
```

Other exchange packages may use different option types or credential classes. Read that package's skill or local docs before generating DI code.

## Safety Rules

Read `references/safety.md` before generating code involving:

- API keys or secrets
- account balances
- trading
- leverage or futures
- transfers or withdrawals
- production bots

Default to read-only examples. For trading examples, prefer explicit limit orders, small placeholder quantities, and cleanup logic. Do not generate withdrawal code unless the user explicitly asks.

## References

- Read `references/shared-apis.md` for detailed cross-exchange interfaces, examples, pagination, and capability checks.
- Read `references/ecosystem.md` for repository names and NuGet package IDs.
- Read `references/safety.md` before account, trading, futures, withdrawal, or credential-handling code.

## Local Examples

When available in a sibling checkout, prefer these examples before inventing method names:

- `../CryptoExchange.Net/Examples/ai-friendly/01-shared-clients-quickstart.cs`
- `../CryptoExchange.Net/Examples/ai-friendly/02-multi-exchange-tickers.cs`
- `../CryptoExchange.Net/Examples/ai-friendly/03-cross-exchange-arbitrage-skeleton.cs`
- `../Binance.Net/Examples/ai-friendly/04-multi-exchange.cs`
