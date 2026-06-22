# Bitget Safety

- Treat API keys, secrets, and passphrases as secrets; use placeholders or environment variables.
- Use read-only keys for examples that do not need trading, margin, futures, copy trading, transfers, or withdrawals.
- Trading endpoints can place real orders. Prefer limit orders with explicit prices for examples.
- Futures examples can change leverage, margin mode, position mode, and create real derivatives exposure; only include futures trading code when the user explicitly asks.
- Spot margin borrow, repay, and margin-order endpoints can create borrowed exposure; only generate those writes when explicitly requested.
- UTA/unified account endpoints can affect account-wide settings and balances; verify intent before generating write operations.
- Always inspect `Success` before using `Data`.
- Include cleanup for example limit orders and subscriptions when appropriate.
- Do not generate withdrawal or transfer code unless explicitly requested.

