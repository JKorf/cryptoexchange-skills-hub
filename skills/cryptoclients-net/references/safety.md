# CryptoClients.Net Safety

## Fan-Out And Results

- Aggregate calls can contact many exchanges concurrently; pass an explicit exchange list unless broad fan-out is intentional.
- Check every `HttpResult` or `WebSocketResult` independently.
- Treat unsupported capabilities as normal per-exchange failures.
- Bound retries and retry only transient failures. Never retry rejected orders or non-idempotent writes blindly.
- Respect combined rate limits, connection counts, and request volume across all selected exchanges.

## Credentials

- Use placeholders or secret storage; never commit real keys, secrets, passphrases, private keys, or addresses.
- Prefer typed `ExchangeCredentials` for known exchanges.
- Inspect `GetDynamicCredentialInfo` before using `DynamicCredentials`.
- Avoid the obsolete string credential overload because exchange credential shapes differ.
- Use read-only keys unless trading or fund movement is explicitly required.

## Trading And Funds

- Aggregate order methods place live orders on the named exchange.
- Validate symbol support, trading mode, precision, minimums, balances, fees, order type, time in force, and exchange-specific semantics.
- SharedApis intentionally omit some native options; switch to the native client instead of guessing.
- Leverage, position mode, trigger orders, TP/SL, transfers, and withdrawals can alter exposure or move funds.
- Generate withdrawals or broad multi-exchange writes only when explicitly requested.

## Websockets And Trackers

- Keep handlers fast and unsubscribe during shutdown.
- `UnsubscribeAllAsync()` closes every connection managed by the aggregate socket client.
- Starting cross-exchange books or trackers can open many sockets and issue snapshot requests.
- User-data trackers require credentials and maintain private account state; stop and dispose them with the application lifecycle.

## Native Access

- Native properties bypass aggregate normalization and use exchange-specific symbols, enums, models, credentials, and behavior.
- Read the corresponding exchange skill before native trading or account work.
- Do not pass SharedApis request/model types to native endpoint methods.
