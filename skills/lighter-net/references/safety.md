# Lighter Safety

- Treat API secrets, public addresses, account indexes, and API key indexes as sensitive operational data; use placeholders or environment variables.
- Use read-only examples for account inspection when possible.
- Lighter trading endpoints can place, edit, and cancel real orders. Prefer limit orders with explicit prices for examples.
- Lighter signer-backed endpoints can change leverage, margin, account tier, and live order state; only include them when the user explicitly asks.
- Let the library manage nonces unless the user has a specific nonce coordination requirement.
- Always inspect `Success` before using `Data`.
- Include cleanup for example limit orders and websocket subscriptions when appropriate.
- Do not generate withdrawal or transfer automation unless explicitly requested.
