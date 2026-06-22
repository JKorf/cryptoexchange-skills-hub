---
name: hyperliquid-net
description: Build C#/.NET HyperLiquid integrations with HyperLiquid.Net, including SpotApi, FuturesApi, REST, websocket subscriptions and queries, market data, balances, account ledger, staking, vaults, transfers, withdrawals, builder fees, HIP-3 DEX parameters, spot and perpetual orders, TWAP, triggers, TP/SL, leverage, positions, HyperLiquidCredentials, native symbol formats, dependency injection, HttpResult, WebSocketResult, QueryResult, trackers, and SharedApis. Use for HyperLiquid market data, account, trading, perpetual futures, websocket, error handling, or idiomatic HyperLiquid.Net code.
---

# HyperLiquid.Net

## Overview

Use `HyperLiquid.Net` for HyperLiquid-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for HyperLiquid spot endpoints, perpetual futures endpoints, user streams, builder fees, HIP-3 DEX, vaults, staking, leverage, positions, or HyperLiquid credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the exchange package:

```bash
dotnet add package HyperLiquid.Net
```

Use these namespaces in examples:

```csharp
using HyperLiquid.Net;
using HyperLiquid.Net.Clients;
using HyperLiquid.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for exchange-agnostic SharedApis code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new HyperLiquidRestClient();
```

Create socket clients for websocket subscriptions and websocket request/query methods:

```csharp
var socket = new HyperLiquidSocketClient();
```

Primary REST API surfaces:

- `rest.SpotApi.ExchangeData`, `rest.SpotApi.Account`, `rest.SpotApi.Trading`, `rest.SpotApi.SharedClient`
- `rest.FuturesApi.ExchangeData`, `rest.FuturesApi.Account`, `rest.FuturesApi.Trading`, `rest.FuturesApi.SharedClient`

Primary socket API surfaces:

- `socket.SpotApi.ExchangeData`, `socket.SpotApi.Account`, `socket.SpotApi.Trading`, `socket.SpotApi.SharedClient`
- `socket.FuturesApi.ExchangeData`, `socket.FuturesApi.Account`, `socket.FuturesApi.Trading`, `socket.FuturesApi.SharedClient`

HyperLiquid.Net does not expose Binance-style roots such as `UsdFuturesApi`, `CoinFuturesApi`, `UnifiedApi`, or `V5Api`. Use `FuturesApi` for HyperLiquid perpetual futures.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new HyperLiquidRestClient();
```

Private account, trading, transfer, withdrawal, staking, vault, and authenticated socket calls need `HyperLiquidCredentials` with public address and ECDSA private key:

```csharp
var client = new HyperLiquidRestClient(options =>
{
    options.ApiCredentials = new HyperLiquidCredentials("PUBLIC_ADDRESS", "PRIVATE_KEY");
});
```

Use placeholders or environment variables in generated code. Do not hardcode real keys or private keys.

## Builder Fee

HyperLiquid.Net enables builder code by default. `BuilderFeePercentage` defaults to `0.01m` (1 bps / 0.01%). Trading code can check approval with `Account.GetApprovedBuilderFeeAsync()` or disable the builder fee intentionally:

```csharp
var client = new HyperLiquidRestClient(options =>
{
    options.ApiCredentials = new HyperLiquidCredentials("PUBLIC_ADDRESS", "PRIVATE_KEY");
    options.BuilderFeePercentage = 0;
});
```

Mention this when generating trading examples.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket request/query methods return `QueryResult<T>` or `QueryResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Some SharedApis symbol helper methods return `ExchangeCallResult<T>`. Always check `Success` before reading `Data`.

```csharp
var prices = await client.FuturesApi.ExchangeData.GetPricesAsync();
if (!prices.Success)
{
    Console.WriteLine(prices.Error);
    return;
}

Console.WriteLine(prices.Data["ETH"]);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Orders

- Spot symbols use `Base/Quote`, for example `HYPE/USDC`.
- Futures/perpetual symbols use the base asset only, for example `ETH`.
- `HyperLiquidExchange.FormatSymbol("HYPE", "USDC", TradingMode.Spot)` returns `HYPE/USDC`.
- `HyperLiquidExchange.FormatSymbol("ETH", "USDC", TradingMode.PerpetualLinear)` returns `ETH`.
- Market orders still require `price`; HyperLiquid uses it for max slippage calculation.
- Client order ids are optional 128-bit hex strings, for example `0x1234567890abcdef1234567890abcdef`.

Let the library generate client order IDs unless the user needs external correlation.

## REST Patterns

Public spot prices:

```csharp
var prices = await client.SpotApi.ExchangeData.GetPricesAsync();
Console.WriteLine(prices.Data["HYPE/USDC"]);
```

Authenticated spot balances:

```csharp
var balances = await client.SpotApi.Account.GetBalancesAsync();
```

Spot limit order:

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "HYPE/USDC",
    side: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 1m,
    price: 10m,
    timeInForce: TimeInForce.GoodTillCanceled);
```

Futures market order:

```csharp
var prices = await client.FuturesApi.ExchangeData.GetPricesAsync();
var order = await client.FuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETH",
    side: OrderSide.Buy,
    orderType: OrderType.Market,
    quantity: 0.01m,
    price: prices.Data["ETH"]);
```

Futures leverage:

```csharp
await client.FuturesApi.Trading.SetLeverageAsync(
    symbol: "ETH",
    leverage: 5,
    marginType: MarginType.Cross);
```

Use `reduceOnly: true` when closing positions with orders.

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new HyperLiquidSocketClient();

var sub = await socket.FuturesApi.ExchangeData.SubscribeToSymbolUpdatesAsync(
    "ETH",
    update => Console.WriteLine(update.Data.MidPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Authenticated streams use the address from credentials when `address` is `null`:

```csharp
var sub = await socket.FuturesApi.Trading.SubscribeToOrderUpdatesAsync(
    address: null,
    onMessage: update => Console.WriteLine(update.Data.Length));
```

Native socket request/query methods mirror many REST methods and return `QueryResult<T>`, not `HttpResult<T>`.

Use `UnsubscribeAsync` or `UnsubscribeAllAsync` on shutdown. Do not leave example subscriptions running.

## SharedApis

Use SharedApis only when portability matters:

```csharp
using CryptoExchange.Net.SharedApis;

IFuturesTickerRestClient tickers = new HyperLiquidRestClient().FuturesApi.SharedClient;
var result = await tickers.GetFuturesTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.PerpetualLinear, "ETH", "USDC")));
```

HyperLiquid.Net exposes SharedApis from both `SpotApi.SharedClient` and `FuturesApi.SharedClient` on REST and socket clients.

Do not mix HyperLiquid-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddHyperLiquid(options =>
{
    options.ApiCredentials = new HyperLiquidCredentials("PUBLIC_ADDRESS", "PRIVATE_KEY");
});
```

Inject `IHyperLiquidRestClient` and `IHyperLiquidSocketClient`, or the registered HyperLiquid REST/socket client interfaces used by the target project. `AddHyperLiquid` also registers supported SharedApis interfaces from `SpotApi.SharedClient` and `FuturesApi.SharedClient`.

## Safety Rules

- Read `references/safety.md` before generating trading, transfer, withdrawal, staking, vault, leverage, margin, HIP-3 DEX, builder-fee, or credential code.
- Prefer read-only examples for account inspection.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Market orders still require a recent price.
- Futures examples can change leverage and create real exposure; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups, client roots, SharedApis, result types, and symbol formats.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, futures, transfer, withdrawal, staking, vault, websocket, tracker, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../HyperLiquid.Net/Examples/ai-friendly/` for complete compilable examples.
