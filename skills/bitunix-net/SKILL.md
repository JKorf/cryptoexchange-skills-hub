---
name: bitunix-net
description: Build C#/.NET Bitunix integrations with Bitunix.Net, including SpotApi and FuturesApi REST clients, futures websocket subscriptions, spot websocket order queries, balances, deposits, withdrawals, transfers, spot and futures orders, leverage, margin and position modes, positions, TP/SL, BitunixCredentials authentication, dependency injection, local futures order books, user client providers, HttpResult and QueryResult handling, and Shared API V2 strict capabilities. Use for Bitunix market data, account or trading code, futures streams, private updates, error handling, or converting raw Bitunix API usage to idiomatic Bitunix.Net. Do not claim spot market websocket streams or a spot SharedApis ticker capability because the current library does not expose them.
---

# Bitunix.Net

## Overview

Use `Bitunix.Net` for Bitunix-specific C# code involving spot, perpetual futures, account data, trading, futures streams, socket order queries, leverage, margin and position modes, or Bitunix credentials. Use `cryptoexchange-net` when the same workflow must run across exchanges through `CryptoExchange.Net.SharedApis`.

Bitunix.Net has asymmetric socket surfaces: Futures provides market and private subscriptions, while Spot provides authenticated order placement and cancellation queries. Do not invent Spot market subscriptions.

## Setup

This skill targets the anticipated `Bitunix.Net` 0.1.0 release. Until that version is published, validate non-trivial code against the local Bitunix.Net source.

```bash
dotnet add package Bitunix.Net --version 0.1.0
```

```csharp
using Bitunix.Net;
using Bitunix.Net.Clients;
using Bitunix.Net.Enums;
```

Add `using CryptoExchange.Net.SharedApis;` only for portable SharedApis code.

## Client Roots

```csharp
var rest = new BitunixRestClient();
var socket = new BitunixSocketClient();
```

REST surfaces:

- `rest.SpotApi.ExchangeData`, `Account`, `Trading`, `SharedApi`
- `rest.FuturesApi.ExchangeData`, `Account`, `Trading`, `SharedApi`

Socket surfaces:

- `socket.SpotApi`: authenticated place/cancel Spot order queries and `SharedApi`
- `socket.FuturesApi`: public market streams, private user streams, and `SharedApi`

There is no `GeneralApi`, `UsdtFuturesApi`, `CoinFuturesApi`, margin root, options root, or Spot market stream surface. Read `references/api-surfaces.md` when choosing an endpoint or strict shared capability.

## Credentials And Results

Public market data does not require credentials. Private REST and socket operations require an API key and secret:

```csharp
var client = new BitunixRestClient(options =>
{
    options.ApiCredentials = new BitunixCredentials(
        "API_KEY",
        "API_SECRET");
});
```

Use placeholders, injected options, environment variables, or a secret store. Never hardcode real credentials.

- REST methods return `HttpResult<T>` or `HttpResult`.
- Futures subscriptions return `WebSocketResult<UpdateSubscription>`.
- Spot socket order operations return `QueryResult<T>` or `QueryResult`; they are query/response operations, not subscriptions.

Always check `Success` before reading `Data`.

```csharp
var price = await client.SpotApi.ExchangeData.GetLastPriceAsync("ETHUSDT");
if (!price.Success)
{
    Console.WriteLine(price.Error);
    return;
}

Console.WriteLine(price.Data);
```

## Symbols And Order Semantics

- Native Spot and Futures symbols use compact names such as `ETHUSDT`.
- Spot orders use `OrderSide` and `OrderType`; direct placement accepts quantity and optional price, but no time-in-force or client order ID.
- Futures orders use `OrderSide`, `OrderType`, optional `TradeSide`, `TimeInForce`, position/client IDs, reduce-only, and inline TP/SL fields.
- `TradeSide.Open` and `TradeSide.Close` express hedge-mode intent; `tradeSide` is required by Bitunix in hedge mode.
- Futures balances and leverage operations require a margin asset such as `USDT`.

Validate symbols, precision, limits, position mode, margin mode, and current exposure before production trading.

## REST Patterns

Spot limit order:

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "ETHUSDT",
    orderSide: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 0.01m,
    price: 2000m);

if (!order.Success)
{
    Console.WriteLine(order.Error);
    return;
}
```

Futures leverage and limit order:

```csharp
var leverage = await client.FuturesApi.Account.SetLeverageAsync(
    symbol: "ETHUSDT",
    marginAsset: "USDT",
    leverage: 5);

if (!leverage.Success)
{
    Console.WriteLine(leverage.Error);
    return;
}

var order = await client.FuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETHUSDT",
    side: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 0.1m,
    price: 2000m,
    tradeSide: TradeSide.Open,
    timeInForce: TimeInForce.GoodTillCanceled);
```

These placement and leverage calls change live account state. Read `references/safety.md` first.

## Socket Patterns

Futures market subscription:

```csharp
var socket = new BitunixSocketClient();
var sub = await socket.FuturesApi.SubscribeToTickerUpdatesAsync(
    "ETHUSDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Private Futures updates use credentials configured directly on `BitunixSocketClient`; there is no listen-key lifecycle. `SubscribeToUserUpdatesAsync` accepts optional balance, order, position, and TP/SL callbacks.

Spot socket order placement is a query:

```csharp
var result = await socket.SpotApi.PlaceOrderAsync(
    "ETHUSDT",
    OrderSide.Buy,
    OrderType.Limit,
    0.01m,
    price: 2000m);

if (!result.Success)
    Console.WriteLine(result.Error);
```

Do not call `UnsubscribeAsync` for a Spot `QueryResult`.

## Shared API V2

Use strict SharedApis only when portability matters. Depend on the narrow capability the workflow needs:

```csharp
using CryptoExchange.Net.SharedApis;

using var client = new BitunixRestClient();
IGetTickerRest ticker = client.FuturesApi.SharedApi;

var result = await ticker.GetTickerAsync(
    new GetTickerRequest(
        new SharedSymbol(
            TradingMode.PerpetualLinear,
            "ETH",
            "USDT")));
```

Use `IBitunixSharedApiClient` as the aggregate. Its properties are `SpotRest`, `FuturesRest`, `SpotSocket`, and `FuturesSocket`. Use runtime capability lookup only when capability, transport, or trading mode is selected dynamically.

Capability boundaries matter:

- Spot REST supports symbols, assets, balances, deposits, withdrawals, klines, order books, and Spot orders.
- Spot REST does not implement `IGetTickerRest`; use direct `GetLastPriceAsync` for a Spot price.
- Spot socket SharedApi supports order placement and cancellation queries, not streams.
- Futures REST and socket surfaces provide the broader market, order, position, and subscription capabilities.

Do not mix Bitunix-native request/model types with SharedApis request/model types.

## Dependency Injection And Environments

```csharp
services.AddBitunix(options =>
{
    options.ApiCredentials =
        new BitunixCredentials("API_KEY", "API_SECRET");
});
```

Inject `IBitunixRestClient`, `IBitunixSocketClient`, `IBitunixSharedApiClient`, `IBitunixOrderBookFactory`, `IBitunixTrackerFactory`, or `IBitunixUserClientProvider` as needed.

`BitunixEnvironment.Live` is the only built-in environment. `CreateCustom(...)` requires Spot REST, Spot socket, Futures REST, and Futures socket addresses. Do not invent a built-in testnet.

## Safety Rules

- Read `references/safety.md` before generating credentials, withdrawals, transfers, trading, leverage, margin/position mode, TP/SL, close-position, private socket, or user-client code.
- Prefer read-only market and account examples when mutation is unnecessary.
- Mark all orders, cancellations, leverage/mode changes, transfers, withdrawals, and position operations as live account actions.
- Keep socket handlers fast and unsubscribe successful Futures subscriptions during shutdown.
- Retry only transient failures with bounded backoff; reconcile uncertain order outcomes before retrying placement.

## References

- Read `references/api-surfaces.md` for endpoint groups, result types, socket operations, SharedApis capabilities, DI, environments, and symbol rules.
- Read `references/usage.md` for practical market-data, account, order, socket, SharedApis, and DI patterns.
- Read `references/safety.md` for credential, live-trading, futures, transfer, withdrawal, socket, and retry guardrails.
- In a maintainer checkout, read `../Bitunix.Net/Bitunix.Net/Interfaces/Clients/**` for exact current signatures and `../Bitunix.Net/llms-full.txt` for the detailed current API map.
