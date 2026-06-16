---
name: cryptoexchange-net
description: Build C#/.NET applications with the CryptoExchange.Net ecosystem, especially exchange-agnostic workflows using CryptoExchange.Net.SharedApis, CryptoClients.Net, WebCallResult handling, websocket subscriptions, symbol normalization, DI setup, and multi-exchange code. Use when the user asks to integrate multiple crypto exchanges, write code that can swap between Binance/Kucoin/OKX/Bybit/Kraken/etc., use shared clients, inspect balances/tickers/orders through common interfaces, or choose the right CryptoExchange.Net-based package.
---

# CryptoExchange.Net

## Overview

Use this skill for application code that consumes CryptoExchange.Net-based libraries. CryptoExchange.Net itself is the base library; install exchange-specific packages or `CryptoClients.Net` for actual API access.

## First Choice

1. Use `CryptoExchange.Net.SharedApis` when the user wants code that works across exchanges.
2. Use an exchange-specific skill when the user asks for a single exchange or endpoint that is not in the shared interface.
3. Use `CryptoClients.Net` when the user explicitly wants one package containing all exchange clients.
4. Read `references/ecosystem.md` when choosing package IDs, repository names, or supported exchange coverage.

## Core Patterns

Use shared clients exposed from exchange API surfaces:

```csharp
using Binance.Net.Clients;
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new BinanceRestClient().SpotApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");
var result = await tickers.GetSpotTickerAsync(new GetTickerRequest(symbol));

if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

Always handle `WebCallResult<T>`/`CallResult<T>` explicitly. Do not assume `Data` is usable when `Success` is false.

Normalize cross-exchange symbols with `SharedSymbol` instead of passing raw exchange-specific strings into shared interfaces.

Close websocket subscriptions when the workflow ends:

```csharp
var subscription = await socketClient.SubscribeToTickerUpdatesAsync(request, update => { });
if (subscription.Success)
    await subscription.Data.CloseAsync();
```

## References

- Read `references/shared-apis.md` for cross-exchange client patterns and example prompts.
- Read `references/ecosystem.md` for repository and NuGet package names across the ecosystem.
- Read `references/safety.md` before generating trading, withdrawal, or credential-handling code.

## Local Example Sources

When working inside the maintainer workspace, prefer these examples before inventing method names:

- `C:\Projects\CryptoExchange.Net\Examples\ai-friendly\`
- `C:\Projects\Binance.Net\Examples\ai-friendly\`
- `C:\Projects\Kucoin.Net\Examples\ai-friendly\`
