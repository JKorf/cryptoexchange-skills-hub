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
| `client.SpotApiV2.SharedClient` | SharedApis REST interfaces for spot workflows |
| `client.FuturesApi.ExchangeData` | Futures symbols, tickers, order books, trades, klines, index/mark prices, funding rates, open interest, premium/index data |
| `client.FuturesApi.Account` | Futures balances, trading fees, leverage |
| `client.FuturesApi.Trading` | Futures orders, stop orders, cancellations, positions, position history, close position, take profit, stop loss, margin adjustment |
| `client.FuturesApi.SharedClient` | SharedApis REST interfaces for perpetual futures workflows |

Do not use exchange roots from other libraries such as `SpotApi`, `UsdFuturesApi`, `CoinFuturesApi`, `PerpetualFuturesApi`, `FuturesApiV2`, `SpotApiV3`, `V5Api`, `ExchangeApi`, or `UnifiedApi`.

## Socket Clients

```csharp
var socket = new CoinExSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.SpotApiV2` | Spot system notices, tickers, order books, trades, index price, book price, private order/stop-order/user-trade/balance streams |
| `socket.SpotApiV2.SharedClient` | SharedApis socket interfaces for spot workflows |
| `socket.FuturesApi` | Futures tickers, order books, trades, index price, book price, premium index, private order/stop-order/user-trade/balance/position streams |
| `socket.FuturesApi.SharedClient` | SharedApis socket interfaces for perpetual futures workflows |

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

## SharedApis Surfaces

Use `.SharedClient` when writing exchange-agnostic code:

```csharp
var spotRest = new CoinExRestClient().SpotApiV2.SharedClient;
var futuresRest = new CoinExRestClient().FuturesApi.SharedClient;
var spotSocket = new CoinExSocketClient().SpotApiV2.SharedClient;
var futuresSocket = new CoinExSocketClient().FuturesApi.SharedClient;
```

Call `SharedClient.Discover()` before relying on optional shared features.

Spot shared clients support `TradingMode.Spot`. Futures shared clients support `TradingMode.PerpetualLinear` and `TradingMode.PerpetualInverse`.

Implemented spot REST shared interfaces include:

- `IAssetsRestClient`
- `IBalanceRestClient`
- `IDepositRestClient`
- `IKlineRestClient`
- `IOrderBookRestClient`
- `IRecentTradeRestClient`
- `ISpotOrderRestClient`
- `ISpotSymbolRestClient`
- `ISpotTickerRestClient`
- `IWithdrawalRestClient`
- `IWithdrawRestClient`
- `IFeeRestClient`
- `ISpotOrderClientIdRestClient`
- `ISpotTriggerOrderRestClient`
- `IBookTickerRestClient`
- `ITransferRestClient`

Implemented futures REST shared interfaces include:

- `IBalanceRestClient`
- `IFuturesTickerRestClient`
- `IFuturesSymbolRestClient`
- `IFuturesOrderRestClient`
- `IKlineRestClient`
- `IMarkPriceKlineRestClient`
- `IIndexPriceKlineRestClient`
- `IRecentTradeRestClient`
- `ILeverageRestClient`
- `IOrderBookRestClient`
- `IOpenInterestRestClient`
- `IFundingRateRestClient`
- `IPositionHistoryRestClient`
- `IFeeRestClient`
- `IFuturesOrderClientIdRestClient`
- `IFuturesTriggerOrderRestClient`
- `IFuturesTpSlRestClient`
- `IBookTickerRestClient`

Implemented spot socket shared interfaces include `ITickerSocketClient`, `ITickersSocketClient`, `ITradeSocketClient`, `IBookTickerSocketClient`, `IOrderBookSocketClient`, `IBalanceSocketClient`, `ISpotOrderSocketClient`, and `IUserTradeSocketClient`.

Implemented futures socket shared interfaces include `ITickerSocketClient`, `ITickersSocketClient`, `ITradeSocketClient`, `IBookTickerSocketClient`, `IOrderBookSocketClient`, `IBalanceSocketClient`, `IFuturesOrderSocketClient`, `IUserTradeSocketClient`, and `IPositionSocketClient`.
