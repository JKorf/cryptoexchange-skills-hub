# Upbit.Net Safety

Use this file before generating Upbit regional, symbol, websocket, local order-book, tracker, or capability guidance.

## Capability Boundary

- Install `JKorf.Upbit.Net`, not `Upbit.Net` or another guessed package id.
- Current Upbit.Net exposes public quotation data only.
- Do not generate credentials, API-key configuration, balances, account endpoints, trading, orders, deposits, withdrawals, transfers, wallet methods, or private streams.
- Do not claim SharedApis account or trading support.

## Result Handling

- REST methods return `HttpResult<T>`.
- Websocket subscriptions return `WebSocketResult<UpdateSubscription>`.
- Shared cache helpers can return `ExchangeCallResult<T>`.
- Always check `Success` before using `Data`.
- Retry only transient failures. Do not retry invalid symbols, unsupported intervals, invalid order-book levels, or wrong-region markets blindly.

## Regions And Symbols

- Select `UpbitEnvironment.Live`, `Singapore`, `Indonesia`, or `Thailand` deliberately.
- Native symbols use `QUOTE-BASE`, such as `KRW-BTC` or `USDT-ETH`.
- Do not generate Binance-style `ETHUSDT`, OKX-style `ETH-USDT`, or reversed `BASE-QUOTE` names for native calls.
- Validate symbols with `GetSymbolsAsync(includeNotifications: true)` in the selected environment.
- Match regional fiat quotes: `KRW`, `SGD`, `IDR`, or `THB`.

## Websockets

- Keep handlers fast. Offload heavier work to a queue, channel, or background service.
- Check subscription `Success` before assuming a stream is live.
- Always unsubscribe with `UnsubscribeAsync` or `UnsubscribeAllAsync` on shutdown.
- Use supported order-book levels; common shared levels are `1`, `5`, `15`, and `30`.
- Use `KlineInterval` enum values rather than arbitrary interval strings.

## Local Order Books And Trackers

- Create native local order books with `IUpbitOrderBookFactory.CreateSpot(symbol, ...)`.
- Start and stop local order books explicitly and handle failed `StartAsync` results.
- Use `IUpbitTrackerFactory` for shared public kline or trade tracking workflows.
- Keep long-lived order books and trackers managed by application services instead of creating one per request.

## SharedApis

- Use SharedApis for exchange-agnostic public market data.
- Use native Upbit APIs for regional selection, detailed symbol metadata, native order-book aggregation, and Upbit-specific models.
- Do not mix native Upbit models with SharedApis models.
