# Bybit.Net API Surfaces

Use this reference when choosing where a Bybit endpoint belongs.

## REST Clients

```csharp
var client = new BybitRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.V5Api.ExchangeData` | Announcements, server time, spot symbols/tickers, linear/inverse symbols/tickers, options symbols/tickers, spread symbols/tickers, order books, klines, mark/index/premium klines, recent trades, funding, open interest, volatility, risk limits, price limits, system status |
| `client.V5Api.Account` | Wallet balances, asset balances, deposits, withdrawals, transfers, fees, transaction history, leverage, margin mode, position mode, risk limit, collateral assets, spot margin, convert, broker, demo funds, agreements |
| `client.V5Api.Trading` | Orders, batch orders, order history, positions, user trades, risk confirmation, trading stops, closed PnL, leverage-token orders, spread orders, order pre-checks, disconnect-cancel-all |
| `client.V5Api.SubAccount` | Subaccount creation, subaccount listing, subaccount API keys, subaccount deposit addresses |
| `client.V5Api.CryptoLoan` | Loan collateral, borrowable assets, limits, borrow, repay, open loans, completed loans, collateral adjustment |
| `client.V5Api.Earn` | Earn product info, earn orders, order history, staked positions |
| `client.V5Api.SharedApi` | SharedApis REST interfaces across spot and derivatives modes |

Do not use exchange roots from other libraries such as `SpotApi`, `UsdFuturesApi`, `CoinFuturesApi`, `PerpetualFuturesApi`, `FuturesApiV2`, `SpotApiV3`, or `ExchangeApi`.

## Socket Clients

```csharp
var socket = new BybitSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.V5SpotApi` | Spot public tickers, trades, price limits, and shared spot public streams |
| `socket.V5LinearApi` | Linear public tickers, trades, liquidations, insurance pool, price limits, ADL alerts, and shared linear public streams |
| `socket.V5InverseApi` | Inverse public tickers, trades, liquidations, insurance pool, price limits, ADL alerts, and shared inverse public streams |
| `socket.V5OptionsApi` | Options public klines, order books, tickers, and trades |
| `socket.V5SpreadApi` | Spread public tickers, trades, and order books |
| `socket.V5PrivateApi` | Private greeks, orders, positions, user trades, wallet updates, spread private updates, and disconnect-cancel-all topic |

Websocket subscription methods return `WebSocketResult<UpdateSubscription>`.

## Common Method Names

Market data commonly uses:

- `GetSpotSymbolsAsync`
- `GetSpotTickersAsync`
- `GetLinearInverseSymbolsAsync`
- `GetLinearInverseTickersAsync`
- `GetOptionSymbolsAsync`
- `GetOptionTickersAsync`
- `GetSpreadSymbolsAsync`
- `GetSpreadTickersAsync`
- `GetOrderbookAsync`
- `GetSpreadOrderBookAsync`
- `GetKlinesAsync`
- `GetIndexPriceKlinesAsync`
- `GetMarkPriceKlinesAsync`
- `GetPremiumIndexPriceKlinesAsync`
- `GetTradeHistoryAsync`
- `GetFundingRateHistoryAsync`
- `GetOpenInterestAsync`
- `GetRiskLimitAsync`
- `GetOrderPriceLimitAsync`

Account commonly uses:

- `GetBalancesAsync`
- `GetAllAssetBalancesAsync`
- `GetAssetBalanceAsync`
- `GetDepositAddressAsync`
- `GetDepositsAsync`
- `GetWithdrawalsAsync`
- `WithdrawAsync`
- `CancelWithdrawalAsync`
- `CreateInternalTransferAsync`
- `CreateUniversalTransferAsync`
- `GetFeeRateAsync`
- `SetLeverageAsync`
- `SetMarginModeAsync`
- `SwitchCrossIsolatedMarginAsync`
- `SwitchPositionModeAsync`
- `SetRiskLimitAsync`
- `SetAutoAddMarginAsync`
- `AddOrReduceMarginAsync`

Trading commonly uses:

- `GetOrdersAsync`
- `GetOrderHistoryAsync`
- `GetPositionsAsync`
- `GetUserTradesAsync`
- `PlaceOrderAsync`
- `CancelOrderAsync`
- `CancelAllOrderAsync`
- `EditOrderAsync`
- `SetTradingStopAsync`
- `PlaceMultipleOrdersAsync`
- `CancelMultipleOrdersAsync`
- `EditMultipleOrdersAsync`
- `PreCheckOrderAsync`
- `GetClosedProfitLossAsync`
- `PlaceSpreadOrderAsync`
- `CancelSpreadOrderAsync`
- `GetOpenSpreadOrdersAsync`

Sockets commonly use:

- `SubscribeToTickerUpdatesAsync`
- `SubscribeToTradeUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToOrderbookUpdatesAsync`
- `SubscribeToLiquidationUpdatesAsync`
- `SubscribeToPriceLimitUpdatesAsync`
- `SubscribeToOrderUpdatesAsync`
- `SubscribeToPositionUpdatesAsync`
- `SubscribeToUserTradeUpdatesAsync`
- `SubscribeToWalletUpdatesAsync`
- `SubscribeToGreekUpdatesAsync`
- `SubscribeToDisconnectCancelAllTopicAsync`

Confirm exact overloads from the local library source or `../Bybit.Net/Examples/ai-friendly/` before generating non-trivial code.

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.V5Api.SharedApi`
- `socketClient.V5SpotApi.SharedApi`
- `socketClient.V5LinearApi.SharedApi`
- `socketClient.V5InverseApi.SharedApi`
- `socketClient.V5PrivateApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `IBybitSharedApiClient`. It exposes:

- `Rest` as `IBybitRestClientSharedApi`
- `SpotSocket` as `IBybitSocketClientSpotSharedApi`
- `LinearSocket` as `IBybitSocketClientLinearSharedApi`
- `InverseSocket` as `IBybitSocketClientInverseSharedApi`
- `PrivateSocket` as `IBybitSocketClientPrivateSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `IBybitSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.
