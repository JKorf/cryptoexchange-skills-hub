---
name: xt-net
description: Build C#/.NET XT integrations with XT.Net, including SpotApi, USDT-M and Coin-M futures REST APIs, the combined futures websocket API, market data, balances, orders, positions, leverage, private streams, SharedApis, dependency injection, local order books, trackers, HttpResult handling, and WebSocketResult handling. Use when the user asks for XT market data, account, spot or futures trading, websocket updates, private streams, or idiomatic XT.Net code.
---

# XT.Net

## Overview

Use `XT.Net` for XT-specific spot, USDT-M futures, Coin-M futures, account, trading, and websocket workflows. Use `cryptoexchange-net` for portable multi-exchange workflows through SharedApis.

## Setup

```bash
dotnet add package XT.Net
```

```csharp
using XT.Net;
using XT.Net.Clients;
using XT.Net.Enums;
```

## Client Roots

```csharp
var rest = new XTRestClient();
var socket = new XTSocketClient();
```

REST has three product roots:

- `rest.SpotApi`
- `rest.UsdtFuturesApi`
- `rest.CoinFuturesApi`

Websocket has two roots:

- `socket.SpotApi`
- `socket.FuturesApi`: combined futures socket surface for XT futures

Do not invent separate `UsdtFuturesApi` or `CoinFuturesApi` roots on `XTSocketClient`.

## Credentials

Private calls use an API key and secret only:

```csharp
var client = new XTRestClient(options =>
{
    options.ApiCredentials = new XTCredentials("API_KEY", "API_SECRET");
});
```

Do not add a passphrase. Keep real credentials in secret storage.

## Result Handling

- REST: `HttpResult<T>` or `HttpResult`
- Socket subscriptions: `WebSocketResult<UpdateSubscription>`
- Shared helpers: `ExchangeCallResult<T>`

Always inspect `Success` before using `Data` and retry only transient failures.

## Symbols And Products

- Native spot symbols are lowercase with an underscore, for example `eth_usdt`.
- Native futures symbols are uppercase with an underscore, for example `ETH_USDT`.
- Spot orders require both `TimeInForce` and `BusinessType`.
- Use `BusinessType.Spot` for spot and `BusinessType.Leverage` for margin where supported.
- XT spot market buys use `quoteQuantity`; market sells use `quantity`.
- Futures orders require a `PositionSide`.

Validate current symbol metadata, status, precision, minimums, and product type before trading.

## REST Patterns

```csharp
var ticker = await rest.SpotApi.ExchangeData.GetTickersAsync("eth_usdt");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}
```

Live spot order:

```csharp
var order = await rest.SpotApi.Trading.PlaceOrderAsync(
    symbol: "eth_usdt",
    orderSide: OrderSide.Buy,
    orderType: OrderType.Limit,
    timeInForce: TimeInForce.GoodTillCanceled,
    businessType: BusinessType.Spot,
    quantity: 0.01m,
    price: 1000m);
```

Live USDT-M futures order:

```csharp
var order = await rest.UsdtFuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETH_USDT",
    orderSide: OrderSide.Buy,
    orderType: OrderType.Market,
    quantity: 0.01m,
    positionSide: PositionSide.Long);
```

Read `references/safety.md` before generating trading code.

## Websocket Patterns

```csharp
var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "eth_usdt",
    update => Console.WriteLine(update.Data));

if (sub.Success)
    await socket.UnsubscribeAsync(sub.Data);
```

Public futures subscriptions use `socket.FuturesApi` and uppercase underscore symbols.

Private spot streams use a websocket token from `rest.SpotApi.Account.GetWebsocketTokenAsync()`. Private futures streams use a listen key from `rest.UsdtFuturesApi.Account.GetListenKeyAsync()`. Credentialed socket clients also expose overloads that acquire these automatically.

## SharedApis

Shared clients are available at:

- `rest.SpotApi.SharedClient`
- `rest.UsdtFuturesApi.SharedClient`
- `rest.CoinFuturesApi.SharedClient`
- `socket.SpotApi.SharedClient`
- `socket.FuturesApi.SharedClient`

Use `SharedClient.Discover()` for runtime capability metadata. Do not mix native XT models with SharedApis request or response models.

## Dependency Injection

```csharp
services.AddXT(options =>
{
    options.ApiCredentials = new XTCredentials("API_KEY", "API_SECRET");
});
```

Inject `IXTRestClient`, `IXTSocketClient`, `IXTOrderBookFactory`, `IXTTrackerFactory`, or `IXTUserClientProvider`.

## Safety Rules

- Read `references/safety.md` before credentials, orders, leverage, positions, withdrawals, transfers, or private streams.
- Prefer public market data and read-only account examples.
- Treat `CloseAllPositionsAsync()` as especially destructive: it closes all futures positions visible to the API account.
- Current source exposes live and custom environments, not a built-in testnet.

## References

- Read `references/api-surfaces.md` for product roots, capabilities, SharedApis, factories, and result types.
- Read `references/usage.md` for task-specific code patterns.
- Read `references/safety.md` for trading, futures, funds, stream, and retry guardrails.
- In a maintainer checkout, read `../XT.Net/Examples/ai-friendly/` for source-aligned examples.
