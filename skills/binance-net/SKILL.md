---
name: binance-net
description: Build C#/.NET Binance integrations with Binance.Net, including spot and futures REST clients, websocket subscriptions, account reads, order placement/cancellation, BinanceCredentials, Binance-specific symbols, enums, and SharedApis access. Use when the user asks for Binance market data, Binance account or trading code, Binance futures, Binance websocket updates, or converting raw Binance API usage to idiomatic Binance.Net.
---

# Binance.Net

## Overview

Use `Binance.Net` for Binance-specific C# application code. Prefer this skill over the generic CryptoExchange.Net skill when the user asks for Binance-only behavior or endpoint names.

## Setup

```bash
dotnet add package Binance.Net
```

Use these namespaces in examples:

```csharp
using Binance.Net;
using Binance.Net.Clients;
using Binance.Net.Enums;
using Binance.Net.Objects;
```

## Client Patterns

Public market data:

```csharp
var client = new BinanceRestClient();
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTCUSDT");
```

Authenticated calls:

```csharp
var client = new BinanceRestClient(options =>
{
    options.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
});
```

Handle all results:

```csharp
if (!ticker.Success)
{
    Console.WriteLine(ticker.Error);
    return;
}
```

Use `client.SpotApi.SharedClient` for cross-exchange SharedApis code.

## Binance Conventions

- Spot symbols usually use compact format, for example `BTCUSDT`.
- Common spot order enums include `OrderSide`, `SpotOrderType`, and `TimeInForce`.
- Let the library generate client order IDs unless the user needs external correlation.
- Reuse clients; do not create a new REST or socket client per request.
- Prefer maintained ai-friendly examples before guessing endpoint names.

## References

- Read `references/usage.md` for task-specific patterns.
- Read `references/safety.md` before generating trading, leverage, futures, or credential code.
- In the maintainer workspace, read `C:\Projects\Binance.Net\Examples\ai-friendly\` for complete compilable examples.
