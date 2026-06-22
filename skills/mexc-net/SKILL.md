---
name: mexc-net
description: Build C#/.NET MEXC integrations with Mexc.Net, including JK.Mexc.Net package setup, SpotApi, FuturesApi, REST clients, websocket subscriptions, spot market data, futures market data, account balances, deposits, withdrawals, transfers, subaccounts, spot order placement/test orders/cancellation, futures leverage, futures position mode, futures positions, futures order placement/cancellation/editing, plan orders, TP/SL orders, trailing orders, MexcCredentials HMAC/RSA authentication, MEXC spot symbols, MEXC futures symbols, spot listen-key user streams, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, ExchangeCallResult shared helper handling, trackers, and SharedApis access. Use when the user asks for MEXC spot market data, MEXC account or trading code, MEXC futures, MEXC websocket updates, MEXC private streams, MEXC error handling, or converting raw MEXC API usage to idiomatic Mexc.Net.
---

# Mexc.Net

## Overview

Use `Mexc.Net` for MEXC-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for MEXC spot endpoints, futures endpoints, user streams, subaccounts, leverage, position mode, futures positions, plan orders, TP/SL orders, trailing orders, or MEXC credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the package:

```bash
dotnet add package JK.Mexc.Net
```

Use these namespaces in examples:

```csharp
using CryptoExchange.Net.Objects;
using Mexc.Net;
using Mexc.Net.Clients;
using Mexc.Net.Enums;
```

Add `using CryptoExchange.Net.SharedApis;` only for exchange-agnostic SharedApis code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new MexcRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new MexcSocketClient();
```

Primary REST API surfaces:

- `rest.SpotApi.ExchangeData`, `rest.SpotApi.Account`, `rest.SpotApi.Trading`, `rest.SpotApi.SubAccount`, `rest.SpotApi.SharedClient`
- `rest.FuturesApi.ExchangeData`, `rest.FuturesApi.Account`, `rest.FuturesApi.Trading`, `rest.FuturesApi.SharedClient`

Primary socket API surfaces:

- `socket.SpotApi`: spot public streams plus private account, order, and user trade streams
- `socket.FuturesApi`: futures public streams plus private futures user-data streams for balances, orders, positions, risk, ADL, plan orders, TP/SL, trailing orders, and liquidation updates

Mexc.Net does not expose Binance-style roots such as `UsdFuturesApi`, `CoinFuturesApi`, `V5Api`, or `UnifiedApi`. Use `FuturesApi` for MEXC futures.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new MexcRestClient();
```

Authenticated examples usually use HMAC credentials:

```csharp
var client = new MexcRestClient(options =>
{
    options.ApiCredentials = new MexcCredentials("API_KEY", "API_SECRET");
});
```

`MexcCredentials` also supports `HMACCredential`, `RSAXmlCredential`, and `RSAPemCredential`. Use placeholders, environment variables, or injected options in generated code.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared non-I/O helper methods can return `ExchangeCallResult<T>`. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTCUSDT");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols

- Spot symbols use concatenated names such as `BTCUSDT` and `ETHUSDT`.
- Futures symbols use underscore-separated names such as `BTC_USDT` and `ETH_USDT`.
- `MexcExchange.FormatSymbol("BTC", "USDT", TradingMode.Spot)` returns `BTCUSDT`.
- `MexcExchange.FormatSymbol("ETH", "USDT", TradingMode.PerpetualLinear)` returns `ETH_USDT`.
- Fetch spot metadata with `SpotApi.ExchangeData.GetExchangeInfoAsync(...)`.
- Fetch futures contract metadata with `FuturesApi.ExchangeData.GetSymbolAsync(...)` or `GetSymbolsAsync()`.

## REST Patterns

Public spot ticker:

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTCUSDT");
```

Authenticated spot account:

```csharp
var account = await client.SpotApi.Account.GetAccountInfoAsync();
```

Safe spot order validation:

```csharp
var testOrder = await client.SpotApi.Trading.PlaceTestOrderAsync(
    symbol: "BTCUSDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    price: 50000m);
```

Futures market data and position reads:

```csharp
var contract = await client.FuturesApi.ExchangeData.GetSymbolAsync("ETH_USDT");
var ticker = await client.FuturesApi.ExchangeData.GetTickerAsync("ETH_USDT");
var positions = await client.FuturesApi.Trading.GetPositionsAsync("ETH_USDT");
```

Futures trading uses `FuturesOrderSide` values that encode both direction and open/close action, such as `OpenLong`, `OpenShort`, `CloseLong`, and `CloseShort`.

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new MexcSocketClient();

var sub = await socket.SpotApi.SubscribeToMiniTickerUpdatesAsync(
    "BTCUSDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Spot private streams can automatically obtain and renew a listen key when credentials are configured:

```csharp
var socket = new MexcSocketClient(options =>
{
    options.ApiCredentials = new MexcCredentials("API_KEY", "API_SECRET");
});

var sub = await socket.SpotApi.SubscribeToOrderUpdatesAsync(
    update => Console.WriteLine(update.Data.OrderId));
```

If manually managing a listen key, start it with `rest.SpotApi.Account.StartUserStreamAsync()`, pass it to the subscription, keep it alive or handle `ListenkeyRenewed`, and stop it with `StopUserStreamAsync(...)`.

Futures private streams use the authenticated socket client directly through `socket.FuturesApi.SubscribeToUserDataUpdatesAsync(...)`.

Use `UnsubscribeAsync` or `UnsubscribeAllAsync` on shutdown. Do not leave example subscriptions running.

## SharedApis

Use SharedApis only when portability matters:

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new MexcRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")));
```

Mexc.Net exposes SharedApis from `SpotApi.SharedClient` and `FuturesApi.SharedClient` on REST and socket clients.

Do not mix Mexc-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddMexc(options =>
{
    options.ApiCredentials = new MexcCredentials("API_KEY", "API_SECRET");
});
```

Inject `IMexcRestClient` and `IMexcSocketClient`, or the registered Mexc REST/socket client interfaces used by the target project. `AddMexc` also registers supported SharedApis interfaces from `SpotApi.SharedClient` and `FuturesApi.SharedClient`.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, position mode, transfer, withdrawal, subaccount, private websocket, or credential code.
- Prefer read-only examples for account inspection.
- Prefer `PlaceTestOrderAsync` for spot order examples that only demonstrate syntax.
- Futures order placement, leverage changes, position mode changes, close-all, reverse-position, plan orders, TP/SL, and trailing orders affect live account state; make that explicit.
- Include cleanup for example live orders, listen keys, or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups, client roots, SharedApis, result types, and symbol formats.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, futures, transfer, withdrawal, websocket, tracker, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../Mexc.Net/Examples/ai-friendly/` for complete compilable examples.
