# CryptoExchange.Net Skills Hub

Installable skills for AI coding assistants working with the CryptoExchange.Net ecosystem.

This repository is intentionally agent-facing. It complements the normal documentation by giving coding agents compact, high-signal instructions for using the C# libraries correctly: package names, client patterns, `WebCallResult<T>` handling, SharedApis, websocket cleanup, credentials, and library implementation conventions.

## Skills

| Skill | Use when |
| --- | --- |
| `cryptoexchange-net` | Building exchange-agnostic C# integrations with `CryptoExchange.Net.SharedApis`, `CryptoClients.Net`, or multiple exchange libraries |
| `binance-net` | Building Binance REST, websocket, account, or trading workflows with `Binance.Net` |
| `kucoin-net` | Building Kucoin REST, websocket, account, or trading workflows with `Kucoin.Net` |
| `cryptoexchange-library` | Creating or updating an exchange-specific library built on `CryptoExchange.Net` |

## Ecosystem Scope

The hub is for the open source CryptoExchange.Net library family, including Aster.Net, Binance.Net, BingX.Net, Bitfinex.Net, Bitget.Net, BitMart.Net, BitMEX.Net, Bitstamp.Net, BloFin.Net, Bybit.Net, Coinbase.Net, CoinEx.Net, CoinGecko.Net, CoinW.Net, CryptoClients.Net, CryptoCom.Net, DeepCoin.Net, GateIo.Net, HTX.Net, HyperLiquid.Net, Kraken.Net, Kucoin.Net, Lighter.Net, Mexc.Net, OKX.Net, Polymarket.Net, Toobit.Net, Upbit.Net, Weex.Net, WhiteBit.Net, and XT.Net.

## Design Principles

- Prefer working examples over broad prose.
- Prefer `SharedApis` for cross-exchange workflows.
- Prefer exchange-specific clients when the user asks for exchange-only features.
- Keep trading examples conservative and explicit.
- Never hide failed API calls; always inspect `Success` and `Error`.
- Do not duplicate complete API docs inside skills. Point agents to local `Examples/ai-friendly` files and official docs when detail is needed.

## Suggested Next Additions

- Add per-exchange skills for Bybit.Net, OKX.Net, Kraken.Net, Mexc.Net, Bitget.Net, GateIo.Net, and Coinbase.Net.
- Add generated per-exchange package metadata from the `.csproj` files.
- Add a validation script that checks all `SKILL.md` files and ensures referenced local example paths still exist.
