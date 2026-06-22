---
name: bybit-net
description: Build C#/.NET Bybit integrations with Bybit.Net, including V5Api REST clients, V5 spot/linear/inverse/options/spread/private websocket clients, spot and derivatives market data, unified account balances, order placement/cancellation, positions, leverage, margin and position mode, BybitCredentials, Bybit V5 Category arguments, Bybit compact symbols, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, CallResult batch-item handling, ExchangeCallResult shared helper handling, and SharedApis access. Use when the user asks for Bybit V5 market data, Bybit account or trading code, Bybit websocket updates, Bybit error handling, or converting raw Bybit API usage to idiomatic Bybit.Net.
---

# Bybit.Net

## Overview

Use `Bybit.Net` for Bybit-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for Bybit V5 endpoint groups, Bybit categories, Bybit account modes, spot orders, derivatives positions, leverage, user streams, spread data, earn, crypto loan, or Bybit credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package Bybit.Net
```

Use these namespaces in examples:

```csharp
using Bybit.Net;
using Bybit.Net.Clients;
using Bybit.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new BybitRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new BybitSocketClient();
```

Current source-checked API surfaces:

- `rest.V5Api.ExchangeData`: V5 market data for spot, linear, inverse, options, spreads, klines, order books, tickers, funding, open interest, risk limits, system status, and announcements
- `rest.V5Api.Account`: balances, deposits, withdrawals, transfers, fees, margin/leverage/risk settings, spot margin, convert, broker, demo funds, and asset metadata
- `rest.V5Api.Trading`: orders, batch orders, positions, user trades, risk confirmation, trading stops, leverage-token orders, spread orders, and order pre-checks
- `rest.V5Api.SubAccount`: subaccount and subaccount API key management
- `rest.V5Api.CryptoLoan`: collateral, borrow, repay, open loan, completed loan, and collateral adjustment endpoints
- `rest.V5Api.Earn`: earn product, order, and staked-position endpoints
- `rest.V5Api.SharedClient`: exchange-agnostic SharedApis REST interfaces for spot and derivatives workflows
- `socket.V5SpotApi`: spot public websocket streams
- `socket.V5LinearApi`: linear public websocket streams
- `socket.V5InverseApi`: inverse public websocket streams
- `socket.V5OptionsApi`: options public websocket streams
- `socket.V5SpreadApi`: spread public websocket streams
- `socket.V5PrivateApi`: private order, trade, position, wallet, greeks, spread, and disconnect-cancel-all streams

Do not use exchange roots from other libraries such as `SpotApi`, `UsdFuturesApi`, `CoinFuturesApi`, `PerpetualFuturesApi`, `FuturesApiV2`, `SpotApiV3`, or `ExchangeApi`.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new BybitRestClient();
```

Private account, trading, transfer, withdrawal, earn, crypto-loan, and private websocket calls need `BybitCredentials`:

```csharp
var client = new BybitRestClient(options =>
{
    options.ApiCredentials = new BybitCredentials("API_KEY", "API_SECRET");
});
```

Use environment variables in runnable examples. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared non-I/O symbol/cache helpers can return `ExchangeCallResult<T>`. Some batch REST operations return `HttpResult<CallResult<T>[]>`; check both the outer result and each item result. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.V5Api.ExchangeData.GetSpotTickersAsync("ETHUSDT");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.List.FirstOrDefault()?.LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols, Categories, And Enums

- Bybit V5 symbols usually use compact format, for example `ETHUSDT` and `BTCUSDT`.
- Many V5 REST methods require `Category`, for example `Category.Spot`, `Category.Linear`, `Category.Inverse`, and `Category.Option`.
- Use `KlineInterval` for candles.
- Common trading enums include `OrderSide`, `NewOrderType`, `TimeInForce`, `PositionIdx`, `AccountType`, `MarginMode`, and `PositionMode`.
- Use exchange data, order price limits, and account mode before generating production order quantities, prices, leverage, or position indexes.

Let the library generate client order IDs unless the user needs external correlation.

## REST Patterns

Public spot ticker:

```csharp
var ticker = await client.V5Api.ExchangeData.GetSpotTickersAsync("ETHUSDT");
```

Linear klines and order book:

```csharp
var klines = await client.V5Api.ExchangeData.GetKlinesAsync(
    Category.Linear,
    "ETHUSDT",
    KlineInterval.OneMinute,
    limit: 5);

var book = await client.V5Api.ExchangeData.GetOrderbookAsync(
    Category.Linear,
    "ETHUSDT",
    limit: 25);
```

Unified account balances:

```csharp
var balances = await client.V5Api.Account.GetBalancesAsync(AccountType.Unified, "USDT");
```

Dry-run-first trading pattern:

```csharp
var openOrders = await client.V5Api.Trading.GetOrdersAsync(
    Category.Linear,
    symbol: "ETHUSDT",
    limit: 20);
```

Only place live orders when the user explicitly asks:

```csharp
var order = await client.V5Api.Trading.PlaceOrderAsync(
    Category.Linear,
    "ETHUSDT",
    OrderSide.Buy,
    NewOrderType.Limit,
    quantity: 0.01m,
    price: 1000m,
    timeInForce: TimeInForce.GoodTillCanceled,
    positionIdx: PositionIdx.OneWayMode);
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new BybitSocketClient();

var sub = await socket.V5LinearApi.SubscribeToTickerUpdatesAsync(
    "ETHUSDT",
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

var shared = new BybitRestClient().V5Api.SharedClient;
var info = shared.Discover();
Console.WriteLine($"{info.Exchange} {info.TypeName}");
```

Bybit exposes shared REST on `V5Api.SharedClient`, public shared sockets on `V5SpotApi.SharedClient`, `V5LinearApi.SharedClient`, and `V5InverseApi.SharedClient`, and private shared sockets on `V5PrivateApi.SharedClient`. Call `Discover()` before routing optional shared features. Do not mix Bybit-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddBybit(options =>
{
    options.ApiCredentials = new BybitCredentials("API_KEY", "API_SECRET");
});
```

Inject `IBybitRestClient` and `IBybitSocketClient`, or the registered shared interfaces used by the target project. Match the repository's existing DI pattern if one is present.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, margin mode, position mode, risk limit, transfer, withdrawal, earn, crypto-loan, or credential code.
- Prefer read-only examples for account inspection.
- Keep trading examples dry-run by default unless the user explicitly asks for order placement.
- Futures and options examples can change leverage, margin mode, position mode, risk limits, and real exposure; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, derivatives, transfer, withdrawal, earn, loan, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../Bybit.Net/Examples/ai-friendly/` for complete compilable examples.
