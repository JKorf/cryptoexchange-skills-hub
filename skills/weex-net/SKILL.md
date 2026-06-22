---
name: weex-net
description: Build C#/.NET Weex integrations with Weex.Net, including Weex.Net package setup, SpotApi, FuturesApi, REST clients, websocket subscriptions, spot and futures market data, account balances, bills, transfer/deposit/withdrawal history, spot and futures orders, conditional orders, TP/SL orders, leverage, margin mode, positions, WeexCredentials key/secret/passphrase authentication, compact Weex symbols, dependency injection, user client providers, local order books, trackers, HttpResult REST handling, WebSocketResult subscription handling, ExchangeCallResult shared helper handling, and SharedApis access. Use when the user asks for Weex spot market data, Weex account or trading code, Weex futures, Weex websocket updates, Weex private streams, Weex error handling, or converting raw Weex API usage to idiomatic Weex.Net.
---

# Weex.Net

## Overview

Use `Weex.Net` for Weex-specific C# code involving spot, futures, account data, trading, private streams, leverage, margin modes, conditional orders, or Weex credentials. Use `cryptoexchange-net` for portable workflows through `CryptoExchange.Net.SharedApis`.

## Setup

```bash
dotnet add package Weex.Net
```

```csharp
using CryptoExchange.Net.Objects;
using Weex.Net;
using Weex.Net.Clients;
using Weex.Net.Enums;
```

## Client Roots

```csharp
var rest = new WeexRestClient();
var socket = new WeexSocketClient();
```

REST surfaces:

- `rest.SpotApi.ExchangeData`, `Account`, `Trading`, `SharedClient`
- `rest.FuturesApi.ExchangeData`, `Account`, `Trading`, `SharedClient`

Socket surfaces:

- `socket.SpotApi`: public market streams and private account/order/trade streams
- `socket.FuturesApi`: public market streams and private account/position/order/trade streams
- `SharedClient` on each socket API

Do not use Binance `UsdFuturesApi`/`CoinFuturesApi` or OKX `UnifiedApi`. Weex uses `FuturesApi`.

## Credentials

Public market data does not require credentials. Private endpoints require key, secret, and passphrase:

```csharp
var client = new WeexRestClient(options =>
{
    options.ApiCredentials = new WeexCredentials(
        "API_KEY",
        "API_SECRET",
        "API_PASSPHRASE");
});
```

Never omit the passphrase or hardcode real credentials.

## Result Handling

REST calls return `HttpResult<T>` or `HttpResult`. Subscriptions return `WebSocketResult<UpdateSubscription>`. Shared cache helpers can return `ExchangeCallResult<T>`. Check `Success` before reading `Data`.

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickersAsync(new[] { "ETHUSDT" });
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.Single().LastPrice);
```

Retry only transient errors.

## Symbols And Order Types

- Spot and futures commonly use compact symbols such as `ETHUSDT` and `BTCUSDT`.
- Validate symbols and precision with exchange info before trading.
- Spot orders use `OrderSide`, `OrderType`, and `TimeInForce`.
- Regular futures `PlaceOrderAsync` also uses `OrderType`.
- Futures conditional orders use `FuturesOrderType` with `PlaceConditionalOrderAsync`.
- Futures orders require `PositionSide`; leverage methods require appropriate `MarginType` values.

## REST Patterns

Spot ticker and account:

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickersAsync(new[] { "ETHUSDT" });
var account = await client.SpotApi.Account.GetAccountInfoAsync();
```

Live spot limit order:

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "ETHUSDT",
    side: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 0.1m,
    price: 2000m,
    timeInForce: TimeInForce.GoodTillCanceled);
```

Futures leverage and order:

```csharp
await client.FuturesApi.Account.SetLeverageAsync(
    symbol: "ETHUSDT",
    marginMode: MarginType.Isolated,
    isolatedLongLeverage: 5,
    isolatedShortLeverage: 5);

var order = await client.FuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETHUSDT",
    side: OrderSide.Buy,
    positionSide: PositionSide.Long,
    orderType: OrderType.Market,
    quantity: 0.01m);
```

These are live account actions. Read `references/safety.md` first.

## Websocket Pattern

```csharp
var socket = new WeexSocketClient();
var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "ETHUSDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Private subscriptions require `WeexCredentials`. Keep handlers fast and unsubscribe on shutdown.

## SharedApis

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new WeexRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "ETH", "USDT")));
```

Shared clients are available from both `SpotApi` and `FuturesApi` on REST and socket clients. Do not mix native and SharedApis models.

## Dependency Injection

```csharp
services.AddWeex(options =>
{
    options.ApiCredentials = new WeexCredentials(
        "API_KEY", "API_SECRET", "API_PASSPHRASE");
});
```

Inject `IWeexRestClient`, `IWeexSocketClient`, `IWeexOrderBookFactory`, `IWeexTrackerFactory`, or `IWeexUserClientProvider`.

## Safety Rules

- Read `references/safety.md` before generating credentials, account, trading, leverage, margin, fund-history, conditional order, TP/SL, or private stream code.
- Prefer read-only account and market-data examples.
- Validate tick size, step size, and quantity limits using exchange info.
- Make live orders, leverage changes, position closing, and cancel-all behavior explicit.
- Current source exposes `WeexEnvironment.Live` and custom environments; do not claim a built-in testnet.

## References

- Read `references/api-surfaces.md` for endpoint groups, shared interfaces, streams, result types, and order-type distinctions.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for credential, trading, futures, websocket, transfer, and retry guardrails.
- In a maintainer checkout, read `../Weex.Net/Examples/ai-friendly/` for compilable examples.
