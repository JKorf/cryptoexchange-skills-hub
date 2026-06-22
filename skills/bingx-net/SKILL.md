---
name: bingx-net
description: Build C#/.NET BingX integrations with BingX.Net, including Spot, PerpetualFutures, REST clients, websocket subscriptions, account reads, order placement/cancellation, BingXCredentials, BingX hyphenated symbols, enums, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, and SharedApis access. Use when the user asks for BingX market data, BingX account or trading code, BingX perpetual futures, BingX websocket updates, BingX error handling, or converting raw BingX API usage to idiomatic BingX.Net.
---

# BingX.Net

## Overview

Use `BingX.Net` for BingX-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for BingX-only endpoint groups, BingX spot trading, BingX perpetual futures, subaccounts, user data, order placement, or BingX-specific enums and models.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package JK.BingX.Net
```

Use these namespaces in examples:

```csharp
using BingX.Net;
using BingX.Net.Clients;
using BingX.Net.Enums;
using CryptoExchange.Net.Objects;
```

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new BingXRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new BingXSocketClient();
```

Primary API surfaces:

- `rest.SpotApi`: spot market data, spot account, spot trading
- `rest.PerpetualFuturesApi`: perpetual futures market data, futures account, futures trading
- `rest.SubAccountApi`: subaccount endpoints
- `socket.SpotApi`: spot websocket subscriptions
- `socket.PerpetualFuturesApi`: perpetual futures websocket subscriptions

There is no `FuturesApi` root. Use `PerpetualFuturesApi`.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new BingXRestClient();
```

Private account and trading calls need `BingXCredentials`:

```csharp
var client = new BingXRestClient(options =>
{
    options.ApiCredentials = new BingXCredentials("API_KEY", "API_SECRET");
});
```

Use placeholders or environment variables in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Always check `Success` before reading `Data`.

```csharp
var tickers = await client.SpotApi.ExchangeData.GetTickersAsync("BTC-USDT");
if (!tickers.Success)
{
    Console.WriteLine(tickers.Error);
    return;
}

Console.WriteLine(tickers.Data.Single().LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Enums

- Spot examples should use hyphenated symbols, for example `BTC-USDT`.
- Perpetual futures examples should use hyphenated symbols, for example `ETH-USDT`.
- Spot order enums include `OrderSide`, `OrderType`, and `TimeInForce`.
- Perpetual futures order examples use `OrderSide`, `FuturesOrderType`, `PositionSide`, and optional `MarginMode`.
- In one-way futures mode, use `PositionSide.Both` where the endpoint requires a position side.
- Use exchange info and trading rules before generating production order quantities or prices.

Let the library generate client order IDs unless the user needs external correlation.

## REST Patterns

Public spot ticker:

```csharp
var tickers = await client.SpotApi.ExchangeData.GetTickersAsync("BTC-USDT");
var ticker = tickers.Success ? tickers.Data.Single() : null;
```

Authenticated spot balances:

```csharp
var balances = await client.SpotApi.Account.GetBalancesAsync();
```

Spot limit order:

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTC-USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    price: 50000m,
    timeInForce: TimeInForce.GoodTillCanceled);
```

Perpetual futures leverage and order:

```csharp
await client.PerpetualFuturesApi.Account.SetLeverageAsync("ETH-USDT", PositionSide.Long, 5);

var order = await client.PerpetualFuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETH-USDT",
    side: OrderSide.Buy,
    type: FuturesOrderType.Market,
    positionSide: PositionSide.Long,
    quantity: 0.01m);
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new BingXSocketClient();

var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "BTC-USDT",
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

## SharedApis

Use SharedApis only when portability matters:

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new BingXRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")));
```

Do not mix BingX-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddBingX(options =>
{
    options.ApiCredentials = new BingXCredentials("API_KEY", "API_SECRET");
});
```

Inject `IBingXRestClient` and `IBingXSocketClient`, or the registered BingX REST/socket client interfaces used by the target project. Match the repository's existing DI pattern if one is present.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, perpetual futures, withdrawal, transfer, subaccount, or credential code.
- Prefer read-only examples for account inspection.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Perpetual futures examples can change leverage and create real exposure; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, perpetual futures, subaccount, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../BingX.Net/Examples/ai-friendly/` for complete compilable examples.
