---
name: bitstamp-net
description: Build C#/.NET Bitstamp integrations with Bitstamp.Net, including ExchangeApi REST clients, websocket subscriptions, spot and derivative market data, account balances, fees, deposits, withdrawals, order placement/cancellation, derivatives positions, leverage settings, BitstampCredentials, Bitstamp slash symbols, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, ExchangeCallResult helper handling, and SharedApis access. Use when the user asks for Bitstamp market data, Bitstamp account or trading code, Bitstamp derivatives, Bitstamp websocket updates, Bitstamp error handling, or converting raw Bitstamp API usage to idiomatic Bitstamp.Net.
---

# Bitstamp.Net

## Overview

Use `Bitstamp.Net` for Bitstamp-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for Bitstamp endpoint groups, Bitstamp symbols, spot orders, derivative positions, leverage settings, withdrawals, or Bitstamp credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package Bitstamp.Net
```

Use these namespaces in examples:

```csharp
using Bitstamp.Net;
using Bitstamp.Net.Clients;
using Bitstamp.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new BitstampRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new BitstampSocketClient();
```

Current source-checked API surfaces:

- `rest.ExchangeApi.ExchangeData`: spot and derivative symbols, assets, tickers, order books, trades, klines, conversion rate, funding rates, margin tiers, collateral assets
- `rest.ExchangeApi.Account`: account balances, fees, user transactions, account symbols, max trade quantity, deposits, withdrawals, margin info, leverage settings
- `rest.ExchangeApi.Trading`: spot orders, open orders, order history, replacements, derivatives user trades, derivative positions, close positions, collateral updates
- `rest.ExchangeApi.SharedClient`: exchange-agnostic SharedApis REST interfaces for spot and derivatives workflows
- `socket.ExchangeApi`: public trade/order book/funding subscriptions plus private order and user trade subscriptions
- `socket.ExchangeApi.SharedClient`: exchange-agnostic spot socket interfaces for trades and order books

Do not use exchange roots from other libraries such as `SpotApi`, `UsdFuturesApi`, `FuturesApiV2`, `SpotApiV3`, `CoinFuturesApi`, or `PerpetualFuturesApi`.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new BitstampRestClient();
```

Private account, trading, derivatives position, withdrawal, leverage, and private websocket calls need `BitstampCredentials` with API key and API secret:

```csharp
var client = new BitstampRestClient(options =>
{
    options.ApiCredentials = new BitstampCredentials("API_KEY", "API_SECRET");
});
```

Use placeholders or environment variables in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared non-I/O symbol/cache helpers can return `ExchangeCallResult<T>`. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.ExchangeApi.ExchangeData.GetTickerAsync("ETH/USD");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Enums

- Spot symbols use slash format, for example `ETH/USD` and `BTC/USD`.
- Derivative symbols use Bitstamp derivative format, for example `ETH/USD-PERP`.
- Do not use Binance-style compact symbols such as `ETHUSDT`.
- Use `KlineInterval` for REST candles.
- Common trading enums include `OrderSide`, `OrderType`, and `MarginMode`.

Let the library generate client order IDs unless the user needs external correlation. Bitstamp examples often use explicit client IDs for traceability.

## REST Patterns

Public spot ticker:

```csharp
var ticker = await client.ExchangeApi.ExchangeData.GetTickerAsync("ETH/USD");
```

Authenticated balances:

```csharp
var balances = await client.ExchangeApi.Account.GetAccountBalancesAsync();
```

Spot limit order:

```csharp
var order = await client.ExchangeApi.Trading.PlaceLimitOrderAsync(
    symbol: "ETH/USD",
    side: OrderSide.Buy,
    price: 1m,
    orderType: OrderType.Limit,
    quantity: 0.01m);
```

Derivative positions and leverage:

```csharp
var positions = await client.ExchangeApi.Trading.GetOpenPositionsAsync("ETH/USD-PERP");
var leverage = await client.ExchangeApi.Account.GetLeverageSettingsAsync(
    MarginMode.Cross,
    "ETH/USD-PERP");
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new BitstampSocketClient();

var sub = await socket.ExchangeApi.SubscribeToTradeUpdatesAsync(
    "ETH/USD",
    update => Console.WriteLine($"{update.Data.Quantity} @ {update.Data.Price}"));

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

var shared = new BitstampRestClient().ExchangeApi.SharedClient;
var info = shared.Discover();
Console.WriteLine($"{info.Exchange} {info.TypeName}");
```

Bitstamp exposes shared clients on `ExchangeApi.SharedClient`. Shared REST supports spot and perpetual linear workflows; shared socket support is spot-only. Call `Discover()` before routing optional shared features. Do not mix Bitstamp-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddBitstamp(options =>
{
    options.Rest.ApiCredentials = new BitstampCredentials("API_KEY", "API_SECRET");
    options.Socket.ApiCredentials = new BitstampCredentials("API_KEY", "API_SECRET");
});
```

Inject `IBitstampRestClient` and `IBitstampSocketClient`, or the registered shared interfaces used by the target project. Match the repository's existing DI pattern if one is present.

## Safety Rules

- Read `references/safety.md` before generating trading, derivatives, leverage, withdrawal, collateral, close-position, or credential code.
- Prefer read-only examples for account inspection.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Derivative position, leverage, close-position, and collateral update calls can change account risk; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, derivatives, withdrawal, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../Bitstamp.Net/Examples/ai-friendly/` for complete compilable examples.
