---
name: blofin-net
description: Build C#/.NET BloFin integrations with BloFin.Net, including AccountApi, FuturesApi REST clients, futures websocket subscriptions, futures market data, account balances, futures balances, order placement/cancellation, positions, leverage, margin mode, position mode, BloFinCredentials with passphrase, BloFin futures symbols, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, CallResult batch-item handling, and SharedApis access. Use when the user asks for BloFin futures market data, BloFin account or trading code, BloFin websocket updates, BloFin error handling, or converting raw BloFin API usage to idiomatic BloFin.Net.
---

# BloFin.Net

## Overview

Use `BloFin.Net` for BloFin-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for BloFin futures endpoint groups, BloFin futures symbols, futures account settings, order placement, positions, leverage, deposits/withdrawals, or BloFin credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package BloFin.Net
```

Use these namespaces in examples:

```csharp
using BloFin.Net;
using BloFin.Net.Clients;
using BloFin.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new BloFinRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new BloFinSocketClient();
```

Current source-checked API surfaces:

- `rest.AccountApi`: general account balances, transfers, account config, API key info, deposits, withdrawals, and account SharedApis
- `rest.FuturesApi.ExchangeData`: futures symbols, tickers, order books, trades, index/mark prices, funding rates, klines, and position tiers
- `rest.FuturesApi.Account`: futures balances, margin mode, position mode, leverage reads and updates
- `rest.FuturesApi.Trading`: futures positions, orders, batch orders, TP/SL orders, trigger orders, cancellations, close position, user trades, price limits, and position history
- `rest.FuturesApi.SharedClient`: exchange-agnostic SharedApis REST interfaces for futures workflows
- `socket.FuturesApi`: futures public subscriptions plus private position, order, trigger-order, and balance subscriptions
- `socket.FuturesApi.SharedClient`: exchange-agnostic SharedApis socket interfaces for futures workflows

Do not use exchange roots from other libraries such as `ExchangeApi`, `SpotApi`, `UsdFuturesApi`, `FuturesApiV2`, `SpotApiV3`, `CoinFuturesApi`, or `PerpetualFuturesApi`.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public futures market data does not need credentials:

```csharp
var client = new BloFinRestClient();
```

Private account, trading, deposits/withdrawals, futures account settings, and private websocket calls need `BloFinCredentials` with API key, API secret, and passphrase:

```csharp
var client = new BloFinRestClient(options =>
{
    options.ApiCredentials = new BloFinCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
});
```

Use placeholders or environment variables in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Some batch REST operations return `HttpResult<CallResult<T>[]>`; check both the outer result and each item result. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.FuturesApi.ExchangeData.GetTickersAsync("ETH-USDT");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.FirstOrDefault()?.LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Enums

- BloFin futures symbols use dash format, for example `ETH-USDT`.
- Do not use Binance-style compact symbols such as `ETHUSDT`.
- Use `KlineInterval` for candles.
- Common account and trading enums include `AccountType`, `ProductType`, `OrderSide`, `OrderType`, `MarginMode`, `PositionMode`, and `PositionSide`.
- Use exchange data and price limits before generating production order quantities or prices.

Let the library generate client order IDs unless the user needs external correlation. BloFin examples often use explicit client IDs for traceability.

## REST Patterns

Public futures ticker:

```csharp
var ticker = await client.FuturesApi.ExchangeData.GetTickersAsync("ETH-USDT");
```

Authenticated general account balance:

```csharp
var balances = await client.AccountApi.GetAccountBalancesAsync(AccountType.Futures, "USDT");
```

Authenticated futures balances:

```csharp
var futuresBalances = await client.FuturesApi.Account.GetBalancesAsync(ProductType.UsdtFutures);
```

Futures limit order:

```csharp
var order = await client.FuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETH-USDT",
    side: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 1m,
    marginMode: MarginMode.Cross,
    price: 1m,
    positionSide: PositionSide.Long);
```

Positions and leverage:

```csharp
var positions = await client.FuturesApi.Trading.GetPositionsAsync("ETH-USDT");
var leverage = await client.FuturesApi.Account.GetLeverageAsync("ETH-USDT", MarginMode.Cross);
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new BloFinSocketClient();

var sub = await socket.FuturesApi.SubscribeToTickerUpdatesAsync(
    "ETH-USDT",
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

var shared = new BloFinRestClient().FuturesApi.SharedClient;
var info = shared.Discover();
Console.WriteLine($"{info.Exchange} {info.TypeName}");
```

BloFin exposes account shared clients on `AccountApi.SharedClient` and futures shared clients on `FuturesApi.SharedClient`. Call `Discover()` before routing optional shared features. Do not mix BloFin-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddBloFin(options =>
{
    options.ApiCredentials = new BloFinCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
});
```

Inject `IBloFinRestClient` and `IBloFinSocketClient`, or the registered shared interfaces used by the target project. Match the repository's existing DI pattern if one is present.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, margin mode, position mode, close-position, transfer, withdrawal, or credential code.
- Prefer read-only examples for account inspection.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Futures examples can change leverage, margin mode, position mode, or create real exposure; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, futures, transfer, withdrawal, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../BloFin.Net/Examples/ai-friendly/` for complete compilable examples.
