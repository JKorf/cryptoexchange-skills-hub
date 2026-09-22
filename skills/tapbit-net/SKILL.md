---
name: tapbit-net
description: Build C#/.NET Tapbit spot integrations with Tapbit.Net, including SpotApi REST market data, asset metadata, balances, limit and batch orders, TapbitCredentials, slash-separated symbols, HttpResult handling, Shared API V2 strict capabilities and aggregates, dependency injection, multi-user REST clients, and REST-polled spot user-data tracking. Use when the user asks for Tapbit spot market data, Tapbit balances or order code, Tapbit error handling, or converting raw Tapbit REST calls to idiomatic Tapbit.Net. Do not use for Tapbit futures or websocket workflows because version 1.0.0 does not expose those clients.
---

# Tapbit.Net

## Overview

Use `Tapbit.Net` for Tapbit-specific C# spot REST integrations. Prefer this skill over `cryptoexchange-net` when the user needs native Tapbit endpoints, credentials, symbols, models, batch orders, or Tapbit-specific capability boundaries.

Use `cryptoexchange-net` when the same code must run across exchanges through `CryptoExchange.Net.SharedApis`.

Tapbit.Net 1.0.0 is spot REST only. Do not invent `TapbitSocketClient`, futures APIs, websocket subscriptions, market-order placement overloads, deposit/withdrawal/transfer endpoints, or user-trade endpoints.

## Setup

This skill targets the anticipated `Tapbit.Net` 1.0.0 release. Until that version is published, validate non-trivial code against the local Tapbit.Net source.

```bash
dotnet add package Tapbit.Net --version 1.0.0
```

Use these namespaces as needed:

```csharp
using CryptoExchange.Net.Objects;
using Tapbit.Net;
using Tapbit.Net.Clients;
using Tapbit.Net.Enums;
using Tapbit.Net.Objects.Models;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Client Root

```csharp
var client = new TapbitRestClient();
```

The complete native exchange surface is:

- `client.SpotApi.ExchangeData`: server time, symbols, order books, tickers, klines, recent trades, assets and networks
- `client.SpotApi.Account`: all balances or one balance by asset
- `client.SpotApi.Trading`: limit-order placement, batch placement, cancellation, batch cancellation, open/closed orders, and order lookup
- `client.SpotApi.SharedApi`: shared spot REST interfaces

There is no native socket or futures client. Read `references/api-surfaces.md` before selecting less common methods or tracker features.

## Credentials

Public market data requires no credentials:

```csharp
var client = new TapbitRestClient();
```

Account and trading methods require an API key and secret:

```csharp
var client = new TapbitRestClient(options =>
{
    options.ApiCredentials = new TapbitCredentials("API_KEY", "API_SECRET");
});
```

Tapbit credentials do not use a passphrase. Use placeholders, injected configuration, environment variables, or a secrets provider; never hardcode real credentials.

## Results

REST calls return `HttpResult<T>`. Check `Success` before reading `Data`.

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTC/USDT");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

Batch placement and cancellation return `HttpResult<CallResult<TapbitOrderId>[]>`. Check both the outer request and every inner item.

Use `Error.Code`, `Message`, `ErrorType`, and `IsTransient` for diagnostics. Retry only transient errors with bounded backoff. Reconcile uncertain order outcomes before retrying a placement request.

## Symbols And Orders

- Native symbols use `BASE/QUOTE`, such as `BTC/USDT`.
- `TapbitExchange.FormatSymbol("BTC", "USDT", TradingMode.Spot)` returns `BTC/USDT`.
- Native order IDs are `long`; `GetOrderAsync` and `CancelOrderAsync` take only an order ID.
- The public placement surface supports limit orders only and therefore has no order-type parameter.
- Query symbol metadata before trading and validate price precision, quantity precision, minimum quantity, minimum notional, price-fluctuation bounds, and balance.

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTC/USDT",
    side: OrderSide.Buy,
    quantity: 0.001m,
    price: 50000m);

if (!order.Success)
{
    Console.WriteLine(order.Error);
    return;
}

Console.WriteLine(order.Data.OrderId);
```

The library has no test-order method. Valid authenticated placement calls can create live orders.

## Shared API V2

Use Shared APIs only when portability matters. Depend on the narrow capability needed by the workflow:

```csharp
using CryptoExchange.Net.SharedApis;

using var client = new TapbitRestClient();
IGetTickerRest ticker = client.SpotApi.SharedApi;

var result = await ticker.GetTickerAsync(
    new GetTickerRequest(
        new SharedSymbol(TradingMode.Spot, "BTC", "USDT")));
```

Use `ITapbitSharedApiClient` as the exchange aggregate. Its aggregate properties—`SpotRest`—expose the supported Shared API surfaces at compile time. Use `GetCapability` only when the capability, trading mode, or transport is selected dynamically.

Tapbit currently exposes no Shared API V2 socket surface. Do not mix Tapbit-native request/model types with Shared API request/model types.

## Dependency Injection And Multi-User Clients

```csharp
services.AddTapbit(options =>
{
    options.Rest.ApiCredentials =
        new TapbitCredentials("API_KEY", "API_SECRET");
});
```

Inject `ITapbitRestClient`, `ITapbitUserClientProvider`, or `ITapbitTrackerFactory`. Reuse clients in long-running applications.

`TapbitEnvironment.Live` is the only built-in environment. Use `TapbitEnvironment.CreateCustom(...)` only for an intentional custom endpoint; do not invent a testnet.

## Tracker Boundary

Tapbit.Net has no kline tracker or public-trade tracker because it has no websocket client. `CanCreateKlineTracker(...)` and `CanCreateTradeTracker(...)` return false, and the corresponding create methods throw.

`ITapbitTrackerFactory.CreateUserSpotDataTracker(...)` provides REST-polled balance and order tracking. It requires at least one `TrackedSymbols` entry and `TrackTrades = false`; Tapbit does not expose user-trade history.

## Safety Rules

- Read `references/safety.md` before generating credentials, balance, order, batch, user-provider, or tracker code.
- Prefer read-only account and order-query examples when mutation is unnecessary.
- Mark order placement and cancellation as live account operations.
- Validate each batch result independently and reconcile partial success.
- Never expose real API keys or secrets.

## References

- Read `references/api-surfaces.md` for native methods, SharedApis coverage, environments, DI registrations, result types, and tracker support.
- Read `references/usage.md` for practical market-data, balance, order, batch, SharedApis, DI, user-provider, and tracker patterns.
- Read `references/safety.md` for credentials, live orders, batch semantics, retries, and capability guardrails.
- In a maintainer checkout, read `../Tapbit.Net/Examples/ai-friendly/` and `../Tapbit.Net/docs/ai-api-map.md` for current compilable examples and method routing.
