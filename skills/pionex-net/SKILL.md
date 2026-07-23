---
name: pionex-net
description: Build C#/.NET Pionex integrations with Pionex.Net, including Pionex.Net package setup, SpotApi REST clients, public and private websocket subscriptions, symbols, tickers, trades, klines, order books, balances, full wallet balances, spot order placement/query/cancellation, user trades, PionexCredentials authentication, Pionex underscore-separated symbols, dependency injection, user client providers, order book/tracker factories, HttpResult REST handling, WebSocketResult subscription handling, and CryptoExchange.Net SharedApis access. Use when the user asks for Pionex spot market data, Pionex account or trading code, Pionex websocket updates, Pionex error handling, or converting raw Pionex API usage to idiomatic Pionex.Net. Current Pionex.Net does not expose a futures, margin, options, or derivatives client.
---

# Pionex.Net

## Overview

Use `Pionex.Net` for Pionex-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user needs native Pionex Spot endpoints, Pionex credentials, private streams, local order books, trackers, or Pionex symbol rules.

Use `cryptoexchange-net` when the same code must run across exchanges through `CryptoExchange.Net.SharedApis`.

Pionex.Net currently exposes Spot only. Do not invent futures, perpetual, margin, options, bot-management, or derivatives clients.

## Setup

Install:

```bash
dotnet add package Pionex.Net
```

Use:

```csharp
using CryptoExchange.Net.Objects;
using Pionex.Net;
using Pionex.Net.Clients;
using Pionex.Net.Enums;
```

Add `using CryptoExchange.Net.SharedApis;` only for SharedApis code.

## Client Roots

```csharp
var rest = new PionexRestClient();
var socket = new PionexSocketClient();
```

REST surfaces:

- `rest.SpotApi.ExchangeData`: server time, symbols, trades, books, tickers, book tickers, klines
- `rest.SpotApi.Account`: simple balances and full wallet/bot/trader balances
- `rest.SpotApi.Trading`: place, query, cancel orders and query fills
- `rest.SpotApi.SharedClient`: shared Spot REST interfaces

Socket surfaces:

- `socket.SpotApi`: public trades/order books and private orders/fills/balances
- `socket.SpotApi.SharedClient`: shared Spot socket interfaces

Direct socket methods sit on `SpotApi`; there are no socket `ExchangeData`, `Account`, or `Trading` children.

Read `references/api-surfaces.md` when selecting a method or shared interface.

## Credentials

Public market data needs no credentials:

```csharp
var client = new PionexRestClient();
```

Private account and trading methods require `PionexCredentials`:

```csharp
var client = new PionexRestClient(options =>
{
    options.ApiCredentials =
        new PionexCredentials("API_KEY", "API_SECRET");
});
```

Configure private socket credentials directly on `PionexSocketClient`. Pionex.Net has no listen-key workflow.

Use placeholders, environment variables, or injected options. Never hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Check `Success` before reading `Data`.

```csharp
var ticker = await client.SpotApi.ExchangeData
    .GetTickersAsync("BTC_USDT", SymbolType.Spot);

if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.Single().ClosePrice);
```

Ticker methods return arrays even with one symbol. Native `PionexTicker` uses `ClosePrice`; shared ticker models use `LastPrice`.

Use `result.Error.Code`, `Message`, `ErrorType`, and `IsTransient` for diagnostics. Retry only transient failures.

## Symbols and Orders

- Native symbols use `BASE_QUOTE`, for example `BTC_USDT`.
- Do not use `BTCUSDT`, `BTC-USDT`, or `BTC/USDT` in native calls.
- Query `GetSymbolsAsync` and validate `Enable`, precision, and quantity limits before trading.
- Limit orders use `quantity` and `price`.
- Market buys use `quoteQuantity`.
- Market sells use `quantity`.

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "ETH_USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.01m,
    price: 2000m);

if (!order.Success)
{
    Console.WriteLine(order.Error);
    return;
}

Console.WriteLine(order.Data.OrderId);
```

Pionex.Net does not expose a test-order method. Live placement, cancellation, and cancel-all methods affect the real account.

## Websocket Pattern

```csharp
var socket = new PionexSocketClient();

var sub = await socket.SpotApi.SubscribeToTradeUpdatesAsync(
    "BTC_USDT",
    update =>
    {
        foreach (var trade in update.Data)
            Console.WriteLine(trade.Price);
    });

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Direct public streams are trades and order books. Do not invent native ticker or kline subscriptions. Private streams are orders, user trades, and balances, authenticated by socket credentials.

Keep handlers fast and unsubscribe on shutdown.

## SharedApis

Use SharedApis when portability matters:

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers =
    new PionexRestClient().SpotApi.SharedClient;

var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(
        new SharedSymbol(TradingMode.Spot, "BTC", "USDT")));
```

Do not mix native Pionex request/model types with SharedApis request/model types. Use `SharedClient.Discover()` before depending on optional shared capabilities.

## Dependency Injection

```csharp
services.AddPionex(options =>
{
    options.ApiCredentials =
        new PionexCredentials("API_KEY", "API_SECRET");
});
```

Inject `IPionexRestClient`, `IPionexSocketClient`, `IPionexOrderBookFactory`, `IPionexTrackerFactory`, or `IPionexUserClientProvider`.

Only `PionexEnvironment.Live` is built in. Use `PionexEnvironment.CreateCustom(...)` for intentional custom endpoints; do not invent a testnet.

## Safety Rules

- Read `references/safety.md` before generating account, trading, cancel-all, private websocket, tracker, user-client-provider, or credential code.
- Prefer read-only examples for balances and order inspection.
- State clearly when examples call live order endpoints.
- Validate symbol state, precision, quantity rules, prices, and balances before a live order.
- Use client order IDs for reconciliation when appropriate.
- Never expose real API keys or secrets.

## References

- Read `references/api-surfaces.md` for client roots, method names, socket streams, shared interfaces, result types, and environments.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, order, websocket, retry, credential, and lifecycle guardrails.
- In a maintainer checkout with sibling repositories, read `../Pionex.Net/Examples/ai-friendly/` for complete compilable examples.
