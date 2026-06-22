# HyperLiquid.Net Safety

Use this file before generating HyperLiquid credential, account, transfer, withdrawal, staking, vault, builder-fee, trading, leverage, HIP-3 DEX, tracker, or websocket code.

## Credentials

- Use `HyperLiquidCredentials("PUBLIC_ADDRESS", "PRIVATE_KEY")`.
- The private key is an ECDSA signing key. Treat it as a wallet secret.
- Prefer environment variables, secret stores, or dependency-injected options.
- Never hardcode real public/private key pairs, destination addresses, vault addresses, validator addresses, or client order ids.
- Public market data examples should avoid credentials unless private endpoints are needed.

## Builder Fee

- HyperLiquid.Net defaults `BuilderFeePercentage` to `0.01m` (1 bps / 0.01%).
- Trading code should mention the default builder fee when relevant.
- Use `Account.GetApprovedBuilderFeeAsync()` to check whether the account approved the configured builder fee.
- Set `options.BuilderFeePercentage = 0` only when the user wants no builder fee.

## Result Handling

- REST methods return `HttpResult<T>` or `HttpResult`.
- Websocket request/query methods return `QueryResult<T>` or `QueryResult`.
- Websocket subscriptions return `WebSocketResult<UpdateSubscription>`.
- Shared non-I/O helpers can return `ExchangeCallResult<T>`.
- Always check `Success` before using `Data`.
- Use `Error.IsTransient` for retry decisions. Do not retry validation, credential, symbol, slippage, builder-fee, leverage, or account-state errors blindly.

## Symbols

- Native spot symbols use `Base/Quote`, for example `HYPE/USDC`.
- Native futures symbols use base asset only, for example `ETH`.
- Do not generate Binance-style `ETHUSDC`, Gate.io-style `ETH_USDC`, or OKX-style `ETH-USDC` for native HyperLiquid methods.
- SharedApis can format symbols, but native methods expect HyperLiquid-native symbols.

## Trading

- Prefer read-only examples unless the user explicitly asks for trading.
- Prefer conservative limit-order examples over market orders unless the user asks for immediate execution.
- Market orders still require a recent price. Fetch prices or ticker metadata first.
- Include cancellation cleanup when demonstrating live order placement.
- Use `reduceOnly: true` in close-position examples to avoid flipping exposure.
- Client order ids must be 128-bit hex strings if supplied.
- Batch methods can return nested per-item failures even when the outer result succeeds.
- The deadman switch `CancelAfterAsync` may require sufficient trading volume; do not assume it is available for every account.

## Futures, Margin, And HIP-3 DEX

- Futures orders create leveraged exposure. Say so in generated examples when relevant.
- `SetLeverageAsync` changes account/symbol leverage state.
- `UpdateIsolatedMarginAsync` adds or removes isolated margin and can affect liquidation risk.
- HIP-3 DEX-aware methods take optional `dex` parameters. Use them only when requested or when the target workflow needs a non-default DEX.
- For position-closing examples, inspect positions first and use the correct opposite side.

## Transfers, Withdrawals, Staking, And Vaults

- Transfer, withdrawal, staking, validator delegation, and vault actions move assets or affect wallet state. Generate them only when explicitly requested.
- Validate destination address, subaccount, asset/token id, quantity, vault address, validator, and direction before sending.
- `WithdrawAsync` initiates the bridge withdrawal flow and may incur fees and finalization delay.
- `TransferInternalAsync` moves USD between spot and perp account classes.
- `TransferSpotAsync` sends spot assets to another address.
- `DepositIntoStakingAsync`, `WithdrawFromStakingAsync`, `DelegateOrUndelegateStakeFromValidatorAsync`, and `DepositOrWithdrawFromVaultAsync` should be treated as live asset operations.

## Websockets And Trackers

- Keep handlers fast. Offload heavier work to a queue, channel, or background service.
- Check subscription `Success` before assuming a stream is live.
- Always unsubscribe on shutdown with `UnsubscribeAsync` or `UnsubscribeAllAsync`.
- Authenticated streams use the credential address when `address` is `null`.
- Socket query and order methods return `QueryResult<T>` and can place/cancel live orders. Treat them like trading endpoints.
- For long-running user data services, consider `IHyperLiquidTrackerFactory` instead of ad hoc subscription management.

## SharedApis

- Use SharedApis when the user needs exchange-agnostic code.
- Use native HyperLiquid APIs for builder fees, vaults, staking, HIP-3 DEX, detailed TWAP/trigger behavior, and other HyperLiquid-specific features.
- Do not mix native HyperLiquid request/model types with SharedApis request/model types.
