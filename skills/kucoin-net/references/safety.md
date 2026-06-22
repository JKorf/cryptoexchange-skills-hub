# Kucoin.Net Safety

Use this file before generating Kucoin credential, account, transfer, withdrawal, Earn, margin, lending, futures, leverage, trading, tracker, private websocket, or Unified account code.

## Credentials

- Install `Kucoin.Net`, not a guessed package id.
- KuCoin credentials require key, secret, and passphrase: `new KucoinCredentials(key, secret, pass)`.
- Prefer environment variables, secret stores, or dependency-injected options.
- Never hardcode real API keys, secrets, passphrases, wallet addresses, memos, withdrawal ids, or client order ids.
- Private spot, futures, Unified, transfer, withdrawal, margin, Earn, and websocket streams require credentials with the correct exchange permissions enabled.

## Result Handling

- REST methods return `HttpResult<T>` or `HttpResult`.
- Websocket subscriptions return `WebSocketResult<UpdateSubscription>`.
- Shared non-I/O helpers can return `ExchangeCallResult<T>`.
- Always check `Success` before using `Data`.
- Use `Error.IsTransient` for retry decisions. Do not retry validation, permission, passphrase, insufficient balance, leverage, margin, withdrawal, or order rejection errors blindly.

## Symbols

- Spot symbols use names such as `BTC-USDT` and `ETH-USDT`.
- Futures symbols are contract names such as `ETHUSDTM` and `XBTUSDTM`.
- Do not generate Binance-style `BTCUSDT` for spot or `BTC-USDT` for futures contract endpoints.
- Fetch spot metadata with `SpotApi.ExchangeData.GetSymbolAsync(...)` and use `BaseIncrement`, `PriceIncrement`, and related metadata before sizing orders.
- Fetch futures contract metadata with `FuturesApi.ExchangeData.GetContractAsync(...)` before sizing futures orders.

## Spot Trading

- Prefer `PlaceTestOrderAsync` for spot order examples that demonstrate order syntax.
- Use `PlaceOrderAsync` only when the user explicitly wants a live order.
- Include cancellation cleanup when demonstrating live limit order placement.
- Spot market buys can use base `quantity` or quote `quoteQuantity` depending on the workflow; be explicit.
- Bulk methods can return nested per-item failures even when the outer result succeeds.
- High-frequency spot endpoints under `HfTrading` are separate from regular spot trading. Use them only when requested.

## Futures Trading

- Futures orders create leveraged exposure. Say so in generated examples when relevant.
- Prefer `FuturesApi.Trading.PlaceTestOrderAsync` for futures order syntax examples.
- `FuturesApi.Trading.PlaceOrderAsync` places live futures orders. Use read-only or test futures examples unless the user asks to trade.
- Use `reduceOnly: true` for close-position examples.
- Read positions with `FuturesApi.Account.GetPositionAsync(symbol)` or `GetPositionsAsync(...)` before close-position examples.
- `FuturesApi.Account.SetMarginModeAsync`, `SetMarginModesAsync`, `SetCrossMarginLeverageAsync`, and position/risk-limit methods change live account state.
- Futures quantity is contract count unless parameters such as `quantityInBaseAsset` or `quantityInQuoteAsset` are used.

## Unified Account

- `UnifiedApi` is native KuCoin-specific. It does not expose `SharedClient`.
- `UnifiedApi.Account.SetAccountModeAsync`, `SetLeverageAsync`, `SetCrossMarginLeverageAsync`, `TransferAsync`, and `WithdrawAsync` change live account state.
- Unified websocket methods require the correct `UnifiedAccountType`; do not assume spot if the user asks for futures or margin.
- Use native Unified models and request parameters, not SharedApis request models.

## Earn, Margin, Transfers, And Withdrawals

- Earn purchase/redeem methods move funds. Generate them only when explicitly requested.
- Margin borrow, repay, lending subscribe/redeem, leverage multiplier, and isolated/cross margin changes affect live balances or risk. Prefer read-only margin examples unless asked otherwise.
- Withdrawal code should be generated only when explicitly requested.
- Validate asset, network/chain, destination address, memo/tag, amount, fee deduction, and internal/external withdrawal mode before sending.
- Transfers move funds between accounts. Make source account, destination account, asset, and quantity explicit.
- Prefer read-only deposit/withdrawal status and quota examples unless the user asks for live movement.

## Websockets And Trackers

- Keep handlers fast. Offload heavier work to a queue, channel, or background service.
- Check subscription `Success` before assuming a stream is live.
- Always unsubscribe on shutdown with `UnsubscribeAsync` or `UnsubscribeAllAsync`.
- Private spot, futures, margin, and Unified streams require credentials and appropriate API permissions.
- For long-running user data services, consider `IKucoinTrackerFactory` instead of ad hoc subscription management.

## SharedApis

- Use SharedApis when the user needs exchange-agnostic code.
- Use native Kucoin APIs for Unified account, high-frequency spot trading, Earn, detailed margin, wallet transfer/withdrawal details, and futures account management.
- Do not mix native Kucoin request/model types with SharedApis request/model types.
