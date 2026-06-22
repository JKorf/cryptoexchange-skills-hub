---
name: coinex-net
description: Build C#/.NET CoinEx integrations with CoinEx.Net, including SpotApiV2 and FuturesApi REST clients, spot and futures websocket subscriptions, spot/margin/futures market data, account balances, deposits, withdrawals, transfers, margin borrow/repay, order placement/cancellation, stop orders, futures positions, leverage, take-profit/stop-loss, CoinExCredentials, CoinEx compact symbols, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, CallResult batch-item handling, ExchangeCallResult shared helper handling, and SharedApis access. Use when the user asks for CoinEx spot market data, CoinEx account or trading code, CoinEx futures, CoinEx websocket updates, CoinEx error handling, or converting raw CoinEx API usage to idiomatic CoinEx.Net.
---

# CoinEx.Net

## Overview

Use `CoinEx.Net` for CoinEx-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for CoinEx V2 spot endpoints, CoinEx futures, margin account actions, stop orders, leverage, user streams, or CoinEx credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package CoinEx.Net
```

Use these namespaces in examples:

```csharp
using CoinEx.Net;
using CoinEx.Net.Clients;
using CoinEx.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new CoinExRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new CoinExSocketClient();
```

Current source-checked API surfaces:

- `rest.SpotApiV2.ExchangeData`: spot server time, symbols, assets, tickers, order books, trades, klines, and index prices
- `rest.SpotApiV2.Account`: spot, margin, financial, credit, and AMM balances; fees; account config; margin borrow/repay; deposits; withdrawals; transfers
- `rest.SpotApiV2.Trading`: spot/margin orders, stop orders, batch orders, edits, cancellations, open/closed orders, user trades, and order trades
- `rest.SpotApiV2.SharedClient`: exchange-agnostic SharedApis REST interfaces for spot workflows
- `rest.FuturesApi.ExchangeData`: futures symbols, tickers, order books, trades, klines, index/mark prices, funding rates, open interest, and related market data
- `rest.FuturesApi.Account`: futures balances, trading fees, and leverage
- `rest.FuturesApi.Trading`: futures orders, stop orders, cancellations, positions, position history, close-position, TP/SL, and margin adjustment
- `rest.FuturesApi.SharedClient`: exchange-agnostic SharedApis REST interfaces for perpetual futures workflows
- `socket.SpotApiV2`: spot public streams plus private spot order, stop-order, trade, and balance streams
- `socket.FuturesApi`: futures public streams plus private futures order, stop-order, trade, balance, and position streams

Do not use exchange roots from other libraries such as `SpotApi`, `UsdFuturesApi`, `CoinFuturesApi`, `PerpetualFuturesApi`, `FuturesApiV2`, `SpotApiV3`, `V5Api`, `ExchangeApi`, or `UnifiedApi`.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new CoinExRestClient();
```

Private account, trading, transfers, deposits/withdrawals, margin, futures account settings, and private websocket calls need `CoinExCredentials`:

```csharp
var client = new CoinExRestClient(options =>
{
    options.ApiCredentials = new CoinExCredentials("API_KEY", "API_SECRET");
});
```

Use placeholders or environment variables in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared non-I/O symbol/cache helpers can return `ExchangeCallResult<T>`. Batch REST operations can return `HttpResult<CallResult<T>[]>` or `HttpResult<CoinExBatchResult<T>[]>`; check both the outer result and each item result. Always check `Success` before reading `Data`.

```csharp
var tickers = await client.SpotApiV2.ExchangeData.GetTickersAsync(new[] { "BTCUSDT" });
if (!tickers.Success)
{
    Console.WriteLine(tickers.Error);
    return;
}

Console.WriteLine(tickers.Data.First().LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Enums

- CoinEx symbols use compact format, for example `BTCUSDT` and `ETHUSDT`.
- Do not use dash-separated symbols such as `BTC-USDT`.
- Use `KlineInterval` for candles.
- Common trading enums include `AccountType`, `OrderSide`, `OrderTypeV2`, and `MarginMode`.
- Query symbol metadata and round using precision fields before generating production order quantities or prices.

Let the library generate client order IDs unless the user needs external correlation.

## REST Patterns

Public spot ticker:

```csharp
var tickers = await client.SpotApiV2.ExchangeData.GetTickersAsync(new[] { "BTCUSDT" });
```

Authenticated spot balances:

```csharp
var balances = await client.SpotApiV2.Account.GetBalancesAsync();
```

Spot limit order:

```csharp
var order = await client.SpotApiV2.Trading.PlaceOrderAsync(
    symbol: "BTCUSDT",
    accountType: AccountType.Spot,
    side: OrderSide.Buy,
    type: OrderTypeV2.Limit,
    quantity: 0.001m,
    price: 50000m);
```

Futures leverage, positions, and close:

```csharp
await client.FuturesApi.Account.SetLeverageAsync("ETHUSDT", MarginMode.Cross, 5);
var positions = await client.FuturesApi.Trading.GetPositionsAsync("ETHUSDT");
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new CoinExSocketClient();

var sub = await socket.SpotApiV2.SubscribeToTickerUpdatesAsync(
    new[] { "BTCUSDT" },
    update => Console.WriteLine(update.Data.First().LastPrice));

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

var shared = new CoinExRestClient().SpotApiV2.SharedClient;
var info = shared.Discover();
Console.WriteLine($"{info.Exchange} {info.TypeName}");
```

CoinEx exposes shared clients on `SpotApiV2.SharedClient` and `FuturesApi.SharedClient`. Spot shared clients support `TradingMode.Spot`; futures shared clients support `TradingMode.PerpetualLinear` and `TradingMode.PerpetualInverse`. Call `Discover()` before routing optional shared features. Do not mix CoinEx-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddCoinEx(options =>
{
    options.ApiCredentials = new CoinExCredentials("API_KEY", "API_SECRET");
});
```

Inject `ICoinExRestClient` and `ICoinExSocketClient`, or the registered shared interfaces used by the target project. Match the repository's existing DI pattern if one is present.

## Safety Rules

- Read `references/safety.md` before generating trading, futures, leverage, close-position, margin borrow/repay, transfer, withdrawal, or credential code.
- Prefer read-only examples for account inspection.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Futures examples can change leverage and create real exposure; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, futures, margin, transfer, withdrawal, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../CoinEx.Net/Examples/ai-friendly/` for complete compilable examples.
