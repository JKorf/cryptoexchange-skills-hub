---
name: cryptocom-net
description: Build C#/.NET Crypto.com integrations with CryptoCom.Net, including ExchangeApi REST clients, ExchangeApi websocket subscriptions, market data, account balances, deposits, withdrawals, staking, spot trading, margin, derivatives/perpetuals, positions, order placement/cancellation, OCO orders, CryptoComCredentials, Crypto.com underscore spot symbols, derivative instrument symbols, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, QueryResult socket request handling, CallResult batch-item handling, ExchangeCallResult shared helper handling, and SharedApis access. Use when the user asks for Crypto.com exchange market data, Crypto.com account or trading code, Crypto.com derivatives, Crypto.com staking, Crypto.com websocket updates, Crypto.com error handling, or converting raw Crypto.com API usage to idiomatic CryptoCom.Net.
---

# CryptoCom.Net

## Overview

Use `CryptoCom.Net` for Crypto.com Exchange-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for Crypto.com exchange endpoints, trading, derivatives/perpetuals, staking, websocket streams, socket request methods, or Crypto.com credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package CryptoCom.Net
```

Use these namespaces in examples:

```csharp
using CryptoCom.Net;
using CryptoCom.Net.Clients;
using CryptoCom.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for exchange-agnostic SharedApis code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new CryptoComRestClient();
```

Create socket clients for websocket subscriptions and socket API requests:

```csharp
var socket = new CryptoComSocketClient();
```

Primary API surfaces:

- `rest.ExchangeApi.ExchangeData`: public market data, symbols, valuations, announcements
- `rest.ExchangeApi.Account`: balances, account settings, fees, deposits, withdrawals, isolated margin transfers
- `rest.ExchangeApi.Trading`: orders, OCO orders, positions, user trades
- `rest.ExchangeApi.Staking`: stake, unstake, staking positions, conversion
- `socket.ExchangeApi`: public streams, user streams, and private socket request methods

CryptoCom.Net does not expose Binance-style roots such as `SpotApi`, `UsdFuturesApi`, or `CoinFuturesApi`. Use `ExchangeApi` for both spot and derivatives.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new CryptoComRestClient();
```

Private account, trading, staking, withdrawal, and user stream calls need `CryptoComCredentials`:

```csharp
var client = new CryptoComRestClient(options =>
{
    options.ApiCredentials = new CryptoComCredentials("API_KEY", "API_SECRET");
});
```

Use placeholders or environment variables in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Socket API request methods return `QueryResult<T>` or `QueryResult`. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.ExchangeApi.ExchangeData.GetTickersAsync("BTC_USDT");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.FirstOrDefault()?.LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Enums

- Spot symbols use underscore instrument names, for example `BTC_USDT` and `ETH_USDT`.
- Perpetual examples commonly use symbols such as `ETHUSD_PERP`, while `CryptoComExchange.FormatSymbol("ETH", "USD", TradingMode.PerpetualLinear)` formats as `ETHUSD-PERP`; prefer exact names from `GetSymbolsAsync()` when unsure.
- Delivery futures require a delivery date when formatting from base/quote assets.
- Order enums include `OrderSide`, `OrderType`, `TimeInForce`, `PriceType`, `OrderTypeFilter`, `SelfTradePreventionScope`, and `SelfTradePreventionMode`.
- Klines use `KlineInterval`.
- Validate instrument names, tick sizes, price decimals, quantity tick sizes, leverage, isolated margin parameters, and account permissions before production orders.

Let the library generate client order IDs unless the user needs external correlation.

## REST Patterns

Public ticker:

```csharp
var ticker = await client.ExchangeApi.ExchangeData.GetTickersAsync("BTC_USDT");
```

Authenticated balances:

```csharp
var balances = await client.ExchangeApi.Account.GetBalancesAsync();
```

Spot limit order:

```csharp
var order = await client.ExchangeApi.Trading.PlaceOrderAsync(
    symbol: "BTC_USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    price: 50000m);
```

Derivative position and close:

```csharp
var positions = await client.ExchangeApi.Trading.GetPositionsAsync("ETHUSD_PERP");

var close = await client.ExchangeApi.Trading.ClosePositionAsync(
    symbol: "ETHUSD_PERP",
    orderType: OrderType.Market);
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new CryptoComSocketClient();

var sub = await socket.ExchangeApi.SubscribeToTickerUpdatesAsync(
    "BTC_USDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Use `UnsubscribeAsync` or `UnsubscribeAllAsync` on shutdown. Do not leave example subscriptions running.

## SharedApis

Use SharedApis only when portability matters:

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new CryptoComRestClient().ExchangeApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")));
```

Do not mix Crypto.com-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddCryptoCom(options =>
{
    options.ApiCredentials = new CryptoComCredentials("API_KEY", "API_SECRET");
});
```

Inject `ICryptoComRestClient` and `ICryptoComSocketClient`, or the registered Crypto.com REST/socket client interfaces used by the target project. `AddCryptoCom` also registers supported SharedApis interfaces from `ExchangeApi.SharedClient`.

## Safety Rules

- Read `references/safety.md` before generating trading, margin, derivatives, staking, withdrawal, transfer, leverage, or credential code.
- Prefer read-only examples for account inspection.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Derivatives examples can change leverage or create real exposure; make that explicit.
- Staking, unstaking, conversion, and withdrawal examples move or lock funds; require explicit user intent.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, derivatives, staking, withdrawal, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../CryptoCom.Net/Examples/ai-friendly/` for complete compilable examples.
