# Safety Guidance

Use conservative defaults for examples involving account access, trading, withdrawal, leverage, or API keys.

## Required Practices

- Never hardcode real credentials.
- Use environment variables or placeholder strings in examples.
- Prefer read-only permissions for market data and account-balance examples.
- Mention required credential shape when it differs by exchange, such as Kucoin passphrase.
- Always inspect `Success` and `Error` before reading `Data`.
- For trading examples, use small quantities and safe limit prices when an example must place an order.
- Include cleanup cancellation for example orders when appropriate.
- Do not generate withdrawal examples unless the user explicitly asks.

## Trading Examples

When the user asks for trading code:

1. Prefer testnet/sandbox if the exchange library supports it.
2. Make order intent explicit: side, order type, quantity, price, time in force.
3. Avoid market orders unless the user explicitly asks for immediate execution.
4. Include a warning comment in code that the call can place a real order.
5. Return or log exchange errors without swallowing them.
