---
name: lighter-net
description: Build C#/.NET Lighter DEX integrations with JKorf.Lighter.Net 1.1.0+, including ExchangeApi REST clients, websocket subscriptions and socket requests, account reads, order placement/cancellation, leverage and margin updates, LighterCredentials with EthKey, L1 signing, optional integrator fees, Lighter spot and perpetual symbols, signer-backed transactions, local order books, trackers, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, QueryResult socket request handling, and SharedApis access including IFundingRateRestClient. Use when the user asks for Lighter market data, Lighter account or trading code, Lighter DEX websocket updates, Lighter order books, Lighter tracker workflows, Lighter error handling, or converting raw Lighter API usage to idiomatic Lighter.Net.
---

# Lighter.Net

## Overview

Use `Lighter.Net` for Lighter DEX-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for Lighter-only endpoint groups, signer-backed transactions, websocket order requests, Lighter account indexes, local order books, or tracker workflows.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package JKorf.Lighter.Net
```

Use these namespaces in examples:

```csharp
using Lighter.Net;
using Lighter.Net.Clients;
using Lighter.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new LighterRestClient();
```

Create socket clients for websocket subscriptions and websocket request endpoints:

```csharp
var socket = new LighterSocketClient();
```

Primary API surfaces:

- `rest.ExchangeApi.ExchangeData`: status, system config, symbols, ticker/detail data, order book, trades, klines, funding, announcements, assets
- `rest.ExchangeApi.Account`: accounts, account limits, metadata, PnL, deposits, transfers, withdrawals, API keys, leverage, margin, integrator approval
- `rest.ExchangeApi.Trading`: place, edit, cancel, and query orders and user trades
- `socket.ExchangeApi.ExchangeData`: public trade, order book, book ticker, ticker, kline, and mark-price kline subscriptions
- `socket.ExchangeApi.Account`: account, user stats, balance subscriptions, leverage and margin websocket requests
- `socket.ExchangeApi.Trading`: order, user trade, position subscriptions, order placement, editing, and cancellation websocket requests

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new LighterRestClient();
```

Private account, trading, leverage, margin, and signer-backed transaction calls need `LighterCredentials` with an `EthKey`:

```csharp
var client = new LighterRestClient(options =>
{
    options.ApiCredentials = new LighterCredentials(
        ethKey: EthKey.FromPrivateKey("PRIVATE_KEY"),
        accountIndex: 123,
        apiKeyIndex: 5,
        apiSecret: "API_SECRET");
});
```

Use `EthKey.FromPrivateKey(...)` when L1 signing is needed. Use `EthKey.FromPublicKey(...)` only for authenticated API-key requests that do not require layer-1 signatures. Do not pass a raw public address string to `LighterCredentials` in version 1.1.0+.

Use placeholders or environment variables in generated code. Do not hardcode real credentials. `GenerateApiKey()` returns a local keypair result; it does not replace account setup or credential storage.

Lighter.Net uses Lighter's optional integrator-code mechanism by default. Set `IntegratorFeePercentage = 0m` or `null` on REST/socket options only when the user explicitly asks to disable the optional integrator fee.

## Result Handling

REST methods return `HttpResult<T>`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Websocket request methods, such as socket order placement and leverage updates, return `QueryResult<T>`. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.ExchangeApi.ExchangeData.GetSymbolDetailsAsync("ETH/USDC");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.SpotSymbols.First().LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Enums

- Spot symbols use slash format, for example `ETH/USDC`.
- Perpetual symbols commonly use base-only format, for example `ETH`.
- Use `GetSymbolsAsync` or `GetSymbolDetailsAsync` before production order placement.
- Common enums include `OrderSide`, `OrderType`, `TimeInForce`, `KlineInterval`, `MarketType`, `MarginMode`, and `PositionSide`.
- Limit order examples require an explicit `price` and `TimeInForce`.
- Let the library set nonce values unless the user explicitly needs manual nonce control.

## REST Patterns

Public spot ticker/detail data:

```csharp
var details = await client.ExchangeApi.ExchangeData.GetSymbolDetailsAsync("ETH/USDC");
```

Public perpetual order book:

```csharp
var book = await client.ExchangeApi.ExchangeData.GetOrderBookAsync("ETH", limit: 50);
```

Authenticated account data:

```csharp
var account = await client.ExchangeApi.Account.GetAccountsAsync();
```

Authenticated limit order:

```csharp
var order = await client.ExchangeApi.Trading.PlaceOrderAsync(
    symbol: "ETH",
    side: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 0.1m,
    price: 2000m,
    timeInForce: TimeInForce.GoodTillTime);
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new LighterSocketClient();

var sub = await socket.ExchangeApi.ExchangeData.SubscribeToSpotTickerUpdatesAsync(
    "ETH/USDC",
    update => Console.WriteLine(update.Data.Ticker.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

// On shutdown:
await socket.UnsubscribeAsync(sub.Data);
```

Socket order requests return `QueryResult<T>` rather than `WebSocketResult<UpdateSubscription>` because they are request/response operations over websocket.

## SharedApis

Use SharedApis only when portability matters:

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new LighterRestClient().ExchangeApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "ETH", "USDC")));
```

Do not mix Lighter-native request/model types with `SharedApis` request/model types.

## Order Books And Trackers

For local order books and trade trackers, use DI and resolve the Lighter factories:

```csharp
services.AddLighter();
```

Then resolve `ILighterOrderBookFactory` for local symbol order books or `ILighterTrackerFactory` for tracker workflows. See `references/usage.md` for compact examples.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddLighter(options =>
{
    options.Rest.ApiCredentials = new LighterCredentials(EthKey.FromPrivateKey("PRIVATE_KEY"), 123, 5, "API_SECRET");
    options.Socket.ApiCredentials = new LighterCredentials(EthKey.FromPrivateKey("PRIVATE_KEY"), 123, 5, "API_SECRET");
});
```

Inject `ILighterRestClient` and `ILighterSocketClient`, plus `ILighterOrderBookFactory`, `ILighterTrackerFactory`, or `ILighterUserClientProvider` when needed. Match the repository's existing DI pattern if one is present.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, margin, withdrawal, or credential code.
- Prefer read-only examples for account inspection.
- Prefer limit-order examples with explicit prices over market orders unless the user explicitly asks for immediate execution.
- Lighter transaction calls can change account tier, leverage, margin, and live orders; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, leverage, margin, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../Lighter.Net/Examples/ai-friendly/` for complete AI-oriented examples and `../Lighter.Net/AGENTS.md` for the library-maintained guidance.
