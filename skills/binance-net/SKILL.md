---
name: binance-net
description: Build C#/.NET Binance integrations with Binance.Net, including spot, USD-M futures, COIN-M futures, REST clients, websocket subscriptions, account reads, order placement/cancellation, BinanceCredentials, Binance-specific symbols, enums, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, and SharedApis access. Use when the user asks for Binance market data, Binance account or trading code, Binance futures, Binance websocket updates, Binance error handling, or converting raw Binance API usage to idiomatic Binance.Net.
---

# Binance.Net

## Overview

Use `Binance.Net` for Binance-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for Binance-only endpoint groups, Binance order types, Binance futures, user data streams, exchange filters, or Binance credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package Binance.Net
```

Use these namespaces in examples:

```csharp
using Binance.Net;
using Binance.Net.Clients;
using Binance.Net.Enums;
using Binance.Net.Objects;
using CryptoExchange.Net.Objects;
```

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new BinanceRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new BinanceSocketClient();
```

Primary API surfaces:

- `rest.SpotApi`: spot market data, spot account, spot trading
- `rest.UsdFuturesApi`: USD-M futures market data, futures account, futures trading
- `rest.CoinFuturesApi`: COIN-M futures market data, futures account, futures trading
- `socket.SpotApi`: spot websocket subscriptions and spot user data streams
- `socket.UsdFuturesApi`: USD-M futures websocket subscriptions and user data streams
- `socket.CoinFuturesApi`: COIN-M futures websocket subscriptions and user data streams

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new BinanceRestClient();
```

Private account, trading, and user stream calls need `BinanceCredentials`:

```csharp
var client = new BinanceRestClient(options =>
{
    options.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
});
```

Use placeholders or environment variables in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Always check `Success` before reading `Data`.

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

## Symbols And Enums

- Spot symbols usually use compact Binance format, for example `BTCUSDT`.
- USD-M futures symbols also commonly use compact format, for example `ETHUSDT`.
- COIN-M futures symbols can differ by contract; inspect exchange info when unsure.
- Spot order enums include `OrderSide`, `SpotOrderType`, and `TimeInForce`.
- Futures order enums include `OrderSide`, `FuturesOrderType`, `TimeInForce`, `PositionSide`, and `FuturesMarginType`.
- Use exchange info and filters before generating production order quantities or prices.

Let the library generate client order IDs unless the user needs external correlation.

## REST Patterns

Public spot ticker:

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTCUSDT");
```

Authenticated spot account:

```csharp
var account = await client.SpotApi.Account.GetAccountInfoAsync();
```

Spot limit order:

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTCUSDT",
    side: OrderSide.Buy,
    type: SpotOrderType.Limit,
    quantity: 0.001m,
    price: 50000m,
    timeInForce: TimeInForce.GoodTillCanceled);
```

USD-M futures leverage and order:

```csharp
await client.UsdFuturesApi.Account.ChangeInitialLeverageAsync("ETHUSDT", 5);

var order = await client.UsdFuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETHUSDT",
    side: OrderSide.Buy,
    type: FuturesOrderType.Market,
    quantity: 0.01m);
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new BinanceSocketClient();

var sub = await socket.SpotApi.ExchangeData.SubscribeToTickerUpdatesAsync(
    "BTCUSDT",
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

ISpotTickerRestClient tickers = new BinanceRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")));
```

Do not mix Binance-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddBinance(
    options =>
    {
        options.Rest.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
        options.Socket.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
    });
```

Inject `IBinanceRestClient` and `IBinanceSocketClient`, or the registered Binance REST/socket client interfaces used by the target project. Match the repository's existing DI pattern if one is present.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, futures, withdrawal, or credential code.
- Prefer read-only examples for account inspection.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Futures examples can change leverage and create real exposure; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, futures, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../Binance.Net/Examples/ai-friendly/` for complete compilable examples.
