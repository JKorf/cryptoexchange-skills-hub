---
name: aster-net
description: Build C#/.NET Aster DEX integrations with Aster.Net, including Spot V3, Futures V3, REST clients, websocket subscriptions, account reads, order placement/cancellation, AsterCredentials with V3 user/signer private keys, Aster-specific symbols, enums, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, and Shared API V2 strict capabilities and aggregates. Use when the user asks for Aster market data, Aster account or trading code, Aster futures, Aster websocket updates, Aster error handling, or converting raw Aster API usage to idiomatic Aster.Net.
---

# Aster.Net

## Overview

Use `Aster.Net` for Aster DEX-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for Aster-only endpoint groups, Aster V3 authentication, Aster futures, user data, order placement, or Aster-specific enums and models.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

This skill targets `JKorf.Aster.Net` 4.4.0, the latest published stable release verified on 2026-08-24.

Install the exchange package:

```bash
dotnet add package JKorf.Aster.Net --version 4.4.0
```

Use these namespaces in examples:

```csharp
using Aster.Net;
using Aster.Net.Clients;
using Aster.Net.Enums;
using CryptoExchange.Net.Objects;
```

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new AsterRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new AsterSocketClient();
```

Use V3 by default:

- `rest.SpotV3Api`: spot V3 market data, spot account, spot trading
- `rest.FuturesV3Api`: futures V3 market data, futures account, futures trading
- `socket.SpotV3Api`: spot V3 websocket subscriptions
- `socket.FuturesV3Api`: futures V3 websocket subscriptions

Compatibility branches:

- `SpotApi` and `FuturesApi` are V1 compatibility surfaces.
- Do not use V1 branches in generated examples unless the user explicitly asks for V1 or provides only V1-style credentials.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new AsterRestClient();
```

Private V3 account and trading calls need user and signer private keys:

```csharp
var client = new AsterRestClient(options =>
{
    options.ApiCredentials = new AsterCredentials()
        .WithV3("USER_PRIVATE_KEY", "SIGNER_PRIVATE_KEY");
});
```

Do not use `new AsterCredentials("key", "secret")` for V3 examples; that configures V1 HMAC credentials. Use placeholders or environment variables in generated code. Do not hardcode real private keys.

## Result Handling

REST methods return `HttpResult<T>`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.SpotV3Api.ExchangeData.GetTickerAsync("BTCUSDT");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Enums

- Spot symbols usually use compact format, for example `BTCUSDT`.
- Futures symbols commonly use compact format, for example `ETHUSDT`.
- Spot and futures order examples use `OrderSide`, `OrderType`, and `TimeInForce`.
- Futures position-side examples can use `PositionSide.Long` or `PositionSide.Short` in hedge mode.
- Futures margin examples can use `MarginType.Isolated`.
- Use exchange info and filters before generating production order quantities or prices.

Let the library generate client order IDs unless the user needs external correlation.

## REST Patterns

Public spot ticker:

```csharp
var ticker = await client.SpotV3Api.ExchangeData.GetTickerAsync("BTCUSDT");
```

Authenticated spot account:

```csharp
var account = await client.SpotV3Api.Account.GetAccountInfoAsync();
```

Spot limit order:

```csharp
var order = await client.SpotV3Api.Trading.PlaceOrderAsync(
    symbol: "BTCUSDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    price: 50000m,
    timeInForce: TimeInForce.GoodTillCanceled);
```

Futures leverage and order:

```csharp
await client.FuturesV3Api.Account.SetLeverageAsync("ETHUSDT", 5);

var order = await client.FuturesV3Api.Trading.PlaceOrderAsync(
    symbol: "ETHUSDT",
    side: OrderSide.Buy,
    type: OrderType.Market,
    quantity: 0.01m);
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new AsterSocketClient();

var sub = await socket.SpotV3Api.SubscribeToTickerUpdatesAsync(
    "BTCUSDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

// On shutdown:
await socket.UnsubscribeAsync(sub.Data);
```

Use `UnsubscribeAsync` or `UnsubscribeAllAsync` on shutdown. Do not leave example subscriptions running.

## Shared API V2

Use Shared APIs only when portability matters. Depend on the narrow capability needed by the workflow:

```csharp
using CryptoExchange.Net.SharedApis;

using var client = new AsterRestClient();
IGetTickerRest ticker = client.SpotV3Api.SharedApi;

var result = await ticker.GetTickerAsync(
    new GetTickerRequest(
        new SharedSymbol(TradingMode.Spot, "BTC", "USDT")));
```

Use `IAsterSharedApiClient` as the exchange aggregate. Its aggregate properties—`SpotRest`, `FuturesRest`, `SpotV3Rest`, `FuturesV3Rest`, `SpotSocket`, `FuturesSocket`, `SpotV3Socket`, `FuturesV3Socket`—expose the supported Shared API surfaces at compile time. Use `GetCapability` only when the capability, trading mode, or transport is selected dynamically.

For WebSocket workflows, use the narrow subscription interface exposed by the applicable socket surface, such as `ISubscribeTickerSocket` or `ISubscribeTradesSocket`. Do not mix Aster-native request/model types with Shared API request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddAster(options =>
{
    options.ApiCredentials = new AsterCredentials()
        .WithV3("USER_PRIVATE_KEY", "SIGNER_PRIVATE_KEY");
});
```

Inject `IAsterRestClient` and `IAsterSocketClient`, or the registered Aster REST/socket client interfaces used by the target project. Match the repository's existing DI pattern if one is present.

`AddAster` selects V3 by default. It selects V1 only when REST credentials contain V1 credentials and no V3 credentials.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, futures, withdrawal, transfer, or credential code.
- Treat user and signer private keys as high-value secrets.
- Prefer read-only examples for account inspection.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Futures examples can change leverage and create real exposure; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, futures, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../Aster.Net/Examples/ai-friendly/` for complete compilable examples.
