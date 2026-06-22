---
name: bitget-net
description: Build C#/.NET Bitget integrations with Bitget.Net, including SpotApiV2, FuturesApiV2, CopyTradingFuturesV2, UnifiedApi, REST clients, websocket subscriptions, spot and futures account reads, order placement/cancellation, BitgetCredentials with passphrase, Bitget compact symbols, product types, enums, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, and SharedApis access. Use when the user asks for Bitget market data, Bitget account or trading code, Bitget futures, Bitget spot margin, Bitget copy trading, Bitget websocket updates, Bitget error handling, or converting raw Bitget API usage to idiomatic Bitget.Net.
---

# Bitget.Net

## Overview

Use `Bitget.Net` for Bitget-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for Bitget V2 endpoint groups, Bitget futures product types, spot margin, copy trading, UTA/unified account endpoints, Bitget order enums, or Bitget credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package JK.Bitget.Net
```

Use these namespaces in examples:

```csharp
using Bitget.Net;
using Bitget.Net.Clients;
using Bitget.Net.Enums;
using Bitget.Net.Enums.V2;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new BitgetRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new BitgetSocketClient();
```

Current source-checked API roots:

- `rest.SpotApiV2`: spot market data, spot account, spot trading, spot margin, spot SharedApis
- `rest.FuturesApiV2`: futures market data, futures account, futures trading, futures SharedApis
- `rest.CopyTradingFuturesV2`: copy trading trader and follower endpoints
- `rest.UnifiedApi`: UTA/unified account, exchange data, and trading endpoints
- `socket.SpotApiV2`: spot public/private websocket subscriptions and spot margin streams
- `socket.FuturesApiV2`: futures public/private websocket subscriptions
- `socket.UnifiedApi`: UTA/unified websocket endpoints

Do not use older or invented roots such as `SpotApi`, `FuturesApi`, `MarginApi`, `UsdFuturesApi`, `PerpetualFuturesApi`, `CopyTradingApi`, or `BrokerV2`.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new BitgetRestClient();
```

Private account, trading, margin, futures, copy trading, transfer, withdrawal, and user stream calls need `BitgetCredentials` with API key, secret, and passphrase:

```csharp
var client = new BitgetRestClient(options =>
{
    options.ApiCredentials = new BitgetCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
});
```

Use placeholders or environment variables in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.SpotApiV2.ExchangeData.GetTickersAsync("BTCUSDT");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.Single().LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols, Product Types, And Enums

- Bitget symbols are compact, for example `BTCUSDT`, `ETHUSDT`, `BGBUSDT`.
- Do not use `BTC-USDT`, `BTC/USDT`, `BTC_USDT`, or Bitfinex-style `tBTCUSD`.
- Futures calls generally require `BitgetProductTypeV2`, for example `BitgetProductTypeV2.UsdtFutures`.
- Futures trading calls commonly also require a margin asset such as `"USDT"`.
- Use `Bitget.Net.Enums` for `BitgetProductTypeV2` and websocket kline enums such as `BitgetStreamKlineIntervalV2`.
- Use `Bitget.Net.Enums.V2` for V2 order/account enums such as `OrderSide`, `OrderType`, `TimeInForce`, `MarginMode`, `TradeSide`, `PositionSide`, and `PositionMode`.

Let the library generate client order IDs unless the user needs external correlation.

## REST Patterns

Public spot ticker:

```csharp
var ticker = await client.SpotApiV2.ExchangeData.GetTickersAsync("BTCUSDT");
```

Authenticated spot balances:

```csharp
var balances = await client.SpotApiV2.Account.GetSpotBalancesAsync();
```

Spot limit order:

```csharp
var order = await client.SpotApiV2.Trading.PlaceOrderAsync(
    symbol: "BTCUSDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    timeInForce: TimeInForce.GoodTillCanceled,
    price: 1m);
```

Futures ticker, balances, and order:

```csharp
var productType = BitgetProductTypeV2.UsdtFutures;
const string symbol = "BTCUSDT";
const string marginAsset = "USDT";

var ticker = await client.FuturesApiV2.ExchangeData.GetTickerAsync(productType, symbol);
var balances = await client.FuturesApiV2.Account.GetBalancesAsync(productType);

var order = await client.FuturesApiV2.Trading.PlaceOrderAsync(
    productType: productType,
    symbol: symbol,
    marginAsset: marginAsset,
    side: OrderSide.Buy,
    type: OrderType.Limit,
    marginMode: MarginMode.CrossMargin,
    quantity: 0.001m,
    price: 1m,
    timeInForce: TimeInForce.GoodTillCanceled,
    tradeSide: TradeSide.Open);
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new BitgetSocketClient();

var sub = await socket.SpotApiV2.SubscribeToTickerUpdatesAsync(
    "BTCUSDT",
    update =>
    {
        var ticker = update.Data.FirstOrDefault();
        if (ticker != null)
            Console.WriteLine(ticker.LastPrice);
    });

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

ISpotTickerRestClient tickers = new BitgetRestClient().SpotApiV2.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")));
```

Bitget exposes shared clients on both `SpotApiV2` and `FuturesApiV2`. Do not mix Bitget-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddBitget(options =>
{
    options.Rest.ApiCredentials = new BitgetCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
    options.Socket.ApiCredentials = new BitgetCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
});
```

Inject `IBitgetRestClient` and `IBitgetSocketClient`, or the registered shared interfaces used by the target project. Match the repository's existing DI pattern if one is present.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, futures, margin, UTA, transfer, withdrawal, or credential code.
- Prefer read-only examples for account inspection.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Futures and margin examples can change leverage, borrow funds, or create real exposure; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, futures, margin, UTA, withdrawal, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../Bitget.Net/Examples/ai-friendly/` for complete compilable examples.
