# Implementation Guidance

Use the target exchange repository as the primary source of truth. Match its existing layout and naming.

## Preferred Patterns

- Use `CryptoExchange.Net` base clients, request definitions, result types, websocket infrastructure, authentication providers, serializers, and rate limiter support.
- Keep API surface async.
- Preserve nullable annotations and model attributes used elsewhere in the repository.
- Return `WebCallResult<T>` for REST calls and subscription result types for socket calls.
- Map exchange errors into the existing error model.
- Keep request parameter validation close to existing local patterns.
- Do not introduce custom retry loops in endpoint methods unless the repository already does so for that scenario.

## SharedApis

When adding or changing shared clients:

- Implement the narrow shared interface only when the exchange supports the required semantics.
- Normalize symbols with `SharedSymbol`.
- Normalize order status, side, type, time in force, balances, and positions to the shared models.
- Add tests for shared request validation and expected unsupported-feature behavior.

## Credentials

- Use the exchange-specific credential type when the exchange needs more than key/secret.
- Do not reintroduce legacy generic `ApiCredentials` patterns removed in recent CryptoExchange.Net versions.
- Keep signing in the authentication provider, not in endpoint methods.

## Rate Limits

- Model rate limits with the existing request definition/rate limiter mechanisms.
- Prefer endpoint-specific weights when the exchange documents them.
- Avoid ad hoc sleeps in client methods.

## Tests

- Add or update unit tests for request generation, model deserialization, shared API behavior, and error handling.
- For models, include representative exchange JSON including optional and missing fields.
- For socket parsing, cover subscription confirmations and data events separately when the repository pattern supports it.
