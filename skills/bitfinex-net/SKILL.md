---
name: bitfinex-net
description: Build C#/.NET Bitfinex integrations with Bitfinex.Net, including spot, margin positions, funding, REST clients, websocket subscriptions, account reads, order placement/cancellation, BitfinexCredentials, Bitfinex-native t/f symbols, enums, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, and SharedApis access. Use when the user asks for Bitfinex market data, Bitfinex account or trading code, Bitfinex funding, Bitfinex websocket updates, Bitfinex error handling, or converting raw Bitfinex API usage to idiomatic Bitfinex.Net.
---

# Bitfinex.Net

## Overview

Use `Bitfinex.Net` for Bitfinex-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for Bitfinex-only endpoint groups, Bitfinex symbols, margin positions, funding offers, wallet types, order flags, or Bitfinex credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package Bitfinex.Net
```

Use these namespaces in examples:

```csharp
using Bitfinex.Net;
using Bitfinex.Net.Clients;
using Bitfinex.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new BitfinexRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new BitfinexSocketClient();
```

Primary API surfaces:

- `rest.ExchangeApi.ExchangeData`: public platform status, symbols, tickers, trades, order books, klines, funding market data, derivatives status, liquidation data
- `rest.ExchangeApi.Account`: wallets, margin info, movements, ledgers, user info, fees, deposit addresses, transfers, withdrawals
- `rest.ExchangeApi.Trading`: open/closed orders, user trades, positions, order placement/cancellation, position actions
- `rest.GeneralApi.Funding`: active offers, funding history, submit/cancel offers, loans, credits, funding info, auto-renew
- `socket.ExchangeApi`: public ticker, funding ticker, order book, raw book, trades, klines, liquidations, derivatives, user updates, websocket order/funding commands

Bitfinex.Net uses a single `ExchangeApi` root instead of product-specific API roots. Spot, margin, funding-market, and derivatives-related market data is available under `ExchangeApi.ExchangeData`; socket subscriptions are under `socket.ExchangeApi`. Verify trading support from the local source before generating derivatives trading code.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new BitfinexRestClient();
```

Private account, trading, funding management, and user stream calls need `BitfinexCredentials`:

```csharp
var client = new BitfinexRestClient(options =>
{
    options.ApiCredentials = new BitfinexCredentials("API_KEY", "API_SECRET");
});
```

Use placeholders or environment variables in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.ExchangeApi.ExchangeData.GetTickerAsync("tBTCUSD");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Enums

- Trading pairs use the Bitfinex `t` prefix, for example `tBTCUSD`.
- Funding currencies use the Bitfinex `f` prefix, for example `fUSD`.
- Derivatives symbols keep the Bitfinex format, for example `tBTCF0:USTF0`.
- SharedApis symbols do not use Bitfinex prefixes; use `new SharedSymbol(TradingMode.Spot, "BTC", "USD")`.
- Spot exchange-wallet limit orders use `OrderType.ExchangeLimit`.
- Margin orders use non-exchange order types such as `OrderType.Limit`.
- Funding offers use `FundingOrderType` on REST funding endpoints and funding symbols such as `fUSD`.

Let the library generate order IDs unless the user needs external correlation.

## REST Patterns

Public spot ticker:

```csharp
var ticker = await client.ExchangeApi.ExchangeData.GetTickerAsync("tBTCUSD");
```

Authenticated wallet balances:

```csharp
var wallets = await client.ExchangeApi.Account.GetBalancesAsync();
```

Spot exchange limit order:

```csharp
var order = await client.ExchangeApi.Trading.PlaceOrderAsync(
    symbol: "tBTCUSD",
    side: OrderSide.Buy,
    type: OrderType.ExchangeLimit,
    quantity: 0.001m,
    price: 50000m);
```

Margin positions:

```csharp
var positions = await client.ExchangeApi.Trading.GetPositionsAsync();
```

Funding offer:

```csharp
var offer = await client.GeneralApi.Funding.SubmitFundingOfferAsync(
    fundingOrderType: FundingOrderType.Limit,
    symbol: "fUSD",
    quantity: 100m,
    rate: 0.0002m,
    period: 2);
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new BitfinexSocketClient();

var sub = await socket.ExchangeApi.SubscribeToTickerUpdatesAsync(
    "tBTCUSD",
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

ISpotTickerRestClient tickers = new BitfinexRestClient().ExchangeApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USD")));
```

Do not mix Bitfinex-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddBitfinex(options =>
{
    options.Rest.ApiCredentials = new BitfinexCredentials("API_KEY", "API_SECRET");
    options.Socket.ApiCredentials = new BitfinexCredentials("API_KEY", "API_SECRET");
});
```

Inject `IBitfinexRestClient` and `IBitfinexSocketClient`, or the registered shared interfaces used by the target project. Match the repository's existing DI pattern if one is present.

## Safety Rules

- Read `references/safety.md` before generating trading, margin, funding, withdrawal, or credential code.
- Prefer read-only examples for account inspection.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Margin, derivatives, and funding workflows can create real exposure; make that explicit.
- Include cleanup for example orders, funding offers, or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, funding, withdrawal, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../Bitfinex.Net/Examples/ai-friendly/` for complete compilable examples.
