---
name: whitebit-net
description: Build C#/.NET WhiteBit integrations with WhiteBit.Net, including WhiteBit.Net package setup, V4Api, REST clients, websocket subscriptions and request/response methods, spot and collateral/perpetual market data, balances, deposits, withdrawals, transfers, subaccounts, convert, codes, spot orders, collateral orders, OCO/conditional/OTO orders, leverage, hedge mode, positions, WhiteBitCredentials authentication, WhiteBit underscore and perpetual symbols, dependency injection, user client providers, local order books, trackers, HttpResult REST handling, WebSocketResult subscription handling, QueryResult socket request handling, ExchangeCallResult shared helper handling, and Shared API V2 strict capabilities and aggregates. Use when the user asks for WhiteBit market data, WhiteBit account or trading code, WhiteBit collateral/futures, WhiteBit websocket updates or requests, WhiteBit private streams, WhiteBit error handling, or converting raw WhiteBit API usage to idiomatic WhiteBit.Net.
---

# WhiteBit.Net

## Overview

Use `WhiteBit.Net` for WhiteBit-specific spot, collateral/perpetual, account, fund movement, subaccount, convert, code, and websocket workflows. Use `cryptoexchange-net` for portable workflows through SharedApis.

## Setup

This skill targets `WhiteBit.Net` 4.4.0, the latest published stable release verified on 2026-08-24.

```bash
dotnet add package WhiteBit.Net --version 4.4.0
```

```csharp
using CryptoExchange.Net.Objects;
using WhiteBit.Net;
using WhiteBit.Net.Clients;
using WhiteBit.Net.Enums;
```

## Client Root

```csharp
var rest = new WhiteBitRestClient();
var socket = new WhiteBitSocketClient();
```

WhiteBit uses one `V4Api` root:

- `rest.V4Api.ExchangeData`
- `rest.V4Api.Account`
- `rest.V4Api.Trading`: spot orders and general order history
- `rest.V4Api.CollateralTrading`: perpetual/collateral and margin orders/positions
- `rest.V4Api.SubAccount`, `Convert`, `Codes`, `SharedApi`
- `socket.V4Api`: public/private subscriptions and socket request/response methods
- `socket.V4Api.SharedApi`

Do not invent `SpotApi`, `FuturesApi`, or `UnifiedApi` roots.

## Credentials

Public data needs no credentials. Private calls use key and secret:

```csharp
var client = new WhiteBitRestClient(options =>
{
    options.ApiCredentials = new WhiteBitCredentials("API_KEY", "API_SECRET");
});
```

Use placeholders or secret storage. Do not add a passphrase; `WhiteBitCredentials` is HMAC key/secret.

## Result Handling

- REST: `HttpResult<T>` or `HttpResult`
- Socket subscriptions: `WebSocketResult<UpdateSubscription>`
- Socket request/response methods: `QueryResult<T>`
- Shared helpers: `ExchangeCallResult<T>`

Check `Success` before reading `Data`.

```csharp
var tickers = await client.V4Api.ExchangeData.GetTickersAsync();
if (!tickers.Success)
{
    Console.WriteLine(tickers.Error);
    return;
}
```

Retry only transient errors.

## Symbols And Products

- Native spot symbols use underscores, for example `ETH_USDT` and `BTC_USDT`.
- Perpetual collateral symbols use names such as `ETH_PERP`.
- Do not use compact `ETHUSDT` or dash-separated `ETH-USDT` in native calls.
- Validate trading status, minimum quantity/value, fees, and collateral product metadata before orders.
- Spot and collateral orders use `NewOrderType`; collateral orders can also use `PositionSide` and `reduceOnly`.

## REST Patterns

Spot ticker and balance:

```csharp
var tickers = await client.V4Api.ExchangeData.GetTickersAsync();
var balances = await client.V4Api.Account.GetSpotBalancesAsync();
```

Live spot order:

```csharp
var order = await client.V4Api.Trading.PlaceSpotOrderAsync(
    symbol: "ETH_USDT",
    side: OrderSide.Buy,
    type: NewOrderType.Limit,
    quantity: 0.01m,
    price: 2000m);
```

Collateral leverage and order:

```csharp
await client.V4Api.Account.SetAccountLeverageAsync(5);

var order = await client.V4Api.CollateralTrading.PlaceOrderAsync(
    symbol: "ETH_PERP",
    side: OrderSide.Buy,
    type: NewOrderType.Market,
    quantity: 0.1m);
```

These are live actions. Read `references/safety.md` first.

## Websocket Patterns

```csharp
var socket = new WhiteBitSocketClient();
var sub = await socket.V4Api.SubscribeToTickerUpdatesAsync(
    "ETH_USDT",
    update => Console.WriteLine(update.Data.Ticker.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Socket requests such as `GetTickerAsync`, `GetOrderBookAsync`, and private balance/order queries return `QueryResult<T>`, not subscription results.

## Shared API V2

Use Shared APIs only when portability matters. Depend on the narrow capability needed by the workflow:

```csharp
using CryptoExchange.Net.SharedApis;

using var client = new WhiteBitRestClient();
IGetTickerRest ticker = client.V4Api.SharedApi;

var result = await ticker.GetTickerAsync(
    new GetTickerRequest(
        new SharedSymbol(TradingMode.Spot, "ETH", "USDT")));
```

Use `IWhiteBitSharedApiClient` as the exchange aggregate. Its aggregate properties—`V4Rest`, `V4Socket`—expose the supported Shared API surfaces at compile time. Use `GetCapability` only when the capability, trading mode, or transport is selected dynamically.

For WebSocket workflows, use the narrow subscription interface exposed by the applicable socket surface, such as `ISubscribeTickerSocket` or `ISubscribeTradesSocket`. Do not mix WhiteBit-native request/model types with Shared API request/model types.

## Dependency Injection

```csharp
services.AddWhiteBit(options =>
{
    options.ApiCredentials = new WhiteBitCredentials("API_KEY", "API_SECRET");
});
```

Inject `IWhiteBitRestClient`, `IWhiteBitSocketClient`, `IWhiteBitOrderBookFactory`, `IWhiteBitTrackerFactory`, or `IWhiteBitUserClientProvider`.

## Safety Rules

- Read `references/safety.md` before generating credentials, orders, collateral exposure, leverage, hedge mode, withdrawal, transfer, convert confirmation, code, subaccount, kill-switch, or private stream code.
- Prefer market-data and read-only account examples.
- Close collateral positions with an opposite-side `reduceOnly: true` order.
- Treat cancel-all, kill-switch, withdrawal, transfer, convert confirmation, code application, and subaccount administration as live actions.
- Current source exposes live and custom environments, not a built-in testnet.

## References

- Read `references/api-surfaces.md` for V4 groups, socket requests, streams, SharedApis, result types, and symbols.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for trading, collateral, fund movement, administrative, websocket, and retry guardrails.
- In a maintainer checkout, read `../WhiteBit.Net/Examples/ai-friendly/` for compilable examples.
