# BitMart Safety

- Treat API keys, secrets, and memo/passphrase values as secrets; use placeholders or environment variables.
- Use read-only keys for examples that do not need trading, margin, futures, sub-account transfers, transfers, or withdrawals.
- Trading endpoints can place real orders. Prefer limit orders with explicit prices for examples.
- USD futures examples can change leverage, position mode, and create real derivatives exposure; only include futures trading code when the user explicitly asks.
- Spot margin borrow, repay, and margin-order endpoints can create borrowed exposure; only generate those writes when explicitly requested.
- Sub-account transfer endpoints move funds between accounts; do not generate them unless explicitly requested.
- Always inspect `Success` before using `Data`.
- Include cleanup for example limit orders and subscriptions when appropriate.
- Do not generate withdrawal code unless explicitly requested.

