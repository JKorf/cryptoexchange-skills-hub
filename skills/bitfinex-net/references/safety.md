# Bitfinex Safety

- Treat API keys as secrets; use placeholders or environment variables.
- Use read-only keys for examples that do not need trading, funding management, transfers, or withdrawals.
- Trading endpoints can place real orders. Prefer limit orders with explicit prices for examples.
- Use `OrderType.ExchangeLimit` for exchange-wallet spot limit orders; use margin order types only when the user explicitly asks for margin trading.
- Margin and derivatives workflows can create leveraged exposure; make this explicit before generating code that changes positions.
- Funding offer endpoints can lock funds or create real funding exposure. Include cancellation or cleanup for examples that submit offers.
- Always inspect `Success` before using `Data`.
- Include cleanup for example limit orders, funding offers, and subscriptions when appropriate.
- Do not generate withdrawal or wallet transfer code unless explicitly requested.

