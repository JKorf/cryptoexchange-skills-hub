# BingX Safety

- Treat API keys and secrets as secrets; use placeholders, environment variables, or secret storage.
- Use read-only public clients for market-data examples.
- Use read-only keys for examples that do not need trading.
- Trading endpoints can place real orders. Prefer limit orders with explicit prices for examples.
- Perpetual futures endpoints can change leverage and create real derivatives exposure; only include futures trading code when the user explicitly asks.
- Always inspect `Success` before using `Data`.
- Include cleanup for example limit orders when appropriate.
- Always unsubscribe websocket subscriptions when examples end.
- Do not generate withdrawal, transfer, or subaccount-management code unless explicitly requested.
- Use `PerpetualFuturesApi`, not an invented `FuturesApi`.
