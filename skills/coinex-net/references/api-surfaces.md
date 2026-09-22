# CoinEx.Net API Surfaces

Use this reference when choosing where a CoinEx endpoint belongs.

## REST Clients

```csharp
var client = new CoinExRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.SpotApiV2.ExchangeData` | Spot server time, symbols, assets, tickers, order book, trade history, klines, index prices |
| `client.SpotApiV2.Account` | Trading fees, account config, spot/margin/financial/credit/AMM balances, margin borrow/repay, borrow history/limits, deposit addresses, deposit/withdraw history, withdrawals, transfers |
| `client.SpotApiV2.Trading` | Spot and margin orders, stop orders, batch orders, open/closed orders, edits, cancellations, client-order-id cancellations, user trades, order trades |
| `client.SpotApiV2.SharedApi` | SharedApis REST interfaces for spot workflows |
| `client.FuturesApi.ExchangeData` | Futures symbols, tickers, order books, trades, klines, index/mark prices, funding rates, open interest, premium/index data |
| `client.FuturesApi.Account` | Futures balances, trading fees, leverage |
| `client.FuturesApi.Trading` | Futures orders, stop orders, cancellations, positions, position history, close position, take profit, stop loss, margin adjustment |
| `client.FuturesApi.SharedApi` | SharedApis REST interfaces for perpetual futures workflows |

Do not use exchange roots from other libraries such as `SpotApi`, `UsdFuturesApi`, `CoinFuturesApi`, `PerpetualFuturesApi`, `FuturesApiV2`, `SpotApiV3`, `V5Api`, `ExchangeApi`, or `UnifiedApi`.

## Socket Clients

```csharp
var socket = new CoinExSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.SpotApiV2` | Spot system notices, tickers, order books, trades, index price, book price, private order/stop-order/user-trade/balance streams |
| `socket.SpotApiV2.SharedApi` | SharedApis socket interfaces for spot workflows |
| `socket.FuturesApi` | Futures tickers, order books, trades, index price, book price, premium index, private order/stop-order/user-trade/balance/position streams |
| `socket.FuturesApi.SharedApi` | SharedApis socket interfaces for perpetual futures workflows |

Websocket subscription methods return `WebSocketResult<UpdateSubscription>`.

## Common Method Names

Spot market data commonly uses:

- `GetServerTimeAsync`
- `GetSymbolsAsync`
- `GetAssetsAsync`
- `GetTickersAsync`
- `GetOrderBookAsync`
- `GetTradeHistoryAsync`
- `GetKlinesAsync`
- `GetIndexPricesAsync`

Spot account and trading commonly uses:

- `GetBalancesAsync`
- `GetMarginBalancesAsync`
- `GetFinancialBalancesAsync`
- `GetTradingFeesAsync`
- `GetDepositAddressAsync`
- `GetDepositHistoryAsync`
- `WithdrawAsync`
- `CancelWithdrawalAsync`
- `GetWithdrawalHistoryAsync`
- `TransferAsync`
- `MarginBorrowAsync`
- `MarginRepayAsync`
- `PlaceOrderAsync`
- `PlaceStopOrderAsync`
- `PlaceMultipleOrdersAsync`
- `PlaceMultipleStopOrdersAsync`
- `GetOrderAsync`
- `GetOpenOrdersAsync`
- `GetClosedOrdersAsync`
- `GetOpenStopOrdersAsync`
- `EditOrderAsync`
- `CancelOrderAsync`
- `CancelOrdersAsync`
- `CancelStopOrderAsync`
- `GetUserTradesAsync`
- `GetOrderTradesAsync`

Futures account and trading commonly uses:

- `GetBalancesAsync`
- `GetTradingFeesAsync`
- `SetLeverageAsync`
- `PlaceOrderAsync`
- `PlaceStopOrderAsync`
- `CancelOrderAsync`
- `CancelStopOrderAsync`
- `GetPositionsAsync`
- `GetPositionHistoryAsync`
- `ClosePositionAsync`
- `SetTakeProfitAsync`
- `SetStopLossAsync`
- `AdjustPositionMarginAsync`

Sockets commonly use:

- `SubscribeToTickerUpdatesAsync`
- `SubscribeToOrderBookUpdatesAsync`
- `SubscribeToTradeUpdatesAsync`
- `SubscribeToIndexPriceUpdatesAsync`
- `SubscribeToBookPriceUpdatesAsync`
- `SubscribeToPremiumIndexUpdatesAsync`
- `SubscribeToOrderUpdatesAsync`
- `SubscribeToStopOrderUpdatesAsync`
- `SubscribeToUserTradeUpdatesAsync`
- `SubscribeToBalanceUpdatesAsync`
- `SubscribeToPositionUpdatesAsync`

Confirm exact overloads from the local library source or `../CoinEx.Net/Examples/ai-friendly/` before generating non-trivial code.

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.SpotApiV2.SharedApi`
- `restClient.FuturesApi.SharedApi`
- `socketClient.SpotApiV2.SharedApi`
- `socketClient.FuturesApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `ICoinExSharedApiClient`. It exposes:

- `SpotRest` as `ICoinExRestClientSpotSharedApi`
- `FuturesRest` as `ICoinExRestClientFuturesSharedApi`
- `SpotSocket` as `ICoinExSocketClientSpotSharedApi`
- `FuturesSocket` as `ICoinExSocketClientFuturesSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `ICoinExSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.
