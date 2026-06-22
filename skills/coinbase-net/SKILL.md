---
name: coinbase-net
description: Build C#/.NET Coinbase integrations with Coinbase.Net, including AdvancedTradeApi REST clients, ExchangeApi market data, Advanced Trade and Exchange websocket subscriptions, accounts, portfolios, product metadata, spot and futures/perpetual trading, dry-run order workflows, deposits, withdrawals, transfers, ECDSA CoinbaseCredentials with key name and private key, dash-separated Coinbase product ids, dependency injection, HttpResult REST handling, CoinbaseOrderResult placement handling, WebSocketResult subscription handling, ExchangeCallResult shared helper handling, and SharedApis access. Use when the user asks for Coinbase Advanced Trade market data, Coinbase account or trading code, Coinbase websocket updates, Coinbase error handling, or converting raw Coinbase API usage to idiomatic Coinbase.Net.
---

# Coinbase.Net

## Overview

Use `Coinbase.Net` for Coinbase-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for Coinbase Advanced Trade, Coinbase App account endpoints, Coinbase Exchange market data, portfolios, Coinbase product ids, Coinbase ECDSA credentials, deposits, withdrawals, futures/perpetuals, or Coinbase websocket streams.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package JKorf.Coinbase.Net
```

Use these namespaces in examples:

```csharp
using Coinbase.Net;
using Coinbase.Net.Clients;
using Coinbase.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new CoinbaseRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new CoinbaseSocketClient();
```

Current source-checked API surfaces:

- `rest.AdvancedTradeApi.ExchangeData`: Advanced Trade products, candles, order books, trades, book tickers, fiat/crypto assets, exchange rates, buy/sell/spot prices
- `rest.AdvancedTradeApi.Account`: accounts, portfolios, portfolio transfers, futures/perpetual account settings and balances, fees, API key info, payment methods, converts, deposits, withdrawals, transactions, deposit addresses
- `rest.AdvancedTradeApi.Trading`: Advanced Trade orders, edits, cancels, fills, futures positions, perpetual positions, and close-position calls
- `rest.AdvancedTradeApi.SharedClient`: exchange-agnostic SharedApis REST interfaces for spot and Coinbase futures/perpetual workflows
- `rest.ExchangeApi.ExchangeData`: Coinbase Exchange server time, assets, and symbols
- `socket.AdvancedTradeApi`: Advanced Trade heartbeat, trades, klines, tickers, symbols, order book, user updates, and futures balance updates
- `socket.AdvancedTradeApi.SharedClient`: exchange-agnostic SharedApis socket interfaces for spot and Coinbase futures/perpetual workflows
- `socket.ExchangeApi`: Coinbase Exchange heartbeat, exchange info, ticker, batched ticker, and order book streams

Do not use exchange roots from other libraries such as `SpotApi`, `UsdFuturesApi`, `CoinFuturesApi`, `PerpetualFuturesApi`, `FuturesApiV2`, `SpotApiV3`, `V5Api`, or `ExchangeApi` as the primary trading root. In Coinbase.Net, `ExchangeApi` exists, but it is the Coinbase Exchange market-data surface, not the Advanced Trade trading surface.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new CoinbaseRestClient();
```

Private account, trading, transfer, deposit, withdrawal, portfolio, and private websocket calls need ECDSA `CoinbaseCredentials` with API key name and private key:

```csharp
var client = new CoinbaseRestClient(options =>
{
    options.ApiCredentials = new CoinbaseCredentials("API_KEY_NAME", "PRIVATE_KEY");
});
```

Use environment variables in runnable examples. Coinbase private keys may contain escaped newlines; normalize `\\n` to `\n` when reading from environment variables.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared non-I/O symbol/cache helpers can return `ExchangeCallResult<T>`. Always check `Success` before reading `Data`.

Order placement returns `HttpResult<CoinbaseOrderResult>` and requires a second success check:

```csharp
var order = await client.AdvancedTradeApi.Trading.PlaceOrderAsync(
    "ETH-USD",
    OrderSide.Buy,
    NewOrderType.Limit,
    quantity: 0.01m,
    price: 1000m);

if (!order.Success)
{
    Console.WriteLine(order.Error);
    return;
}

if (!order.Data.Success)
{
    Console.WriteLine(order.Data.ErrorResponse.Message);
    return;
}

Console.WriteLine(order.Data.SuccessResponse.OrderId);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Enums

- Coinbase product ids use dash format, for example `ETH-USD` and `BTC-USD`.
- Do not use Binance-style compact symbols such as `ETHUSDT`.
- Use `KlineInterval` for candles.
- Common trading enums include `OrderSide`, `NewOrderType`, `OrderStatus`, and `TimeInForce`.
- Inspect product metadata before generating production order quantities or prices.

Let the library generate client order IDs unless the user needs external correlation.

## REST Patterns

Public Advanced Trade product:

```csharp
var product = await client.AdvancedTradeApi.ExchangeData.GetSymbolAsync("ETH-USD");
```

Public candles and order book:

```csharp
var candles = await client.AdvancedTradeApi.ExchangeData.GetKlinesAsync(
    "ETH-USD",
    KlineInterval.OneMinute,
    limit: 5);

var book = await client.AdvancedTradeApi.ExchangeData.GetOrderBookAsync("ETH-USD", limit: 50);
```

Authenticated accounts:

```csharp
var accounts = await client.AdvancedTradeApi.Account.GetAccountsAsync(limit: 100);
```

Dry-run-first trading:

```csharp
var openOrders = await client.AdvancedTradeApi.Trading.GetOrdersAsync(
    symbols: new[] { "ETH-USD" },
    orderStatus: new[] { OrderStatus.Open },
    limit: 20);
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new CoinbaseSocketClient();

var sub = await socket.AdvancedTradeApi.SubscribeToTickerUpdatesAsync(
    "ETH-USD",
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

var shared = new CoinbaseRestClient().AdvancedTradeApi.SharedClient;
var info = shared.Discover();
Console.WriteLine($"{info.Exchange} {info.TypeName}");
```

Coinbase exposes shared clients on `AdvancedTradeApi.SharedClient`. Shared REST and socket support `TradingMode.Spot`, `TradingMode.PerpetualLinear`, and `TradingMode.DeliveryLinear`. Call `Discover()` before routing optional shared features. Do not mix Coinbase-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddCoinbase(options =>
{
    options.ApiCredentials = new CoinbaseCredentials("API_KEY_NAME", "PRIVATE_KEY");
});
```

Inject `ICoinbaseRestClient` and `ICoinbaseSocketClient`, or the registered shared interfaces used by the target project. Match the repository's existing DI pattern if one is present.

## Safety Rules

- Read `references/safety.md` before generating trading, futures/perpetuals, portfolio transfer, deposit, withdrawal, convert, close-position, or credential code.
- Prefer read-only examples for account inspection.
- Keep trading examples dry-run by default unless the user explicitly asks for order placement.
- Order placement requires both `HttpResult.Success` and `CoinbaseOrderResult.Success`.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, transfers, deposits, withdrawals, futures/perpetuals, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../Coinbase.Net/Examples/ai-friendly/` for complete compilable examples.
