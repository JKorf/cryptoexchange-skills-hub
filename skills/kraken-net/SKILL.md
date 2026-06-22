---
name: kraken-net
description: Build C#/.NET Kraken integrations with Kraken.Net, including KrakenExchange.Net package setup, SpotApi, FuturesApi, Earn endpoints, REST clients, websocket subscriptions, spot websocket request/order methods, spot market data, futures market data, account balances, deposits, withdrawals, transfers, Earn allocations, spot order placement/cancellation/editing, futures leverage, futures positions, futures order placement/cancellation/editing, KrakenCredentials with separate spot and futures HMAC credentials, Kraken REST spot symbols, Kraken spot websocket symbols, Kraken futures symbols, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, QueryResult spot websocket request handling, ExchangeCallResult shared helper handling, trackers, and SharedApis access. Use when the user asks for Kraken spot market data, Kraken account or trading code, Kraken futures, Kraken websocket updates, Kraken Earn, Kraken error handling, or converting raw Kraken API usage to idiomatic Kraken.Net.
---

# Kraken.Net

## Overview

Use `Kraken.Net` for Kraken-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for Kraken spot endpoints, futures endpoints, Earn endpoints, user streams, websocket order requests, leverage, positions, or Kraken credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the package:

```bash
dotnet add package KrakenExchange.Net
```

Use these namespaces in examples:

```csharp
using CryptoExchange.Net.Authentication;
using CryptoExchange.Net.Objects;
using Kraken.Net;
using Kraken.Net.Clients;
using Kraken.Net.Enums;
```

Add `using CryptoExchange.Net.SharedApis;` only for exchange-agnostic SharedApis code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new KrakenRestClient();
```

Create socket clients for websocket subscriptions and spot websocket order requests:

```csharp
var socket = new KrakenSocketClient();
```

Primary REST API surfaces:

- `rest.SpotApi.ExchangeData`, `rest.SpotApi.Account`, `rest.SpotApi.Trading`, `rest.SpotApi.Earn`, `rest.SpotApi.SharedClient`
- `rest.FuturesApi.ExchangeData`, `rest.FuturesApi.Account`, `rest.FuturesApi.Trading`, `rest.FuturesApi.SharedClient`

Primary socket API surfaces:

- `socket.SpotApi`: spot public streams, private streams, and websocket v2 order request methods
- `socket.FuturesApi`: futures public and private streams

Kraken.Net does not expose Binance-style roots such as `UsdFuturesApi`, `CoinFuturesApi`, or `UnifiedApi`. Use `FuturesApi` for Kraken futures.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new KrakenRestClient();
```

Kraken.Net supports separate HMAC credential slots for spot and futures. Prefer the separate-credential constructor:

```csharp
var spotClient = new KrakenRestClient(options =>
{
    options.ApiCredentials = new KrakenCredentials(
        new HMACCredential("SPOT_KEY", "SPOT_SECRET"),
        null);
});

var futuresClient = new KrakenRestClient(options =>
{
    options.ApiCredentials = new KrakenCredentials(
        null,
        new HMACCredential("FUTURES_KEY", "FUTURES_SECRET"));
});
```

Use placeholders or environment variables in generated code. Do not use the obsolete `KrakenCredentials(string apiKey, string secret)` constructor in new examples.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Spot websocket request/order methods return `QueryResult<T>` or `QueryResult`. Some SharedApis symbol helper methods return `ExchangeCallResult<T>`. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("ETHUSDT");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.Values.First().LastTrade.Price);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols

- REST spot symbols use Kraken REST pair names such as `ETHUSDT`.
- Spot websocket v2 subscriptions use websocket names such as `ETH/USDT`.
- Fetch spot symbols with `SpotApi.ExchangeData.GetSymbolsAsync(...)` and use `KrakenSymbol.WebsocketName` when converting REST symbols to websocket symbols.
- Futures symbols use Kraken futures symbols such as `PF_ETHUSD`.
- `KrakenExchange.FormatSymbol("ETH", "USDT", TradingMode.Spot)` returns `ETHUSDT`.
- `KrakenExchange.FormatSymbol("ETH", "USD", TradingMode.PerpetualLinear)` returns `PF_ETHUSD`.

## REST Patterns

Public spot ticker:

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("ETHUSDT");
```

Authenticated spot balances:

```csharp
var balances = await client.SpotApi.Account.GetBalancesAsync();
```

Safe spot order validation:

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "ETHUSDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.01m,
    price: 1000m,
    validateOnly: true);
```

Futures market data, leverage, and order:

```csharp
var ticker = await client.FuturesApi.ExchangeData.GetTickerAsync("PF_ETHUSD");

await client.FuturesApi.Trading.SetLeverageAsync("PF_ETHUSD", 5);

var order = await client.FuturesApi.Trading.PlaceOrderAsync(
    symbol: "PF_ETHUSD",
    side: OrderSide.Buy,
    type: FuturesOrderType.Limit,
    quantity: 0.1m,
    price: 1000m);
```

Use `validateOnly: true` for spot examples that should not place live orders. Futures order examples are live unless you intentionally avoid private order placement.

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new KrakenSocketClient();

var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "ETH/USDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Futures public streams use futures symbols:

```csharp
var sub = await socket.FuturesApi.SubscribeToTickerUpdatesAsync(
    "PF_ETHUSD",
    update => Console.WriteLine(update.Data));
```

Native spot websocket order request methods, such as `socket.SpotApi.PlaceOrderAsync(...)`, return `QueryResult<T>`, not `HttpResult<T>`. Futures socket surface exposes subscriptions, not the spot websocket v2 order request set.

Use `UnsubscribeAsync` or `UnsubscribeAllAsync` on shutdown. Do not leave example subscriptions running.

## SharedApis

Use SharedApis only when portability matters:

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new KrakenRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "ETH", "USDT")));
```

Kraken.Net exposes SharedApis from both `SpotApi.SharedClient` and `FuturesApi.SharedClient` on REST and socket clients.

Do not mix Kraken-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddKraken(options =>
{
    options.ApiCredentials = new KrakenCredentials(
        new HMACCredential("SPOT_KEY", "SPOT_SECRET"),
        new HMACCredential("FUTURES_KEY", "FUTURES_SECRET"));
});
```

Inject `IKrakenRestClient` and `IKrakenSocketClient`, or the registered Kraken REST/socket client interfaces used by the target project. `AddKraken` also registers supported SharedApis interfaces from `SpotApi.SharedClient` and `FuturesApi.SharedClient`.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, Earn allocation, transfer, withdrawal, websocket order, or credential code.
- Prefer read-only examples for account inspection.
- Prefer `validateOnly: true` in spot order examples that only demonstrate parameter validation.
- Futures orders and leverage changes are live account operations; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups, client roots, SharedApis, result types, and symbol formats.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, futures, Earn, transfer, withdrawal, websocket, tracker, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../Kraken.Net/Examples/ai-friendly/` for complete compilable examples.
