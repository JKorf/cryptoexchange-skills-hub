# Bitstamp Safety

- Treat API keys and secrets as secrets; use placeholders or environment variables.
- Use read-only keys for examples that do not need trading, derivatives, leverage, collateral, transfer, or withdrawal operations.
- Trading endpoints can place real orders. Prefer limit orders with explicit prices for examples.
- Derivative position, leverage, close-position, and collateral update endpoints can materially change account risk; only generate those writes when explicitly requested.
- Transfer and withdrawal endpoints move funds; do not generate them unless explicitly requested.
- Always inspect `Success` before using `Data`.
- Include cleanup for example limit orders and subscriptions when appropriate.

