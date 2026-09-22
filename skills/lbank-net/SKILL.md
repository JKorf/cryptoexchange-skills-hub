---
name: lbank-net
description: Build C#/.NET LBank integrations with LBank.Net, including package setup, SpotApi REST clients, public and private websocket subscriptions, market data, balances, deposits, withdrawals, spot order management, HMAC or RSA credentials, lowercase underscore-separated symbols, dependency injection, local order books, trackers, user client providers, HttpResult REST handling, WebSocketResult subscription handling, and CryptoExchange.Net Shared API V2 strict capabilities and aggregates. Use when the user asks for LBank spot market data, wallet or account code, trading, user streams, websocket updates, error handling, or converting raw LBank API usage to idiomatic LBank.Net. Current LBank.Net does not expose futures, derivatives, or margin clients.
---

# LBank.Net

## Overview

Use `LBank.Net` for LBank-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill for native LBank Spot endpoints, credentials, wallet operations, private streams, local order books, or trackers.

Use `cryptoexchange-net` when the same code must run across exchanges through `CryptoExchange.Net.SharedApis`.

LBank.Net currently exposes Spot only. Do not invent futures, derivatives, margin, or unified trading roots.

## Setup

This skill targets `LBank.Net` 1.1.0, the latest published stable release verified on 2026-08-24.

Install:

```bash
dotnet add package LBank.Net --version 1.1.0
```

Use:

```csharp
using CryptoExchange.Net.Objects;
using LBank.Net;
using LBank.Net.Clients;
using LBank.Net.Enums;
```

Add `using CryptoExchange.Net.SharedApis;` only for SharedApis code.

## Client Roots

```csharp
var rest = new LBankRestClient();
var socket = new LBankSocketClient();
```

REST surfaces:

- `rest.SpotApi.ExchangeData`: time, symbols, assets, books, prices, tickers, trades, and klines
- `rest.SpotApi.Account`: balances, account information, deposits, withdrawals, fees, and user-stream keys
- `rest.SpotApi.Trading`: place, query, paginate, and cancel Spot orders
- `rest.SpotApi.SharedApi`: shared Spot REST interfaces

Socket surfaces:

- `socket.SpotApi`: public trades, klines, books, tickers, and private order/balance updates
- `socket.SpotApi.SharedApi`: shared Spot socket interfaces

Read `references/api-surfaces.md` when selecting a method or shared interface.

## Credentials

Public market data needs no credentials. Private REST and socket operations use `LBankCredentials`.

HMAC:

```csharp
var client = new LBankRestClient(options =>
{
    options.ApiCredentials =
        new LBankCredentials("API_KEY", "API_SECRET");
});
```

RSA XML is supported with `WithRSAXml(...)`; RSA PEM is supported with `WithRSAPem(...)` on compatible targets. Never substitute the base `ApiCredentials` type or invent a passphrase.

Use placeholders, environment variables, or injected options. Never hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Check `Success` before reading `Data`.

```csharp
var tickers = await client.SpotApi.ExchangeData
    .GetTickersAsync("eth_usdt");

if (!tickers.Success)
{
    Console.WriteLine(tickers.Error);
    return;
}

Console.WriteLine(tickers.Data.Single().Ticker.LastPrice);
```

Use `result.Error.Code`, `Message`, `ErrorType`, and `IsTransient` for diagnostics. Retry only transient failures with bounded backoff.

## Symbols, Klines, and Orders

- Native symbols normally use lowercase `base_quote`, for example `eth_usdt`.
- REST klines use `KlineInterval`; socket klines use `StreamKlineInterval`.
- `GetKlinesAsync` requires both `limit` and `afterTime`.
- LBank combines side and execution behavior in `OrderType`; do not pass a separate side or time-in-force.
- `OrderType.BuyMarket` interprets `quantity` in the quote asset.
- Limit-style orders require `price`; market orders do not.
- Order history and open-order queries require `page` and `pageSize`.

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "eth_usdt",
    orderType: OrderType.BuyLimit,
    quantity: 0.01m,
    price: 2000m,
    clientOrderId: $"example-{Guid.NewGuid():N}");

if (!order.Success)
{
    Console.WriteLine(order.Error);
    return;
}
```

The upstream LBank order API may not be enabled for every account. Preserve result handling and do not interpret a structurally correct request as guaranteed access.

## Websocket Pattern

```csharp
var socket = new LBankSocketClient();

var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "eth_usdt",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Public streams cover trades, klines, order books, and tickers. Order-book depth must be `10`, `50`, or `100`.

Private order and balance subscriptions accept a listen key. Pass `null` on an authenticated socket client to let the library acquire and maintain one. Public streams use v3 by default through `LBankSocketOptions.UseV3 = true`; select v2 only when specifically required.

Keep handlers fast and unsubscribe on shutdown.

## Shared API V2

Use Shared APIs only when portability matters. Depend on the narrow capability needed by the workflow:

```csharp
using CryptoExchange.Net.SharedApis;

using var client = new LBankRestClient();
IGetTickerRest ticker = client.SpotApi.SharedApi;

var result = await ticker.GetTickerAsync(
    new GetTickerRequest(
        new SharedSymbol(TradingMode.Spot, "ETH", "USDT")));
```

Use `ILBankSharedApiClient` as the exchange aggregate. Its aggregate properties—`SpotRest`, `SpotSocket`—expose the supported Shared API surfaces at compile time. Use `GetCapability` only when the capability, trading mode, or transport is selected dynamically.

For WebSocket workflows, use the narrow subscription interface exposed by the applicable socket surface, such as `ISubscribeTickerSocket` or `ISubscribeTradesSocket`. Do not mix LBank-native request/model types with Shared API request/model types.

## Dependency Injection

```csharp
services.AddLBank(options =>
{
    options.ApiCredentials =
        new LBankCredentials("API_KEY", "API_SECRET");
});
```

Inject `ILBankRestClient`, `ILBankSocketClient`, `ILBankOrderBookFactory`, `ILBankTrackerFactory`, or `ILBankUserClientProvider`.

Only `LBankEnvironment.Live` is built in. Use `LBankEnvironment.CreateCustom(...)` for intentional custom endpoints; do not invent a testnet.

## Safety Rules

- Read `references/safety.md` before generating account, order, cancellation, withdrawal, private websocket, tracker, user-client-provider, credential, or retry code.
- Prefer read-only examples for balances and order inspection.
- State clearly when examples call live order, cancellation, or withdrawal endpoints.
- Validate current symbol rules, precision, quantity, price, network, fee, address, memo/tag, and balance inputs before mutations.
- Never expose real API keys, secrets, or RSA private keys.

## References

- Read `references/api-surfaces.md` for client roots, native methods, socket streams, shared interfaces, result types, factories, and environments.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for wallet, trading, websocket, retry, credential, and lifecycle guardrails.
- In a maintainer checkout with sibling repositories, read `../LBank.Net/Examples/ai-friendly/`, `../LBank.Net/docs/ai-api-map.md`, and the source interfaces for complete, current examples.
