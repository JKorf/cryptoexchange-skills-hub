---
name: polymarket-net
description: Build C#/.NET Polymarket integrations with Polymarket.Net, including Polymarket.Net package setup, ClobApi, GammaApi, DataApi, REST clients, websocket subscriptions, CLOB token market data, Gamma market and event discovery, account allowances, L1 and L2 PolymarketCredentials authentication, order placement/cancellation, user streams, local order books, dependency injection, user client providers, HttpResult REST handling, WebSocketResult subscription handling, and Polymarket token id safety. Use when the user asks for Polymarket market data, Polymarket CLOB trading, Polymarket events/search, Polymarket websocket updates, Polymarket auth, Polymarket order books, Polymarket error handling, or converting raw Polymarket API usage to idiomatic Polymarket.Net.
---

# Polymarket.Net

## Overview

Use `Polymarket.Net` for Polymarket-specific C# application code. Prefer this skill over Binance-shaped exchange skills when the user asks for Polymarket CLOB market data, Gamma events/markets/search, order placement, wallet/API-key authentication, user streams, sports streams, local order books, or user-specific clients.

Polymarket uses CLOB token ids and market/condition ids, not trading symbols such as `BTCUSDT`.

## Setup

Install the package:

```bash
dotnet add package Polymarket.Net
```

Use these namespaces in examples:

```csharp
using CryptoExchange.Net.Authentication;
using CryptoExchange.Net.Objects;
using Polymarket.Net;
using Polymarket.Net.Clients;
using Polymarket.Net.Enums;
using Polymarket.Net.Objects.Models;
```

## Client Roots

Create REST clients for request/response endpoints:

```csharp
var rest = new PolymarketRestClient();
```

Create socket clients for websocket subscriptions:

```csharp
var socket = new PolymarketSocketClient();
```

Primary REST API surfaces:

- `rest.ClobApi.ExchangeData`: CLOB markets, token prices, order books, spreads, tick size, fee rates, server time
- `rest.ClobApi.Account`: L2 API credentials, balance/allowance, notifications, builder trades
- `rest.ClobApi.Trading`: orders, cancels, open orders, trades, order heartbeat
- `rest.GammaApi`: human-readable events, markets, tags, sports metadata, search
- `rest.DataApi`: user position data

Primary socket API surface:

- `socket.ClobApi`: token updates, platform updates, user order/trade updates, sports updates

Polymarket.Net does not expose Binance-style roots such as `SpotApi`, `UsdFuturesApi`, `CoinFuturesApi`, or `UnifiedApi`. Read `references/api-surfaces.md` when choosing an endpoint group.

## Credentials

Public Gamma, CLOB market data, and public websocket streams do not need credentials:

```csharp
var client = new PolymarketRestClient();
```

Private order, allowance, credential, and user websocket endpoints need L1 wallet credentials and usually L2 API credentials:

```csharp
var credentials = new PolymarketCredentials(
    new PolymarketL1Credential(
        SignType.Poly1271,
        "PRIVATE_KEY",
        "POLYMARKET_FUNDING_OR_DEPOSIT_ADDRESS"),
    new HMACPassCredential(
        "L2_API_KEY",
        "L2_API_SECRET",
        "L2_API_PASSPHRASE"));
```

If only L1 credentials are available, derive or create L2 credentials with `rest.ClobApi.Account.GetOrCreateApiCredentialsAsync()` and then call `rest.UpdateL2Credentials(...)` or `socket.UpdateL2Credentials(...)`.

Use placeholders, environment variables, or injected options in generated code. Never hardcode real private keys or API credentials.

## Result Handling

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Always check `Success` before reading `Data`.

```csharp
var markets = await client.GammaApi.GetMarketsAsync(closed: false, limit: 5);
if (!markets.Success)
{
    Console.WriteLine(markets.Error);
    return;
}

Console.WriteLine(markets.Data[0].Question);
```

Order placement has two success layers: `HttpResult.Success` means the call succeeded, while `PolymarketOrderResult.Success` means Polymarket accepted the order. Check both.

## Tokens, Markets, And Prices

- CLOB order books and orders use `tokenId` values.
- Gamma markets/events expose human-readable metadata and can include CLOB token ids.
- Market/condition ids are not token ids.
- Prices are probabilities between `0` and `1`, for example `0.42` for 42 cents.
- `quantity` is shares unless `QuantityType.Value` is used for supported market buy flows.
- Use `GetTickSizeAsync`, `GetFeeRateBpsAsync`, and `GetBalanceAllowanceAsync` before production trading code.

## REST Patterns

Human-readable market discovery:

```csharp
var markets = await client.GammaApi.GetMarketsAsync(closed: false, limit: 5);
```

CLOB token order book:

```csharp
var book = await client.ClobApi.ExchangeData.GetOrderBookAsync("TOKEN_ID");
```

Allowance check:

```csharp
var allowance = await client.ClobApi.Account.GetBalanceAllowanceAsync(AssetType.Collateral);
```

Live limit order:

```csharp
var order = await client.ClobApi.Trading.PlaceOrderAsync(
    tokenId: "TOKEN_ID",
    side: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 10m,
    price: 0.42m,
    timeInForce: TimeInForce.GoodTillCanceled);
```

Use `references/usage.md` for complete snippets with error handling and cleanup.

## Websocket Patterns

Keep socket handlers fast. Offload heavier work to a queue or channel.

```csharp
var socket = new PolymarketSocketClient();

var sub = await socket.ClobApi.SubscribeToTokenUpdatesAsync(
    new[] { "TOKEN_ID" },
    onBookUpdate: update => Console.WriteLine(update.Data));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Authenticated user streams require L1 and L2 credentials. Use `UnsubscribeAsync` or `UnsubscribeAllAsync` on shutdown.

## Dependency Injection

When working in ASP.NET Core or worker services, prefer DI and reuse clients:

```csharp
services.AddPolymarket(options =>
{
    options.ApiCredentials = credentials;
});
```

Inject `IPolymarketRestClient`, `IPolymarketSocketClient`, `IPolymarketOrderBookFactory`, or `IPolymarketUserClientProvider` as needed.

## Safety Rules

- Read `references/safety.md` before generating trading, credential, allowance, private websocket, order heartbeat, or user-client code.
- Prefer read-only Gamma and CLOB market data examples unless the user asks for trading.
- Treat `PlaceOrderAsync`, batch order methods, cancel-all methods, allowance updates, API key creation/deletion, and order heartbeats as live account actions.
- Check both transport success and order acceptance for order placement.
- Include cancellation cleanup for example live orders or subscriptions when appropriate.

## References

- Read `references/api-surfaces.md` for endpoint groups, client roots, result types, socket streams, credentials, and local order books.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for credential, trading, allowance, websocket, and token-id guardrails.
- In a maintainer checkout with sibling repositories, read `../Polymarket.Net/Examples/ai-friendly/` for complete compilable examples.
