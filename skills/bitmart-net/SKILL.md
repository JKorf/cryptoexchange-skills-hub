---
name: bitmart-net
description: Build C#/.NET BitMart integrations with BitMart.Net, including SpotApi, UsdFuturesApi, REST clients, websocket subscriptions, spot and USD futures account reads, order placement/cancellation, spot margin, sub-accounts, BitMartCredentials with memo/passphrase, BitMart spot and futures symbol formats, enums, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, ExchangeCallResult helper handling, and SharedApis access. Use when the user asks for BitMart market data, BitMart account or trading code, BitMart USD futures, BitMart spot margin, BitMart websocket updates, BitMart error handling, or converting raw BitMart API usage to idiomatic BitMart.Net.
---

# BitMart.Net

## Overview

Use `BitMart.Net` for BitMart-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for BitMart endpoint groups, BitMart spot/futures symbol formats, USD futures, spot margin borrow/repay, sub-account transfers, BitMart order enums, or BitMart credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package BitMart.Net
```

Use these namespaces in examples:

```csharp
using BitMart.Net;
using BitMart.Net.Clients;
using BitMart.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new BitMartRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new BitMartSocketClient();
```

Primary API surfaces:

- `rest.SpotApi.ExchangeData`: spot assets, symbols, tickers, klines, trades, order books, server status/time
- `rest.SpotApi.Account`: funding/spot balances, deposits, withdrawals, isolated margin accounts, fees
- `rest.SpotApi.Margin`: isolated margin borrow, repay, borrow/repay history, borrow info
- `rest.SpotApi.SubAccount`: spot sub-account transfers, balances, transfer history, sub-account list
- `rest.SpotApi.Trading`: spot orders, margin orders, cancellation, order lookup, open/closed orders, user trades
- `rest.UsdFuturesApi.ExchangeData`: USD futures contracts, funding, order book, open interest, klines, trades, leverage brackets
- `rest.UsdFuturesApi.Account`: USD futures balances, transfers, leverage, fee rate, transaction history, position mode
- `rest.UsdFuturesApi.SubAccount`: USD futures sub-account transfers, balances, transfer history
- `rest.UsdFuturesApi.Trading`: USD futures orders, trigger orders, TP/SL, positions, user trades
- `socket.SpotApi`: spot ticker, kline, order book, trade, book ticker, order, and balance streams
- `socket.UsdFuturesApi`: USD futures ticker, kline, order book, trade, balance, order, and position streams

Do not use Binance/Bitget-style roots such as `SpotApiV3`, `FuturesApiV2`, `UsdFuturesApiV3`, `CoinFuturesApi`, or `PerpetualFuturesApi`.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new BitMartRestClient();
```

Private account, trading, margin, futures, sub-account, transfer, withdrawal, and user stream calls need `BitMartCredentials` with API key, secret, and memo/passphrase:

```csharp
var client = new BitMartRestClient(options =>
{
    options.ApiCredentials = new BitMartCredentials("API_KEY", "API_SECRET", "MEMO_OR_PASS");
});
```

Use placeholders or environment variables in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared non-I/O symbol/cache helpers can return `ExchangeCallResult<T>`. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTC_USDT");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Enums

- Spot symbols use underscores, for example `BTC_USDT` and `ETH_USDT`.
- USD futures symbols are compact, for example `BTCUSDT` and `ETHUSDT`.
- Do not use `BTC-USDT`, `BTC/USDT`, `BTC_USDT` for USD futures, or `BTCUSDT` for spot.
- Spot orders use `OrderSide` and `OrderType`.
- USD futures orders use `FuturesSide`, `FuturesOrderType`, `MarginType`, `OrderMode`, `PositionMode`, and trigger/TP-SL enums.

Let the library generate client order IDs unless the user needs external correlation.

## REST Patterns

Public spot ticker:

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTC_USDT");
```

Authenticated spot balances:

```csharp
var balances = await client.SpotApi.Account.GetSpotBalancesAsync();
```

Spot limit order:

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTC_USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    price: 1m);
```

USD futures funding and order:

```csharp
const string symbol = "BTCUSDT";

var funding = await client.UsdFuturesApi.ExchangeData.GetCurrentFundingRateAsync(symbol);

var order = await client.UsdFuturesApi.Trading.PlaceOrderAsync(
    symbol: symbol,
    side: FuturesSide.BuyOpenLong,
    type: FuturesOrderType.Limit,
    quantity: 1,
    price: 1m,
    leverage: 5m,
    marginType: MarginType.CrossMargin,
    orderMode: OrderMode.GoodTilCancel);
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new BitMartSocketClient();

var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "BTC_USDT",
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

ISpotTickerRestClient tickers = new BitMartRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")));
```

BitMart exposes shared clients on both `SpotApi` and `UsdFuturesApi`. Call `SharedClient.Discover()` before routing optional shared features. Do not mix BitMart-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddBitMart(options =>
{
    options.Rest.ApiCredentials = new BitMartCredentials("API_KEY", "API_SECRET", "MEMO_OR_PASS");
    options.Socket.ApiCredentials = new BitMartCredentials("API_KEY", "API_SECRET", "MEMO_OR_PASS");
});
```

Inject `IBitMartRestClient` and `IBitMartSocketClient`, or the registered shared interfaces used by the target project. Match the repository's existing DI pattern if one is present.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, futures, margin, transfer, withdrawal, sub-account, or credential code.
- Prefer read-only examples for account inspection.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- USD futures and margin examples can change leverage, borrow funds, or create real exposure; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, futures, margin, withdrawal, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../BitMart.Net/Examples/ai-friendly/` for complete compilable examples.
