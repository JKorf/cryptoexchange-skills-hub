---
name: bitmex-net
description: Build C#/.NET BitMEX integrations with BitMEX.Net, including ExchangeApi REST clients, websocket subscriptions, market data, account reads, balances, order placement/cancellation, positions, leverage, margin/risk settings, BitMEXCredentials, BitMEX symbols and quantity conversion helpers, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, ExchangeCallResult helper handling, and Shared API V2 strict capabilities and aggregates. Use when the user asks for BitMEX market data, BitMEX account or trading code, BitMEX spot or derivatives symbols, BitMEX websocket updates, BitMEX error handling, or converting raw BitMEX API usage to idiomatic BitMEX.Net.
---

# BitMEX.Net

## Overview

Use `BitMEX.Net` for BitMEX-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for BitMEX endpoint groups, BitMEX symbols, BitMEX quantity conversion, orders, positions, leverage, risk limits, margin settings, withdrawals, or BitMEX credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

This skill targets `JKorf.BitMEX.Net` 4.4.0, the latest published stable release verified on 2026-08-24.

Install the exchange package:

```bash
dotnet add package JKorf.BitMEX.Net --version 4.4.0
```

Use these namespaces in examples:

```csharp
using BitMEX.Net;
using BitMEX.Net.Clients;
using BitMEX.Net.Enums;
using BitMEX.Net.ExtensionMethods;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new BitMEXRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new BitMEXSocketClient();
```

Current source-checked API surfaces:

- `rest.ExchangeApi.ExchangeData`: symbols, active instruments, order books, trades, klines, funding, assets, indexes, settlements, insurance, liquidations, announcements
- `rest.ExchangeApi.Account`: account info, fees, balances, balance history, deposits, withdrawals, transfers, margin status, risk limit, margin transfer, saved addresses, API key info
- `rest.ExchangeApi.Trading`: execution history, orders, cancellations, user executions/trades, positions, cross/isolated leverage
- `rest.ExchangeApi.SharedApi`: exchange-agnostic SharedApis REST interfaces for spot and derivatives workflows
- `socket.ExchangeApi`: public and private websocket subscriptions for trades, klines, order books, book tickers, orders, balances, user trades, positions, settlements, and symbol updates
- `socket.ExchangeApi.SharedApi`: exchange-agnostic SharedApis socket interfaces

Do not use exchange roots from other libraries such as `SpotApi`, `UsdFuturesApi`, `FuturesApiV2`, `SpotApiV3`, `CoinFuturesApi`, or `PerpetualFuturesApi`.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new BitMEXRestClient();
```

Private account, trading, leverage, transfer, withdrawal, and user stream calls need `BitMEXCredentials` with API key and API secret:

```csharp
var client = new BitMEXRestClient(options =>
{
    options.ApiCredentials = new BitMEXCredentials("API_KEY", "API_SECRET");
});
```

Use placeholders or environment variables in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared non-I/O symbol/cache helpers can return `ExchangeCallResult<T>`. Always check `Success` before reading `Data`.

```csharp
var symbols = await client.ExchangeApi.ExchangeData.GetActiveSymbolsAsync();
if (!symbols.Success)
{
    Console.WriteLine(symbols.Error);
    return;
}

Console.WriteLine(symbols.Data.First().Symbol);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols, Quantities, And Enums

- Prefer `client.ExchangeApi.ExchangeData.GetActiveSymbolsAsync()` to discover exact BitMEX symbols.
- Common examples include `XBTUSD` for a perpetual contract.
- BitMEX also supports spot and dated futures symbols; do not assume Binance-style `BTCUSDT` naming.
- Use `BitMEXUtils.UpdateSymbolInfoAsync()` before `ToBitMEXSymbolQuantity`, `ToSharedSymbolQuantity`, or `ToSharedAssetQuantity`.
- Common order enums include `OrderSide`, `OrderType`, `TimeInForce`, `ExecutionInstruction`, and `BinPeriod`.

Let the library generate client order IDs unless the user needs external correlation. BitMEX examples often use explicit client IDs for traceability.

## REST Patterns

Public active symbols and order book:

```csharp
var symbols = await client.ExchangeApi.ExchangeData.GetActiveSymbolsAsync();
var orderBook = await client.ExchangeApi.ExchangeData.GetOrderBookAsync("XBTUSD", 25);
```

Authenticated balances:

```csharp
var balances = await client.ExchangeApi.Account.GetBalancesAsync();
```

Limit order:

```csharp
var order = await client.ExchangeApi.Trading.PlaceOrderAsync(
    symbol: "XBTUSD",
    orderSide: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 100,
    price: 1m,
    timeInForce: TimeInForce.GoodTillCancel,
    executionInstruction: ExecutionInstruction.PostOnly);
```

Positions and leverage:

```csharp
var positions = await client.ExchangeApi.Trading.GetPositionsAsync(
    filter: new Dictionary<string, object> { ["symbol"] = "XBTUSD" });

var leverage = await client.ExchangeApi.Trading.SetCrossMarginLeverageAsync("XBTUSD", leverage: 2m);
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new BitMEXSocketClient();

var sub = await socket.ExchangeApi.SubscribeToTradeUpdatesAsync(
    "XBTUSD",
    update =>
    {
        foreach (var trade in update.Data)
            Console.WriteLine($"{trade.Symbol}: {trade.Quantity} @ {trade.Price}");
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

## Shared API V2

Use Shared APIs only when portability matters. Depend on the narrow capability needed by the workflow:

```csharp
using CryptoExchange.Net.SharedApis;

using var client = new BitMEXRestClient();
IGetTickerRest ticker = client.ExchangeApi.SharedApi;

var result = await ticker.GetTickerAsync(
    new GetTickerRequest(
        new SharedSymbol(TradingMode.PerpetualInverse, "XBT", "USD")));
```

Use `IBitMEXSharedApiClient` as the exchange aggregate. Its aggregate properties—`Rest`, `Socket`—expose the supported Shared API surfaces at compile time. Use `GetCapability` only when the capability, trading mode, or transport is selected dynamically.

For WebSocket workflows, use the narrow subscription interface exposed by the applicable socket surface, such as `ISubscribeTickerSocket` or `ISubscribeTradesSocket`. Do not mix BitMEX-native request/model types with Shared API request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddBitMEX(options =>
{
    options.Rest.ApiCredentials = new BitMEXCredentials("API_KEY", "API_SECRET");
    options.Socket.ApiCredentials = new BitMEXCredentials("API_KEY", "API_SECRET");
});
```

Inject `IBitMEXRestClient` and `IBitMEXSocketClient`, or the registered shared interfaces used by the target project. Match the repository's existing DI pattern if one is present.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, margin, transfer, withdrawal, or credential code.
- Prefer read-only examples for account inspection.
- Prefer post-only/limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Leverage, risk limit, isolated margin, and margin transfer calls can change account risk; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, leverage, withdrawal, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../BitMEX.Net/Examples/ai-friendly/` for complete compilable examples.
