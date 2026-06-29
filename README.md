# CryptoExchange.Net Skills Hub

Installable skills for AI coding assistants working with the CryptoExchange.Net ecosystem.

This repository is intentionally agent-facing. It complements the normal documentation by giving coding agents compact, high-signal instructions for using the C# libraries correctly: package names, client patterns, `HttpResult<T>` and `WebSocketResult<T>` handling, SharedApis, websocket cleanup, credentials, and library implementation conventions.

## Installation

Install only the skills you need so agent context stays focused. The [`skills`](https://skills.sh/) CLI detects supported agents and can install from a local checkout, GitHub shorthand, or a GitHub URL.

List the skills in this checkout:

```bash
npx skills add . --list
```

Install one skill and choose the target agent when prompted:

```bash
npx skills add . --skill binance-net
```

Install every skill for every detected agent without prompts:

```bash
npx skills add . --all
```

The source argument can also be a GitHub repository, for example:

```bash
npx skills add JKorf/cryptoexchange-skills-hub --skill binance-net
```

Use `--agent <agent>` to select an agent explicitly, `--global` for a user-level installation, or `--copy` to copy files instead of linking them. Run `npx skills update` to update installed remote skills. For a linked local installation, update this checkout instead.

### Manual Installation

For agents that support the `SKILL.md` convention but are not handled by the CLI, copy the selected directory to the agent's documented skills location:

```bash
skill=binance-net
agent_skills_dir="$HOME/.your-agent/skills"
mkdir -p "$agent_skills_dir/$skill"
cp -R "./skills/$skill/." "$agent_skills_dir/$skill/"
```

## Skills

| Skill | Use when |
| --- | --- |
| `cryptoexchange-net` | Building exchange-agnostic C# integrations with `CryptoExchange.Net.SharedApis`, `CryptoClients.Net`, or multiple exchange libraries |
| `cryptoclients-net` | Building multi-exchange REST, websocket, SharedApis, native client, order book, tracker, or credential workflows with `CryptoClients.Net` |
| `aster-net` | Building Aster DEX Spot V3, Futures V3, REST, websocket, account, or trading workflows with `Aster.Net` |
| `binance-net` | Building Binance REST, websocket, account, or trading workflows with `Binance.Net` |
| `bingx-net` | Building BingX Spot, Perpetual Futures, REST, websocket, account, or trading workflows with `BingX.Net` |
| `bitfinex-net` | Building Bitfinex spot, margin, funding, REST, websocket, account, or trading workflows with `Bitfinex.Net` |
| `bitget-net` | Building Bitget Spot V2, Futures V2, UnifiedApi, copy trading, REST, websocket, account, or trading workflows with `Bitget.Net` |
| `bitmart-net` | Building BitMart spot, USD futures, margin, REST, websocket, account, or trading workflows with `BitMart.Net` |
| `bitmex-net` | Building BitMEX ExchangeApi, market data, orders, positions, REST, websocket, account, or trading workflows with `BitMEX.Net` |
| `bitstamp-net` | Building Bitstamp ExchangeApi, spot, derivatives, REST, websocket, account, or trading workflows with `Bitstamp.Net` |
| `blofin-net` | Building BloFin AccountApi, FuturesApi, REST, websocket, account, or futures trading workflows with `BloFin.Net` |
| `bybit-net` | Building Bybit V5 REST, websocket, account, spot, derivatives, options, spread, or trading workflows with `Bybit.Net` |
| `coinbase-net` | Building Coinbase Advanced Trade, Exchange market data, REST, websocket, account, portfolio, or trading workflows with `Coinbase.Net` |
| `coinex-net` | Building CoinEx SpotApiV2, FuturesApi, REST, websocket, account, margin, or trading workflows with `CoinEx.Net` |
| `coinw-net` | Building CoinW SpotApi, FuturesApi, REST, websocket, account, transfers, withdrawals, or trading workflows with `CoinW.Net` |
| `cryptocom-net` | Building Crypto.com ExchangeApi, REST, websocket, account, staking, derivatives, or trading workflows with `CryptoCom.Net` |
| `deepcoin-net` | Building DeepCoin ExchangeApi, spot, swaps, REST, websocket, account, leverage, or trading workflows with `DeepCoin.Net` |
| `gateio-net` | Building Gate.io SpotApi, PerpetualFuturesApi, RebateApi, AlphaApi, REST, websocket, account, futures, or trading workflows with `GateIo.Net` |
| `htx-net` | Building HTX SpotApi, UsdtFuturesApi, UsdtFuturesV5Api, REST, websocket, account, margin, futures, or trading workflows with `HTX.Net` |
| `hyperliquid-net` | Building HyperLiquid SpotApi, FuturesApi, REST, websocket, account, staking, vault, perpetual futures, or trading workflows with `HyperLiquid.Net` |
| `kraken-net` | Building Kraken SpotApi, FuturesApi, Earn, REST, websocket, account, futures, or trading workflows with `Kraken.Net` |
| `kucoin-net` | Building KuCoin SpotApi, FuturesApi, UnifiedApi, REST, websocket, account, margin, futures, or trading workflows with `Kucoin.Net` |
| `lighter-net` | Building Lighter DEX ExchangeApi, REST, websocket, account, order book, tracker, SharedApis, or trading workflows with `Lighter.Net` |
| `mexc-net` | Building MEXC SpotApi, FuturesApi, REST, websocket, account, private streams, futures, or trading workflows with `Mexc.Net` |
| `okx-net` | Building OKX UnifiedApi, REST, websocket, account, spot, derivatives, copy trading, socket order request, or trading workflows with `OKX.Net` |
| `polymarket-net` | Building Polymarket CLOB, Gamma, Data API, REST, websocket, token order book, account, auth, or trading workflows with `Polymarket.Net` |
| `toobit-net` | Building Toobit SpotApi, UsdtFuturesApi, REST, websocket, account, user stream, futures, or trading workflows with `Toobit.Net` |
| `upbit-net` | Building Upbit regional public SpotApi REST, websocket, market data, local order book, tracker, or SharedApis workflows with `Upbit.Net` |
| `weex-net` | Building Weex SpotApi, FuturesApi, REST, websocket, account, private stream, leverage, conditional order, or trading workflows with `Weex.Net` |
| `whitebit-net` | Building WhiteBit V4Api REST, websocket, spot, collateral/perpetual, account, fund movement, subaccount, or trading workflows with `WhiteBit.Net` |
| `xt-net` | Building XT SpotApi, USDT-M and Coin-M futures REST, combined futures websocket, account, private stream, leverage, or trading workflows with `XT.Net` |

## Ecosystem Scope

The hub is for the open source CryptoExchange.Net library family, including Aster.Net, Binance.Net, BingX.Net, Bitfinex.Net, Bitget.Net, BitMart.Net, BitMEX.Net, Bitstamp.Net, BloFin.Net, Bybit.Net, Coinbase.Net, CoinEx.Net, CoinGecko.Net, CoinW.Net, CryptoClients.Net, CryptoCom.Net, DeepCoin.Net, GateIo.Net, HTX.Net, HyperLiquid.Net, Kraken.Net, Kucoin.Net, Lighter.Net, Mexc.Net, OKX.Net, Polymarket.Net, Toobit.Net, Upbit.Net, Weex.Net, WhiteBit.Net, and XT.Net.

## Design Principles

- Prefer working examples over broad prose.
- Prefer `SharedApis` for cross-exchange workflows.
- Prefer exchange-specific clients when the user asks for exchange-only features.
- Keep trading examples conservative and explicit.
- Never hide failed API calls; always inspect `Success` and `Error`.
- Do not duplicate complete API docs inside skills. Point agents to local `Examples/ai-friendly` files and official docs when detail is needed.
