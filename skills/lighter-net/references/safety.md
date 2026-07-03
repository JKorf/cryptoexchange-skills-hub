# Lighter Safety

- Treat API secrets, private keys, account indexes, and API key indexes as sensitive operational data; use placeholders or environment variables.
- Use `EthKey.FromPrivateKey(...)` when L1 signing is needed. Use `EthKey.FromPublicKey(...)` only for authenticated API-key requests that do not need layer-1 signatures.
- Do not pass a raw public address string to `LighterCredentials` in version 1.1.0+.
- Use read-only examples for account inspection when possible.
- Lighter trading endpoints can place, edit, and cancel real orders. Prefer limit orders with explicit prices for examples.
- Lighter signer-backed endpoints can change leverage, margin, account tier, and live order state; only include them when the user explicitly asks.
- Lighter.Net uses Lighter's optional integrator-code mechanism by default. Only set `IntegratorFeePercentage = 0m` or `null` when the user explicitly asks to disable it.
- Let the library manage nonces unless the user has a specific nonce coordination requirement.
- Always inspect `Success` before using `Data`.
- Include cleanup for example limit orders and websocket subscriptions when appropriate.
- Do not generate withdrawal or transfer automation unless explicitly requested.
