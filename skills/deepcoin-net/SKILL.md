---
name: deepcoin-net
description: Build C#/.NET DeepCoin integrations with DeepCoin.Net, including ExchangeApi REST clients, ExchangeApi websocket subscriptions, spot and swap market data, account balances, deposits, withdrawals, listen keys, leverage, positions, order placement/cancellation, TP/SL, DeepCoinCredentials with API passphrase, DeepCoin hyphenated spot symbols, DeepCoin swap symbols, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, ExchangeCallResult shared helper handling, and SharedApis access. Use when the user asks for DeepCoin spot market data, DeepCoin account or trading code, DeepCoin swaps/futures, DeepCoin websocket updates, DeepCoin error handling, or converting raw DeepCoin API usage to idiomatic DeepCoin.Net.
---

# DeepCoin.Net

## Overview

Use `DeepCoin.Net` for DeepCoin-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for DeepCoin spot endpoints, swap/futures endpoints, user data streams, leverage, positions, or DeepCoin credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package DeepCoin.Net
```

Use these namespaces in examples:

```csharp
using DeepCoin.Net;
using DeepCoin.Net.Clients;
using DeepCoin.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for exchange-agnostic SharedApis code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new DeepCoinRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new DeepCoinSocketClient();
```

Primary API surfaces:

- `rest.ExchangeApi.ExchangeData`: spot and swap market data, symbols, order books, funding rates
- `rest.ExchangeApi.Account`: balances, bills, leverage, deposits, withdrawals, listen keys
- `rest.ExchangeApi.Trading`: positions, spot/swap orders, cancellations, fills, TP/SL
- `socket.ExchangeApi`: public market streams and private user data streams

DeepCoin.Net does not expose Binance-style roots such as `SpotApi`, `UsdFuturesApi`, or `CoinFuturesApi`. Use `ExchangeApi` for both spot and swaps.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new DeepCoinRestClient();
```

Private account, trading, leverage, withdrawal, listen-key, and user stream calls need `DeepCoinCredentials` with API key, secret, and passphrase:

```csharp
var client = new DeepCoinRestClient(options =>
{
    options.ApiCredentials = new DeepCoinCredentials("API_KEY", "API_SECRET", "API_PASS");
});
```

Use placeholders or environment variables in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Always check `Success` before reading `Data`.

```csharp
var tickers = await client.ExchangeApi.ExchangeData.GetTickersAsync(SymbolType.Spot);
if (!tickers.Success)
{
    Console.WriteLine(tickers.Error);
    return;
}

var ticker = tickers.Data.FirstOrDefault(x => x.Symbol == "ETH-USDT");
Console.WriteLine(ticker?.LastPrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Enums

- Spot symbols use hyphenated pairs, for example `ETH-USDT`.
- Swap/futures symbols use `-SWAP`, for example `ETH-USDT-SWAP`.
- SharedApis can format symbols through `DeepCoinExchange.FormatSymbol`.
- REST market/account methods use `SymbolType.Spot` or `SymbolType.Swap`.
- Swap funding-rate methods use `ProductGroup`, for example `ProductGroup.USDTMargined`.
- Order placement uses `OrderSide`, `OrderType`, `TradeMode`, `QuantityType`, `PositionSide`, and `PositionType`.
- Spot orders should use `TradeMode.Spot`.
- Swap orders commonly use `TradeMode.Cross` or `TradeMode.Isolated` and `PositionSide.Long` or `PositionSide.Short`.

Let the library generate client order IDs unless the user needs external correlation.

## REST Patterns

Public spot ticker:

```csharp
var tickers = await client.ExchangeApi.ExchangeData.GetTickersAsync(SymbolType.Spot);
var ethusdt = tickers.Success
    ? tickers.Data.FirstOrDefault(x => x.Symbol == "ETH-USDT")
    : null;
```

Authenticated spot balances:

```csharp
var balances = await client.ExchangeApi.Account.GetBalancesAsync(SymbolType.Spot);
```

Spot limit order:

```csharp
var order = await client.ExchangeApi.Trading.PlaceOrderAsync(
    symbol: "ETH-USDT",
    side: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 0.01m,
    price: 2000m,
    tradeMode: TradeMode.Spot);
```

Swap leverage and market order:

```csharp
await client.ExchangeApi.Account.SetLeverageAsync(
    "ETH-USDT-SWAP",
    leverage: 5m,
    tradeMode: TradeMode.Cross,
    positionType: PositionType.Split);

var order = await client.ExchangeApi.Trading.PlaceOrderAsync(
    symbol: "ETH-USDT-SWAP",
    side: OrderSide.Buy,
    orderType: OrderType.Market,
    quantity: 0.1m,
    tradeMode: TradeMode.Cross,
    positionSide: PositionSide.Long);
```

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new DeepCoinSocketClient();

var sub = await socket.ExchangeApi.SubscribeToSymbolUpdatesAsync(
    "ETH-USDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Use `UnsubscribeAsync` or `UnsubscribeAllAsync` on shutdown. Do not leave example subscriptions running.

## SharedApis

Use SharedApis only when portability matters:

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new DeepCoinRestClient().ExchangeApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "ETH", "USDT")));
```

Do not mix DeepCoin-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddDeepCoin(options =>
{
    options.ApiCredentials = new DeepCoinCredentials("API_KEY", "API_SECRET", "API_PASS");
});
```

Inject `IDeepCoinRestClient` and `IDeepCoinSocketClient`, or the registered DeepCoin REST/socket client interfaces used by the target project. `AddDeepCoin` also registers supported SharedApis interfaces from `ExchangeApi.SharedClient`.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, swaps/futures, withdrawal, transfer, or credential code.
- Prefer read-only examples for account inspection.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Swap/futures examples can change leverage and create real exposure; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups and client roots.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, swaps/futures, withdrawal, listen-key, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../DeepCoin.Net/Examples/ai-friendly/` for complete compilable examples.
