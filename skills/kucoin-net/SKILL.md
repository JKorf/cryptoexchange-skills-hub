---
name: kucoin-net
description: Build C#/.NET KuCoin integrations with Kucoin.Net, including Kucoin.Net package setup, SpotApi, FuturesApi, UnifiedApi, REST clients, websocket subscriptions, spot market data, futures contract market data, account balances, deposits, withdrawals, transfers, Earn, margin, high-frequency spot trading, spot order placement/test orders/cancellation, futures leverage, futures positions, futures order placement/cancellation, KucoinCredentials key/secret/passphrase authentication, Kucoin spot symbols, Kucoin futures contract symbols, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, ExchangeCallResult shared helper handling, trackers, and SharedApis access. Use when the user asks for Kucoin spot market data, Kucoin account or trading code, Kucoin futures, Kucoin unified account workflows, Kucoin websocket updates, Kucoin Earn, Kucoin margin, Kucoin error handling, or converting raw Kucoin API usage to idiomatic Kucoin.Net.
---

# Kucoin.Net

## Overview

Use `Kucoin.Net` for KuCoin-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for KuCoin spot endpoints, futures contract endpoints, Unified account endpoints, user streams, margin, Earn, high-frequency spot trading, leverage, positions, or KuCoin credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the package:

```bash
dotnet add package Kucoin.Net
```

Use these namespaces in examples:

```csharp
using CryptoExchange.Net.Objects;
using Kucoin.Net;
using Kucoin.Net.Clients;
using Kucoin.Net.Enums;
```

Add `using CryptoExchange.Net.SharedApis;` only for exchange-agnostic SharedApis code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new KucoinRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new KucoinSocketClient();
```

Primary REST API surfaces:

- `rest.SpotApi.ExchangeData`, `rest.SpotApi.Account`, `rest.SpotApi.SubAccount`, `rest.SpotApi.Trading`, `rest.SpotApi.HfTrading`, `rest.SpotApi.Margin`, `rest.SpotApi.Earn`, `rest.SpotApi.SharedClient`
- `rest.FuturesApi.ExchangeData`, `rest.FuturesApi.Account`, `rest.FuturesApi.Trading`, `rest.FuturesApi.SharedClient`
- `rest.UnifiedApi.Account`, `rest.UnifiedApi.ExchangeData`, `rest.UnifiedApi.Trading`

Primary socket API surfaces:

- `socket.SpotApi`: spot public streams plus private spot order, balance, stop-order, and margin streams
- `socket.FuturesApi`: futures public streams plus private futures balance, order, position, margin mode, leverage, and stop-order streams
- `socket.UnifiedApi`: Unified account ticker, kline, order book, trade, balance, order, user trade, position, leverage, and liquidation-warning streams

Kucoin.Net does not expose Binance-style roots such as `UsdFuturesApi`, `CoinFuturesApi`, or `UsdFuturesApi`. Use `FuturesApi` for KuCoin futures contracts and `UnifiedApi` for KuCoin Unified account endpoints.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new KucoinRestClient();
```

Authenticated KuCoin endpoints require key, secret, and API passphrase:

```csharp
var client = new KucoinRestClient(options =>
{
    options.ApiCredentials = new KucoinCredentials(
        "API_KEY",
        "API_SECRET",
        "API_PASSPHRASE");
});
```

Use placeholders, environment variables, or injected options in generated code. Do not omit the passphrase for account, trading, private websocket, transfer, withdrawal, futures, or Unified examples.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared non-I/O helper methods can return `ExchangeCallResult<T>`. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTC-USDT");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols

- Spot symbols use dash-separated names such as `BTC-USDT` and `ETH-USDT`.
- Futures symbols are contract names such as `ETHUSDTM` and `XBTUSDTM`; do not use spot symbols for futures endpoints.
- Unified endpoints generally use KuCoin symbol strings such as `ETH-USDT`, with trade/account type parameters on Unified socket methods.
- Fetch spot metadata with `SpotApi.ExchangeData.GetSymbolAsync(...)` or `GetSymbolsAsync(...)` and use increments before placing orders.
- Fetch futures contract metadata with `FuturesApi.ExchangeData.GetContractAsync(...)` or `GetSymbolsAsync(...)`.
- `KucoinExchange.FormatSymbol("BTC", "USDT", TradingMode.Spot)` returns KuCoin's native spot symbol format.

## REST Patterns

Public spot ticker:

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTC-USDT");
```

Authenticated spot balances:

```csharp
var accounts = await client.SpotApi.Account.GetAccountsAsync();
```

Safe spot order validation:

```csharp
var testOrder = await client.SpotApi.Trading.PlaceTestOrderAsync(
    symbol: "BTC-USDT",
    side: OrderSide.Buy,
    type: NewOrderType.Limit,
    quantity: 0.001m,
    price: 50000m,
    timeInForce: TimeInForce.GoodTillCanceled);
```

Futures market data, leverage, and position reads:

```csharp
var contract = await client.FuturesApi.ExchangeData.GetContractAsync("ETHUSDTM");
var ticker = await client.FuturesApi.ExchangeData.GetTickerAsync("ETHUSDTM");
var positions = await client.FuturesApi.Account.GetPositionAsync("ETHUSDTM");
```

Use `PlaceTestOrderAsync` for spot and futures syntax examples. Use live futures `PlaceOrderAsync` only when the user explicitly wants to create or close a futures position.

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new KucoinSocketClient();

var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "BTC-USDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Private spot, futures, and Unified streams require `KucoinCredentials`.

Use `UnsubscribeAsync` or `UnsubscribeAllAsync` on shutdown. Do not leave example subscriptions running.

## SharedApis

Use SharedApis only when portability matters:

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new KucoinRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")));
```

Kucoin.Net exposes SharedApis from `SpotApi.SharedClient` and `FuturesApi.SharedClient` on REST and socket clients. `UnifiedApi` is native KuCoin-specific and does not expose a SharedApis root.

Do not mix Kucoin-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddKucoin(options =>
{
    options.ApiCredentials = new KucoinCredentials(
        "API_KEY",
        "API_SECRET",
        "API_PASSPHRASE");
});
```

Inject `IKucoinRestClient` and `IKucoinSocketClient`, or the registered Kucoin REST/socket client interfaces used by the target project. `AddKucoin` also registers supported SharedApis interfaces from `SpotApi.SharedClient` and `FuturesApi.SharedClient`.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, margin, Earn, lending, transfer, withdrawal, private websocket, or credential code.
- Prefer read-only examples for account inspection.
- Prefer `PlaceTestOrderAsync` for spot order examples that only demonstrate syntax.
- Futures orders, futures margin mode changes, and futures leverage changes affect live account state; make that explicit.
- Include cleanup for example live orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups, client roots, SharedApis, result types, and symbol formats.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, futures, Unified, margin, Earn, transfer, withdrawal, websocket, tracker, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../Kucoin.Net/Examples/ai-friendly/` for complete compilable examples.
