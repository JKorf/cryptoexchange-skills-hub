---
name: coinw-net
description: Build C#/.NET CoinW integrations with CoinW.Net, including SpotApi and FuturesApi REST clients, spot and futures websocket subscriptions, spot account balances, deposits, withdrawals, transfers, order placement/cancellation, futures margin mode, leverage, positions, take-profit/stop-loss, CoinWCredentials, CoinW spot underscore symbols, CoinW futures instrument symbols, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, CallResult batch-item handling, ExchangeCallResult shared helper handling, and SharedApis access. Use when the user asks for CoinW spot market data, CoinW account or trading code, CoinW futures, CoinW websocket updates, CoinW error handling, or converting raw CoinW API usage to idiomatic CoinW.Net.
---

# CoinW.Net

## Overview

Use `CoinW.Net` for CoinW-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for CoinW spot endpoints, CoinW futures endpoints, CoinW user streams, CoinW margin settings, or CoinW credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package CoinW.Net
```

Use these namespaces in examples:

```csharp
using CoinW.Net;
using CoinW.Net.Clients;
using CoinW.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for exchange-agnostic SharedApis code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new CoinWRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new CoinWSocketClient();
```

Primary API surfaces:

- `rest.SpotApi`: spot market data, spot account, spot trading
- `rest.FuturesApi`: futures market data, futures account settings, futures trading, positions
- `socket.SpotApi`: spot websocket market data and spot user streams
- `socket.FuturesApi`: futures websocket market data and futures user streams

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new CoinWRestClient();
```

Private account, trading, transfer, withdrawal, and user stream calls need `CoinWCredentials`:

```csharp
var client = new CoinWRestClient(options =>
{
    options.ApiCredentials = new CoinWCredentials("API_KEY", "API_SECRET");
});
```

Use placeholders or environment variables in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Always check `Success` before reading `Data`.

```csharp
var tickers = await client.SpotApi.ExchangeData.GetTickersAsync();
if (!tickers.Success)
{
    Console.WriteLine(tickers.Error);
    return;
}

var ticker = tickers.Data.SingleOrDefault(x => x.Symbol == "BTC_USDT");
Console.WriteLine(ticker?.LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Enums

- CoinW spot symbols use an underscore pair format, for example `BTC_USDT` and `ETH_USDT`.
- CoinW futures methods commonly use the base instrument symbol, for example `BTC` or `ETH`.
- Prefer `CoinWExchange.FormatSymbol("BTC", "USDT", TradingMode.Spot)` or SharedApis `SharedSymbol` when generating reusable symbol code.
- Spot order enums include `OrderSide` and `OrderType`.
- Futures order enums include `PositionSide`, `FuturesOrderType`, `QuantityUnit`, `MarginType`, `PositionCombineType`, and `TriggerOrderType`.
- Validate symbol rules, minimum quantities, quantity units, leverage, and margin mode before generating production order code.

Let the library generate client order IDs unless the user needs external correlation.

## REST Patterns

Public spot ticker:

```csharp
var tickers = await client.SpotApi.ExchangeData.GetTickersAsync();
var btcusdt = tickers.Success
    ? tickers.Data.SingleOrDefault(x => x.Symbol == "BTC_USDT")
    : null;
```

Authenticated spot balances:

```csharp
var balances = await client.SpotApi.Account.GetBalancesDetailsAsync();
```

Spot limit order:

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTC_USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    price: 50000m);
```

Futures margin mode and market order:

```csharp
await client.FuturesApi.Account.SetMarginModeAsync(
    MarginType.IsolatedMargin,
    PositionCombineType.Split);

var order = await client.FuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETH",
    side: PositionSide.Long,
    orderType: FuturesOrderType.Market,
    quantity: 1m,
    leverage: 5,
    quantityUnit: QuantityUnit.Contracts,
    marginType: MarginType.IsolatedMargin);
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new CoinWSocketClient();

var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
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

ISpotTickerRestClient tickers = new CoinWRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")));
```

Do not mix CoinW-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddCoinW(options =>
{
    options.ApiCredentials = new CoinWCredentials("API_KEY", "API_SECRET");
});
```

Inject `ICoinWRestClient` and `ICoinWSocketClient`, or the registered CoinW REST/socket client interfaces used by the target project. `AddCoinW` also registers supported SharedApis interfaces for spot and futures.

## Safety Rules

- Read `references/safety.md` before generating trading, margin, futures, withdrawal, transfer, or credential code.
- Prefer read-only examples for account inspection.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Futures examples can change margin mode, leverage, and create real exposure; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, futures, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../CoinW.Net/Examples/ai-friendly/` for complete compilable examples.
