---
name: toobit-net
description: Build C#/.NET Toobit integrations with Toobit.Net, including Toobit.Net package setup, SpotApi, UsdtFuturesApi, REST clients, websocket subscriptions, spot market data, USDT futures market data, account balances, deposits, withdrawals, transfers, subaccounts, spot order placement/test/cancellation, futures order placement/cancellation, leverage, margin type, positions, trading stops, user data streams, ToobitCredentials authentication, Toobit spot symbols, Toobit USDT futures symbols, dependency injection, user client providers, order book/tracker factories, HttpResult REST handling, WebSocketResult subscription handling, ExchangeCallResult shared helper handling, and SharedApis access. Use when the user asks for Toobit spot market data, Toobit account or trading code, Toobit USDT futures, Toobit websocket updates, Toobit user streams, Toobit error handling, or converting raw Toobit API usage to idiomatic Toobit.Net.
---

# Toobit.Net

## Overview

Use `Toobit.Net` for Toobit-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for Toobit spot, USDT futures, Toobit-specific symbols, listen-key user streams, leverage, margin type, local order books, trackers, or Toobit credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the package:

```bash
dotnet add package Toobit.Net
```

Use these namespaces in examples:

```csharp
using CryptoExchange.Net.Objects;
using Toobit.Net;
using Toobit.Net.Clients;
using Toobit.Net.Enums;
```

Add `using CryptoExchange.Net.SharedApis;` only for exchange-agnostic SharedApis code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new ToobitRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new ToobitSocketClient();
```

Primary REST API surfaces:

- `rest.SpotApi.ExchangeData`, `Account`, `Trading`, `SharedClient`
- `rest.UsdtFuturesApi.ExchangeData`, `Account`, `Trading`, `SharedClient`

Primary socket API surfaces:

- `socket.SpotApi`: spot public streams, user data streams, `SharedClient`
- `socket.UsdtFuturesApi`: futures public streams, user data streams, `SharedClient`

Toobit.Net does not expose Binance `UsdFuturesApi`, Binance `CoinFuturesApi`, or OKX-style `UnifiedApi`. Use `UsdtFuturesApi` for Toobit USDT futures.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new ToobitRestClient();
```

Private account, trading, transfer, withdrawal, and user stream calls need `ToobitCredentials`:

```csharp
var client = new ToobitRestClient(options =>
{
    options.ApiCredentials = new ToobitCredentials("API_KEY", "API_SECRET");
});
```

Use placeholders, environment variables, or injected options in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared helper methods can return `ExchangeCallResult<T>`. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickersAsync("BTCUSDT");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.Single().LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Enums

- Spot symbols use compact names such as `BTCUSDT`.
- USDT futures symbols use contract names such as `ETH-SWAP-USDT`.
- Futures index-price methods can use index symbols such as `ETHUSDT`.
- Spot orders use `OrderSide`, `OrderType`, and `TimeInForce`.
- Futures orders use `FuturesOrderSide`, `FuturesNewOrderType`, `FuturesOrderType`, `PriceType`, `PositionSide`, `MarginType`, and `TriggerType`.
- Use exchange info, balances, leverage info, and filters before production trading code.

## REST Patterns

Public spot ticker:

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickersAsync("BTCUSDT");
```

Authenticated spot balances:

```csharp
var balances = await client.SpotApi.Account.GetBalancesAsync();
```

Spot order syntax validation:

```csharp
var test = await client.SpotApi.Trading.PlaceTestOrderAsync(
    symbol: "BTCUSDT",
    orderSide: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 0.001m,
    timeInForce: TimeInForce.GoodTillCanceled,
    price: 50000m);
```

Live spot order:

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTCUSDT",
    orderSide: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 0.001m,
    timeInForce: TimeInForce.GoodTillCanceled,
    price: 50000m);
```

USDT futures leverage and order:

```csharp
const string contract = "ETH-SWAP-USDT";

await client.UsdtFuturesApi.Account.SetLeverageAsync(contract, 10);

var order = await client.UsdtFuturesApi.Trading.PlaceOrderAsync(
    symbol: contract,
    orderSide: FuturesOrderSide.BuyOpen,
    orderType: FuturesNewOrderType.Limit,
    quantity: 10,
    price: 2000m,
    timeInForce: TimeInForce.GoodTillCanceled);
```

Use `PlaceTestOrderAsync` for safe spot order examples when possible. Futures leverage, margin type, live orders, trading stops, transfers, and withdrawals affect live account state.

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new ToobitSocketClient();

var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "BTCUSDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Authenticated user streams can be subscribed with credentials directly or with a listen key from `StartUserStreamAsync`. Use `UnsubscribeAsync` or `UnsubscribeAllAsync` on shutdown.

## SharedApis

Use SharedApis only when portability matters:

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new ToobitRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")));
```

Toobit.Net exposes SharedApis from `SpotApi.SharedClient` and `UsdtFuturesApi.SharedClient` on REST and socket clients. Do not mix Toobit-native request/model types with SharedApis request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddToobit(options =>
{
    options.ApiCredentials = new ToobitCredentials("API_KEY", "API_SECRET");
});
```

Inject `IToobitRestClient`, `IToobitSocketClient`, `IToobitOrderBookFactory`, `IToobitTrackerFactory`, or `IToobitUserClientProvider` as needed.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, margin type, transfer, withdrawal, private websocket, user stream, tracker, or credential code.
- Prefer read-only examples for account inspection.
- Prefer `PlaceTestOrderAsync` for spot order syntax/risk examples when possible.
- Include cleanup for example live orders or subscriptions when appropriate.
- Be explicit that futures leverage, margin type, live orders, trading stops, transfers, withdrawals, and cancel-all methods affect live account state.

## References

- Read `references/api-surfaces.md` for endpoint groups, client roots, SharedApis, socket streams, result types, user streams, and symbol formats.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, futures, websocket, tracker, credential, and transfer guardrails.
- In a maintainer checkout with sibling repositories, read `../Toobit.Net/Examples/ai-friendly/` for complete compilable examples.
