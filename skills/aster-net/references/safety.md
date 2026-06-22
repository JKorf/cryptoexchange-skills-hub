# Aster Safety

- Treat user private keys and signer private keys as secrets; use placeholders, environment variables, or secret storage.
- Never paste real private keys into examples.
- Use read-only public clients for market-data examples.
- Trading endpoints can place real orders. Prefer limit orders with explicit prices for examples.
- Futures endpoints can change leverage and create real derivatives exposure; only include futures trading code when the user explicitly asks.
- Always inspect `Success` before using `Data`.
- Include cleanup for example limit orders when appropriate.
- Always unsubscribe websocket subscriptions when examples end.
- Do not generate withdrawal or transfer code unless explicitly requested.
- Do not use V1 credentials or V1 API surfaces unless the user explicitly asks for V1.
