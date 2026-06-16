---
name: cryptoexchange-library
description: Create, update, or review exchange-specific C# libraries built on CryptoExchange.Net, including REST clients, socket clients, options, credentials, shared API implementations, integration tests, model parsing, rate limits, and ai-friendly examples. Use when working inside a CryptoExchange.Net-based library repository such as Binance.Net, Kucoin.Net, OKX.Net, Bybit.Net, Kraken.Net, Mexc.Net, or a new exchange client implementation.
---

# CryptoExchange Library

## Overview

Use this skill for maintainer work on exchange-specific libraries built on top of `CryptoExchange.Net`. This is not for ordinary app integration; use an exchange skill or `cryptoexchange-net` for consumer code.

## Workflow

1. Inspect the target repository conventions before editing.
2. Match existing client surface naming, folder layout, options types, enum style, and test patterns.
3. Prefer existing CryptoExchange.Net helpers over custom HTTP, websocket, serialization, auth, retry, or rate-limit code.
4. Keep endpoint models close to the exchange API while keeping shared interfaces normalized.
5. Update ai-friendly examples when public usage patterns change.
6. Build and run the most relevant unit tests for the touched library.

## Implementation Guidance

- Read `references/implementation.md` before changing clients, request definitions, auth, options, models, or shared API implementations.
- Read `references/examples.md` before adding or updating `Examples/ai-friendly` files.
- Use local sibling repositories as precedent. For example, compare similar features in `C:\Projects\Binance.Net`, `C:\Projects\Kucoin.Net`, `C:\Projects\OKX.Net`, and `C:\Projects\Bybit.Net`.

## Validation

For narrow changes, run the target project build and focused unit tests. For shared surface changes, also run tests that cover shared APIs and the most similar exchange implementation.
