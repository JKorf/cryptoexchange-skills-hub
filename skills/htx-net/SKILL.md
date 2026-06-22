---
name: htx-net
description: Build C#/.NET HTX integrations with HTX.Net, including JKorf.HTX.Net package setup, SpotApi, UsdtFuturesApi, UsdtFuturesV5Api, REST clients, websocket subscriptions, socket query/order methods, spot market data, spot account ids, margin, deposits, withdrawals, transfers, USDT futures, cross and isolated margin futures, V5 futures orders, leverage, positions, trigger orders, TP/SL, HTXCredentials with HMAC or Ed25519, HTX spot symbols without separators, HTX futures contract codes with hyphens, dependency injection, HttpResult REST handling, WebSocketResult subscription handling, QueryResult socket request handling, ExchangeCallResult shared helper handling, trackers, and SharedApis access. Use when the user asks for HTX spot market data, HTX account or trading code, HTX futures, HTX websocket updates, HTX error handling, or converting raw HTX API usage to idiomatic HTX.Net.
---

# HTX.Net

## Overview

Use `HTX.Net` for HTX-specific C# application code. Prefer this skill over the generic `cryptoexchange-net` skill when the user asks for HTX spot endpoints, USDT futures endpoints, cross/isolated margin, V5 futures, user streams, trackers, leverage, positions, or HTX credentials.

Use `cryptoexchange-net` instead when the user wants the same code to run across multiple exchanges through `CryptoExchange.Net.SharedApis`.

## Setup

Install the package:

```bash
dotnet add package JKorf.HTX.Net
```

Use these namespaces in examples:

```csharp
using HTX.Net;
using HTX.Net.Clients;
using HTX.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for exchange-agnostic SharedApis code.

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new HTXRestClient();
```

Create socket clients for websocket subscriptions and socket query methods:

```csharp
var socket = new HTXSocketClient();
```

Primary REST API surfaces:

- `rest.SpotApi.ExchangeData`, `rest.SpotApi.Account`, `rest.SpotApi.Margin`, `rest.SpotApi.SubAccount`, `rest.SpotApi.Trading`, `rest.SpotApi.SharedClient`
- `rest.UsdtFuturesApi.ExchangeData`, `rest.UsdtFuturesApi.Account`, `rest.UsdtFuturesApi.SubAccount`, `rest.UsdtFuturesApi.Trading`, `rest.UsdtFuturesApi.SharedClient`
- `rest.UsdtFuturesV5Api.ExchangeData`, `rest.UsdtFuturesV5Api.Account`, `rest.UsdtFuturesV5Api.Trading`

Primary socket API surfaces:

- `socket.SpotApi`: public spot streams, private spot streams, spot websocket queries, and spot socket order request methods
- `socket.UsdtFuturesApi`: public USDT futures streams, private futures streams, and SharedApis support
- `socket.UsdtFuturesV5Api`: V5 private futures account, order, position, and trade streams

HTX.Net does not expose Binance-style `UsdFuturesApi`, `CoinFuturesApi`, or `PortfolioMarginApi` roots. Use `UsdtFuturesApi` or `UsdtFuturesV5Api`.

Read `references/api-surfaces.md` when choosing a client root or endpoint group.

## Credentials

Public market data does not need credentials:

```csharp
var client = new HTXRestClient();
```

Private account, trading, transfer, withdrawal, and authenticated socket calls need `HTXCredentials`.

```csharp
var client = new HTXRestClient(options =>
{
    options.ApiCredentials = new HTXCredentials("API_KEY", "API_SECRET");
});
```

`HTXCredentials` supports HMAC by default and also has Ed25519 support through `new HTXCredentials().WithEd25519(key, privateKey)`.

Use placeholders or environment variables in generated code. Do not hardcode real credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Spot socket query/order methods return `QueryResult<T>` or `QueryResult`. Some SharedApis symbol helper methods return `ExchangeCallResult<T>`. Always check `Success` before reading `Data`.

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("ETHUSDT");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.ClosePrice);
```

Use `result.Error.Code`, `result.Error.Message`, `result.Error.ErrorType`, and `result.Error.IsTransient` when writing diagnostics or retry logic. Retry only transient errors.

## Symbols And Account Ids

- Native spot symbols use no separator, for example `ETHUSDT`.
- Native USDT futures contract codes use hyphens, for example `ETH-USDT`.
- `HTXExchange.FormatSymbol("ETH", "USDT", TradingMode.Spot)` returns lowercase `ethusdt`; native examples use uppercase `ETHUSDT`. Do not generate Binance-style assumptions beyond “no separator”.
- `HTXExchange.FormatSymbol("ETH", "USDT", TradingMode.PerpetualLinear)` returns `ETH-USDT`.
- HTX spot trading and balance methods require an account id. Call `SpotApi.Account.GetAccountsAsync()` first and use the returned `Id`.
- USDT futures has explicit cross-margin and isolated-margin methods. Choose the method that matches the requested margin mode.

Let the library generate client order IDs unless the user needs external correlation.

## REST Patterns

Public spot ticker:

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("ETHUSDT");
```

Authenticated spot balances:

```csharp
var accounts = await client.SpotApi.Account.GetAccountsAsync();
if (!accounts.Success)
    return;

var spotAccount = accounts.Data.FirstOrDefault();
if (spotAccount == null)
    return;

var balances = await client.SpotApi.Account.GetBalancesAsync(spotAccount.Id);
```

Spot limit order:

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    accountId: spotAccount.Id,
    symbol: "ETHUSDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.01m,
    price: 2000m);
```

USDT futures cross-margin leverage and market order:

```csharp
const string contractCode = "ETH-USDT";

await client.UsdtFuturesApi.Trading.SetCrossMarginLeverageAsync(
    leverageRate: 5,
    contractCode: contractCode);

var order = await client.UsdtFuturesApi.Trading.PlaceCrossMarginOrderAsync(
    quantity: 1,
    side: OrderSide.Buy,
    leverageRate: 5,
    orderPriceType: OrderPriceType.Market,
    contractCode: contractCode,
    offset: Offset.Open);
```

Use `SetIsolatedMarginLeverageAsync` and `PlaceIsolatedMarginOrderAsync` for isolated margin. Futures `quantity` is contract quantity.

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new HTXSocketClient();

var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "ETHUSDT",
    update => Console.WriteLine(update.Data.ClosePrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

USDT futures public subscriptions use contract codes:

```csharp
var sub = await socket.UsdtFuturesApi.SubscribeToTickerUpdatesAsync(
    "ETH-USDT",
    update => Console.WriteLine(update.Data.ClosePrice));
```

Native spot socket query/order methods, such as `socket.SpotApi.GetKlinesAsync(...)` and `socket.SpotApi.PlaceOrderAsync(...)`, return `QueryResult<T>`, not `HttpResult<T>`.

Use `UnsubscribeAsync` or `UnsubscribeAllAsync` on shutdown. Do not leave example subscriptions running.

## SharedApis

Use SharedApis only when portability matters:

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new HTXRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "ETH", "USDT")));
```

HTX.Net exposes SharedApis from `SpotApi.SharedClient` and `UsdtFuturesApi.SharedClient` on REST and socket clients. `UsdtFuturesV5Api` is a native HTX surface and is not the SharedApis futures root.

Do not mix HTX-native request/model types with `SharedApis` request/model types.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddHTX(options =>
{
    options.ApiCredentials = new HTXCredentials("API_KEY", "API_SECRET");
});
```

Inject `IHTXRestClient` and `IHTXSocketClient`, or the registered HTX REST/socket client interfaces used by the target project. `AddHTX` also registers supported SharedApis interfaces from `SpotApi.SharedClient` and `UsdtFuturesApi.SharedClient`.

## Safety Rules

- Read `references/safety.md` before generating trading, leverage, margin, futures, transfer, withdrawal, API-key, or credential code.
- Prefer read-only examples for account inspection.
- Spot order examples must fetch and validate an account id first.
- Prefer safe limit-order examples over market orders unless the user explicitly asks for immediate execution.
- Futures examples can change leverage and create real exposure; make that explicit.
- Include cleanup for example orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups, client roots, SharedApis, result types, and symbol formats.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for account, trading, futures, transfer, withdrawal, websocket, tracker, and credential guardrails.
- In a maintainer checkout with sibling repositories, read `../HTX.Net/Examples/ai-friendly/` for complete compilable examples.
