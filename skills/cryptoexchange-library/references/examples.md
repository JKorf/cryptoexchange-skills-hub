# AI-Friendly Examples

Maintain `Examples/ai-friendly` as short, self-contained files that coding agents can copy into a console app.

## File Set

Use this shape when applicable:

- `01-spot-quickstart.cs`: client setup, public ticker, authenticated balances, safe limit order, status, cleanup
- `02-futures.cs`: futures market data, leverage or margin settings if supported, order, position lookup, reduce-only close
- `03-websocket.cs`: public subscriptions, private streams if supported, proper teardown
- `04-multi-exchange.cs`: `CryptoExchange.Net.SharedApis` usage
- `05-error-handling.cs`: `WebCallResult` handling, common exchange-specific errors, retry boundaries

## Rules

- Each file should compile after `dotnet new console`, `dotnet add package <PackageId>`, and copying into `Program.cs`.
- Keep examples self-contained; avoid shared helpers.
- Use placeholders for credentials.
- Explain why important lines exist, especially result handling and cleanup.
- Prefer conservative order examples that are unlikely to fill immediately, then cancel.
- Update README tables when adding, removing, or renaming example files.
