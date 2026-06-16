---
name: kucoin-net
description: Build C#/.NET Kucoin integrations with Kucoin.Net, including spot and futures REST clients, websocket subscriptions, account reads, order placement/cancellation, KucoinCredentials with passphrase, Kucoin symbol formats, enums, and SharedApis access. Use when the user asks for Kucoin market data, Kucoin account or trading code, Kucoin futures, Kucoin websocket updates, or converting raw Kucoin API usage to idiomatic Kucoin.Net.
---

# Kucoin.Net

## Overview

Use `Kucoin.Net` for Kucoin-specific C# application code. Prefer this skill over the generic CryptoExchange.Net skill when the user asks for Kucoin-only behavior or endpoint names.

## Setup

```bash
dotnet add package Kucoin.Net
```

Use these namespaces in examples:

```csharp
using Kucoin.Net;
using Kucoin.Net.Clients;
using Kucoin.Net.Enums;
```

## Client Patterns

Public market data:

```csharp
var client = new KucoinRestClient();
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTC-USDT");
```

Authenticated calls require key, secret, and passphrase:

```csharp
var client = new KucoinRestClient(options =>
{
    options.ApiCredentials = new KucoinCredentials("API_KEY", "API_SECRET", "API_PASSPHRASE");
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

## Kucoin Conventions

- Spot symbols usually use dash format, for example `BTC-USDT`.
- Credentials require a passphrase in addition to API key and secret.
- Common spot order enums include `OrderSide`, `NewOrderType`, and `TimeInForce`.
- Let the library generate client order IDs unless the user needs external correlation.
- Reuse clients; do not create a new REST or socket client per request.

## References

- Read `references/usage.md` for task-specific patterns.
- Read `references/safety.md` before generating trading, leverage, futures, or credential code.
- In the maintainer workspace, read `C:\Projects\Kucoin.Net\Examples\ai-friendly\` for complete compilable examples.
