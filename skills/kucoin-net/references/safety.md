# Kucoin Safety

- Treat API keys, secrets, and passphrases as secrets; use placeholders or environment variables.
- Use read-only keys for examples that do not need trading.
- Trading endpoints can place real orders. Prefer limit orders with explicit prices for examples.
- Kucoin futures examples can create real derivatives exposure; only include futures trading code when the user explicitly asks.
- Always inspect `Success` before using `Data`.
- Include cleanup for example limit orders when appropriate.
- Do not generate withdrawal code unless explicitly requested.
