---
name: gateio-net
description: Build C#/.NET Gate.io integrations with GateIo.Net, including SpotApi, PerpetualFuturesApi, RebateApi, AlphaApi, REST clients, websocket subscriptions, socket request methods, spot and perpetual futures market data, account balances, deposits, withdrawals, transfers, unified and margin account operations, order placement/cancellation, trigger orders, futures positions, leverage, GateIoCredentials, Gate.io underscore symbols, settlement assets, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, QueryResult socket request handling, ExchangeCallResult shared helper handling, and SharedApis access. Use when the user asks for Gate.io spot market data, Gate.io account or trading code, Gate.io futures, Gate.io websocket updates, Gate.io error handling, or converting raw Gate.io API usage to idiomatic GateIo.Net.
---

# GateIo.Net

## Overview

Use `GateIo.Net` for Gate.io-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for Gate.io spot endpoints, perpetual futures endpoints, socket order methods, user streams, margin/unified account operations, leverage, positions, or Gate.io credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package GateIo.Net
```

Use these namespaces in examples:

```csharp
using GateIo.Net;
using GateIo.Net.Clients;
using GateIo.Net.Enums;
using GateIo.Net.Objects;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for exchange-agnostic SharedApis code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new GateIoRestClient();
```

Create socket clients for websocket subscriptions and socket request methods:

```csharp
var socket = new GateIoSocketClient();
```

Primary REST API surfaces:

- `rest.SpotApi.ExchangeData`, `rest.SpotApi.Account`, `rest.SpotApi.Trading`, `rest.SpotApi.SharedClient`
- `rest.PerpetualFuturesApi.ExchangeData`, `rest.PerpetualFuturesApi.Account`, `rest.PerpetualFuturesApi.Trading`, `rest.PerpetualFuturesApi.SharedClient`
- `rest.RebateApi`: Gate.io rebate endpoints
- `rest.AlphaApi`: Gate.io Alpha endpoints

Primary socket API surfaces:

- `socket.SpotApi`: public spot streams, private spot user streams, and spot socket order request methods
- `socket.PerpetualFuturesApi`: public futures streams, private futures user streams, and futures socket order request methods

GateIo.Net does not expose Binance-style roots such as `UsdFuturesApi`, `CoinFuturesApi`, or `PortfolioMarginApi`. Use `PerpetualFuturesApi` for Gate.io perpetual futures.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new GateIoRestClient();
```

Private account, trading, transfer, withdrawal, authenticated socket, and futures user stream calls need `GateIoCredentials` with API key and secret:

```csharp
var client = new GateIoRestClient(options =>
{
    options.ApiCredentials = new GateIoCredentials("API_KEY", "API_SECRET");
});
```

Use placeholders or environment variables in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Websocket request/order methods return `QueryResult<T>` or `QueryResult`. Some SharedApis symbol helper methods return `ExchangeCallResult<T>`. Always check `Success` before reading `Data`.

```csharp
var tickerResult = await client.SpotApi.ExchangeData.GetTickersAsync("ETH_USDT");
if (!tickerResult.Success)
{
    Console.WriteLine(tickerResult.Error);
    return;
}

Console.WriteLine(tickerResult.Data.First().LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Enums

- Gate.io native spot symbols use underscores, for example `ETH_USDT`.
- Perpetual futures contracts also use underscore symbols, for example `ETH_USDT`, and most native futures methods also require `settlementAsset`, for example `"usdt"`.
- `GateIoExchange.FormatSymbol("ETH", "USDT", TradingMode.Spot)` returns `ETH_USDT`.
- SharedApis can format symbols through `SharedSymbol`.
- Spot order placement uses `OrderSide`, `NewOrderType`, `TimeInForce`, `SpotAccountType`, and optional trigger-order enums.
- Futures order placement uses `OrderSide`, integer contract quantities, `TimeInForce`, and optional reduce-only, close, trigger, margin, and position-mode parameters.

Let the library generate client order IDs unless the user needs external correlation.

## REST Patterns

Public spot ticker:

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickersAsync("ETH_USDT");
```

Authenticated spot balances:

```csharp
var balances = await client.SpotApi.Account.GetBalancesAsync();
```

Spot limit order:

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "ETH_USDT",
    side: OrderSide.Buy,
    type: NewOrderType.Limit,
    quantity: 0.01m,
    price: 2000m,
    timeInForce: TimeInForce.GoodTillCancel);
```

Perpetual futures contract metadata, leverage, and market-style order:

```csharp
const string settlementAsset = "usdt";
const string contract = "ETH_USDT";

var contractInfo = await client.PerpetualFuturesApi.ExchangeData.GetContractAsync(settlementAsset, contract);

await client.PerpetualFuturesApi.Trading.UpdatePositionLeverageAsync(
    settlementAsset,
    contract,
    leverage: 5);

var order = await client.PerpetualFuturesApi.Trading.PlaceOrderAsync(
    settlementAsset: settlementAsset,
    contract: contract,
    orderSide: OrderSide.Buy,
    quantity: 1,
    price: 0m,
    timeInForce: TimeInForce.ImmediateOrCancel);
```

Gate.io futures quantity is an integer number of contracts, not a decimal asset quantity. Read contract metadata before sizing orders.

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new GateIoSocketClient();

var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "ETH_USDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Futures public subscriptions use settlement asset plus contract:

```csharp
var futuresSub = await socket.PerpetualFuturesApi.SubscribeToTickerUpdatesAsync(
    "usdt",
    "ETH_USDT",
    update => Console.WriteLine(update.Data.FirstOrDefault()?.LastPrice));
```

Native socket order request methods, such as `socket.SpotApi.PlaceOrderAsync(...)` and `socket.PerpetualFuturesApi.PlaceOrderAsync(...)`, return `QueryResult<T>`, not `HttpResult<T>`.

Use `UnsubscribeAsync` or `UnsubscribeAllAsync` on shutdown. Do not leave example subscriptions running.

## SharedApis

Use SharedApis only when portability matters:

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new GateIoRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "ETH", "USDT")));
```

GateIo.Net exposes SharedApis from `SpotApi.SharedClient` and `PerpetualFuturesApi.SharedClient` on both REST and socket clients. `RebateApi` and `AlphaApi` are Gate.io-specific native REST surfaces and are not SharedApis roots.

Do not mix GateIo-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddGateIo(options =>
{
    options.ApiCredentials = new GateIoCredentials("API_KEY", "API_SECRET");
});
```

Inject `IGateIoRestClient` and `IGateIoSocketClient`, or the registered GateIo REST/socket client interfaces used by the target project. `AddGateIo` also registers supported SharedApis interfaces from `SpotApi.SharedClient` and `PerpetualFuturesApi.SharedClient`.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, margin/unified account, futures, transfer, withdrawal, or credential code.
- Prefer read-only examples for account inspection.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Futures examples can change leverage and create real exposure; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups, client roots, SharedApis, result types, and symbol formats.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, futures, transfer, withdrawal, websocket, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../GateIo.Net/Examples/ai-friendly/` for complete compilable examples.
