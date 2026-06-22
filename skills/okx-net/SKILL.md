---
name: okx-net
description: Build C#/.NET OKX integrations with OKX.Net, including JK.OKX.Net package setup, UnifiedApi, REST clients, websocket subscriptions, websocket order request methods, spot market data, derivatives market data, account balances, deposits, withdrawals, transfers, subaccounts, copy trading, spot order placement/check/cancellation, swap/futures/options order placement, algo orders, TP/SL orders, leverage, position mode, positions, OKXCredentials key/secret/passphrase authentication, OKX spot symbols, OKX swap/futures symbols, demo and Europe environment notes, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, QueryResult socket request handling, ExchangeCallResult shared helper handling, trackers, and SharedApis access. Use when the user asks for OKX spot market data, OKX account or trading code, OKX swaps/futures/options, OKX websocket updates, OKX websocket order requests, OKX copy trading, OKX error handling, or converting raw OKX API usage to idiomatic OKX.Net.
---

# OKX.Net

## Overview

Use `OKX.Net` for OKX-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for OKX v5 endpoints, unified account workflows, spot, swaps, futures, options, private streams, websocket order requests, subaccounts, copy trading, leverage, position mode, or OKX credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the package:

```bash
dotnet add package JK.OKX.Net
```

Use these namespaces in examples:

```csharp
using CryptoExchange.Net.Objects;
using OKX.Net;
using OKX.Net.Clients;
using OKX.Net.Enums;
```

Add `using CryptoExchange.Net.SharedApis;` only for exchange-agnostic SharedApis code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new OKXRestClient();
```

Create socket clients for websocket subscriptions and socket order requests:

```csharp
var socket = new OKXSocketClient();
```

Primary REST API surfaces:

- `rest.UnifiedApi.ExchangeData`
- `rest.UnifiedApi.Account`
- `rest.UnifiedApi.Trading`
- `rest.UnifiedApi.SubAccounts`
- `rest.UnifiedApi.CopyTrading`
- `rest.UnifiedApi.SharedClient`

Primary socket API surfaces:

- `socket.UnifiedApi.ExchangeData`: public market streams
- `socket.UnifiedApi.Account`: private account, balance/position, deposit, withdrawal, and greeks streams
- `socket.UnifiedApi.Trading`: private order/algo/user-trade/position streams plus websocket order request methods
- `socket.UnifiedApi.SharedClient`

OKX.Net does not expose Binance-style roots such as `SpotApi`, `UsdFuturesApi`, `CoinFuturesApi`, or `V5Api`. Use `UnifiedApi` for OKX spot, margin, swaps, futures, and options.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new OKXRestClient();
```

Authenticated OKX endpoints require key, secret, and API passphrase:

```csharp
var client = new OKXRestClient(options =>
{
    options.ApiCredentials = new OKXCredentials(
        "API_KEY",
        "API_SECRET",
        "PASSPHRASE");
});
```

Use placeholders, environment variables, or injected options in generated code. Do not omit the passphrase for account, trading, private websocket, transfer, withdrawal, subaccount, copy trading, or websocket order request examples.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Websocket order request methods return `QueryResult<T>` or `QueryResult<CallResult<T>[]>`. Shared non-I/O helper methods can return `ExchangeCallResult<T>`. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.UnifiedApi.ExchangeData.GetTickerAsync("ETH-USDT");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols

- Spot symbols use dash-separated names such as `ETH-USDT` and `BTC-USDT`.
- Perpetual swap symbols use names such as `ETH-USDT-SWAP`.
- Delivery futures symbols use date-suffixed names such as `ETH-USDT-260327`.
- Options use OKX option instrument ids and often require `InstrumentType.Option` plus underlying or instrument family parameters.
- `OKXExchange.FormatSymbol("ETH", "USDT", TradingMode.Spot)` returns `ETH-USDT`.
- `OKXExchange.FormatSymbol("ETH", "USDT", TradingMode.PerpetualLinear)` returns `ETH-USDT-SWAP`.
- Fetch instruments with `UnifiedApi.ExchangeData.GetSymbolsAsync(InstrumentType, ...)`.
- Socket trading request methods use `OKXInstrument.SymbolCode`, not the display symbol string. Fetch it with `UnifiedApi.ExchangeData.GetSymbolsAsync(...)` before calling socket order request methods.

## REST Patterns

Public spot ticker:

```csharp
var ticker = await client.UnifiedApi.ExchangeData.GetTickerAsync("ETH-USDT");
```

Authenticated account balance:

```csharp
var balance = await client.UnifiedApi.Account.GetAccountBalanceAsync("USDT");
```

Spot order syntax validation:

```csharp
var check = await client.UnifiedApi.Trading.CheckOrderAsync(
    symbol: "ETH-USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.01m,
    price: 2000m,
    tradeMode: TradeMode.Cash);
```

Live spot order:

```csharp
var order = await client.UnifiedApi.Trading.PlaceOrderAsync(
    symbol: "ETH-USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.01m,
    price: 2000m,
    tradeMode: TradeMode.Cash);
```

Swap market data, leverage, and position reads:

```csharp
const string swap = "ETH-USDT-SWAP";

var symbols = await client.UnifiedApi.ExchangeData.GetSymbolsAsync(
    InstrumentType.Swap,
    symbol: swap);

var leverage = await client.UnifiedApi.Account.SetLeverageAsync(
    leverage: 5,
    marginMode: MarginMode.Cross,
    symbol: swap,
    positionSide: PositionSide.Long);

var positions = await client.UnifiedApi.Account.GetPositionsAsync(
    InstrumentType.Swap,
    symbol: swap);
```

Use `CheckOrderAsync` for syntax/risk checks when possible. `PlaceOrderAsync`, `PlaceAlgoOrderAsync`, `ClosePositionAsync`, and leverage/account mode methods affect live account state.

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new OKXSocketClient();

var sub = await socket.UnifiedApi.ExchangeData.SubscribeToTickerUpdatesAsync(
    "ETH-USDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Private streams require `OKXCredentials`:

```csharp
var socket = new OKXSocketClient(options =>
{
    options.ApiCredentials = new OKXCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
});

var sub = await socket.UnifiedApi.Trading.SubscribeToOrderUpdatesAsync(
    InstrumentType.Spot,
    symbol: "ETH-USDT",
    instrumentFamily: null,
    onData: update => Console.WriteLine(update.Data.OrderState));
```

Socket order request methods such as `socket.UnifiedApi.Trading.PlaceOrderAsync(symbolCode, ...)`, `CancelOrderAsync(symbolCode, ...)`, and `AmendOrderAsync(symbolCode, ...)` return `QueryResult<T>`. They use numeric `OKXInstrument.SymbolCode` values fetched from REST instrument metadata. Treat them like live trading endpoints.

Use `UnsubscribeAsync` or `UnsubscribeAllAsync` on shutdown. Do not leave example subscriptions running.

## SharedApis

Use SharedApis only when portability matters:

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new OKXRestClient().UnifiedApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "ETH", "USDT")));
```

OKX.Net exposes SharedApis from `UnifiedApi.SharedClient` on REST and socket clients.

Do not mix OKX-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddOKX(options =>
{
    options.ApiCredentials = new OKXCredentials(
        "API_KEY",
        "API_SECRET",
        "PASSPHRASE");
});
```

Inject `IOKXRestClient` and `IOKXSocketClient`, or the registered OKX REST/socket client interfaces used by the target project. `AddOKX` also registers supported SharedApis interfaces from `UnifiedApi.SharedClient`.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, account mode, copy trading, transfer, withdrawal, subaccount, private websocket, socket order request, or credential code.
- Prefer read-only examples for account inspection.
- Prefer `CheckOrderAsync` for order syntax/risk examples when possible.
- Leverage changes, position mode/account mode changes, live orders, algo orders, close position, transfers, withdrawals, and websocket order requests affect live account state; make that explicit.
- Include cleanup for example live orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups, client roots, SharedApis, result types, socket request methods, and symbol formats.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, derivatives, copy trading, transfer, withdrawal, websocket, tracker, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../OKX.Net/Examples/ai-friendly/` for complete compilable examples.
