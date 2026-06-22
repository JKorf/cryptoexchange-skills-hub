---
name: upbit-net
description: Build C#/.NET Upbit public market-data integrations with Upbit.Net, including JKorf.Upbit.Net package setup, SpotApi ExchangeData, REST clients, websocket subscriptions, symbols, tickers, trades, klines, order books, symbol configuration, regional South Korea/Singapore/Indonesia/Thailand environments, Upbit QUOTE-BASE symbols, dependency injection, local order books, trackers, HttpResult REST handling, WebSocketResult subscription handling, ExchangeCallResult shared helper handling, and SharedApis access. Use when the user asks for Upbit public quotation data, Upbit regional markets, Upbit websocket updates, Upbit order books, Upbit error handling, or converting raw public Upbit market-data usage to idiomatic Upbit.Net. Current Upbit.Net does not expose credentials, account, trading, wallet, or private websocket APIs.
---

# Upbit.Net

## Overview

Use `Upbit.Net` for public Upbit spot quotation data. The current source exposes public REST market data and public websocket streams only. Do not invent credentials, account, trading, wallet, withdrawal, deposit, or private user-stream APIs.

Use `cryptoexchange-net` when the user wants portable public market-data code through `CryptoExchange.Net.SharedApis`.

## Setup

Install the package:

```bash
dotnet add package JKorf.Upbit.Net
```

Use these namespaces:

```csharp
using CryptoExchange.Net.Objects;
using Upbit.Net;
using Upbit.Net.Clients;
using Upbit.Net.Enums;
```

Add `using CryptoExchange.Net.SharedApis;` only for SharedApis code.

## Client Roots

```csharp
var rest = new UpbitRestClient();
var socket = new UpbitSocketClient();
```

Available surfaces:

- `rest.SpotApi.ExchangeData`: symbols, trades, tickers, order books, klines, symbol configuration
- `rest.SpotApi.SharedClient`: shared public REST market-data interfaces
- `socket.SpotApi`: ticker, trade, order book, and kline subscriptions
- `socket.SpotApi.SharedClient`: shared public socket interfaces

There is no `SpotApi.Account`, `SpotApi.Trading`, futures API, or authenticated websocket surface.

## Regional Environments

Upbit provides separate live regions:

- `UpbitEnvironment.Live`: South Korea; common quotes `KRW`, `BTC`, `USDT`
- `UpbitEnvironment.Singapore`: common fiat quote `SGD`
- `UpbitEnvironment.Indonesia`: common fiat quote `IDR`
- `UpbitEnvironment.Thailand`: common fiat quote `THB`

```csharp
var client = new UpbitRestClient(options =>
{
    options.Environment = UpbitEnvironment.Singapore;
});
```

Select the region matching the requested market and apply it consistently to REST and socket clients.

## Result Handling

REST methods return `HttpResult<T>`. Websocket subscriptions return `WebSocketResult<UpdateSubscription>`. Shared symbol/cache helpers can return `ExchangeCallResult<T>`. Check `Success` before reading `Data`.

```csharp
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("USDT-ETH");
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

Retry only when `result.Error?.IsTransient == true`.

## Symbols

- Native symbols use `QUOTE-BASE` format, for example `KRW-BTC`, `USDT-ETH`, `SGD-BTC`, `IDR-BTC`, or `THB-BTC`.
- Do not use Binance-style `ETHUSDT` or OKX-style `ETH-USDT` for native Upbit calls.
- Verify availability in the selected region with `GetSymbolsAsync(includeNotifications: true)`.
- `GetTickersByQuoteAssetsAsync(...)` retrieves tickers by regional quote assets.

## REST Patterns

```csharp
var symbols = await client.SpotApi.ExchangeData.GetSymbolsAsync(includeNotifications: true);
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("USDT-ETH");
var trades = await client.SpotApi.ExchangeData.GetTradeHistoryAsync("USDT-ETH", limit: 20);
var klines = await client.SpotApi.ExchangeData.GetKlinesAsync(
    "USDT-ETH", KlineInterval.OneMinute, limit: 20);
var book = await client.SpotApi.ExchangeData.GetOrderBookAsync("USDT-ETH", levels: 15);
```

Read `references/usage.md` for complete snippets with error handling.

## Websocket Pattern

```csharp
var socket = new UpbitSocketClient();
var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "USDT-ETH",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Keep handlers fast and unsubscribe on shutdown.

## SharedApis

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new UpbitRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "ETH", "USDT")));
```

Upbit.Net SharedApis covers public market data only. Do not generate shared account or trading calls for Upbit.

## Dependency Injection

```csharp
services.AddUpbit(options =>
{
    options.Environment = UpbitEnvironment.Live;
});
```

Inject `IUpbitRestClient`, `IUpbitSocketClient`, `IUpbitOrderBookFactory`, or `IUpbitTrackerFactory`.

## Safety Rules

- Read `references/safety.md` before generating regional, symbol, websocket, order-book, or capability guidance.
- Never request or emit API keys for this public-only API surface.
- Never generate account, order placement, transfer, deposit, withdrawal, or private stream methods.
- Validate native symbols and region before requesting data.
- Use supported order-book levels and kline intervals.

## References

- Read `references/api-surfaces.md` for endpoint groups, shared interfaces, regional environments, result types, and supported streams.
- Read `references/usage.md` for task-specific snippets.
- Read `references/safety.md` for capability, region, symbol, websocket, and retry guardrails.
- In a maintainer checkout with sibling repositories, read `../Upbit.Net/Examples/ai-friendly/` for complete compilable examples.
