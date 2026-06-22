# BitMart.Net API Surfaces

Use this reference when choosing where a BitMart endpoint belongs.

## REST Clients

```csharp
var client = new BitMartRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.SpotApi.ExchangeData` | Server status/time, spot assets, symbols, symbol names, tickers, deposit/withdraw info, klines, trades, order book |
| `client.SpotApi.Account` | Funding balances, spot balances, deposit addresses, withdrawal quotas, withdrawals, deposit/withdrawal history, isolated margin accounts/transfers, fees, withdrawal addresses |
| `client.SpotApi.Margin` | Isolated margin borrow, repay, borrow/repay history, borrow info |
| `client.SpotApi.SubAccount` | Spot sub-account transfer operations, transfer history, sub-account balances, sub-account list |
| `client.SpotApi.Trading` | Spot order placement, multiple orders, cancellation, margin order placement, order lookup, client-order lookup, open/closed orders, user trades, order trades |
| `client.UsdFuturesApi.ExchangeData` | Contracts, futures order book, open interest, current/historical funding rate, klines, mark klines, leverage brackets, recent trades |
| `client.UsdFuturesApi.Account` | Futures balances, transfer history, spot/futures transfer, leverage, symbol fee rate, transaction history, position mode |
| `client.UsdFuturesApi.SubAccount` | Futures sub-account transfers, balances, transfer history |
| `client.UsdFuturesApi.Trading` | Futures order lookup, open/closed orders, positions, position risk, user trades, order placement/cancel/edit, trigger orders, trailing orders, TP/SL, cancel-all-after |

Do not use Binance/Bitget-style roots such as `SpotApiV3`, `FuturesApiV2`, `UsdFuturesApiV3`, `CoinFuturesApi`, or `PerpetualFuturesApi`.

## Socket Clients

```csharp
var socket = new BitMartSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.SpotApi` | Spot ticker, kline, partial and incremental order book, trade, order, book ticker, balance streams |
| `socket.UsdFuturesApi` | USD futures ticker/tickers, trade, kline, order book, order, balance, position streams |

Websocket subscription methods return `WebSocketResult<UpdateSubscription>`.

## Common Method Names

Spot market data commonly uses:

- `GetTickerAsync`
- `GetTickersAsync`
- `GetSymbolsAsync`
- `GetSymbolNamesAsync`
- `GetKlinesAsync`
- `GetTradesAsync`
- `GetOrderBookAsync`

Spot account, margin, and trading commonly use:

- `GetSpotBalancesAsync`
- `GetFundingBalancesAsync`
- `GetDepositAddressAsync`
- `WithdrawAsync`
- `BorrowAsync`
- `RepayAsync`
- `PlaceOrderAsync`
- `PlaceMarginOrderAsync`
- `CancelOrderAsync`
- `GetOpenOrdersAsync`
- `GetClosedOrdersAsync`
- `GetUserTradesAsync`

USD futures commonly uses:

- `GetContractsAsync`
- `GetCurrentFundingRateAsync`
- `GetBalancesAsync`
- `SetLeverageAsync`
- `SetPositionModeAsync`
- `GetPositionsAsync`
- `PlaceOrderAsync`
- `CancelOrderAsync`
- `PlaceTriggerOrderAsync`
- `PlaceTpSlOrderAsync`

Sockets commonly use:

- `SubscribeToTickerUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToOrderBookUpdatesAsync`
- `SubscribeToPartialOrderBookUpdatesAsync`
- `SubscribeToTradeUpdatesAsync`
- `SubscribeToOrderUpdatesAsync`
- `SubscribeToBalanceUpdatesAsync`
- `SubscribeToPositionUpdatesAsync`

Confirm exact overloads from the local library source or `../BitMart.Net/Examples/ai-friendly/` before generating non-trivial code.

## SharedApis Surfaces

Use `.SharedClient` when writing exchange-agnostic code:

```csharp
var sharedSpot = new BitMartRestClient().SpotApi.SharedClient;
var sharedFutures = new BitMartRestClient().UsdFuturesApi.SharedClient;
var sharedSpotSocket = new BitMartSocketClient().SpotApi.SharedClient;
var sharedFuturesSocket = new BitMartSocketClient().UsdFuturesApi.SharedClient;
```

Call `SharedClient.Discover()` before relying on optional shared features.

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
- `IBookTickerRestClient`

Implemented USD futures REST shared interfaces include:

- `IBalanceRestClient`
- `IFuturesTickerRestClient`
- `IFuturesSymbolRestClient`
- `IKlineRestClient`
- `IRecentTradeRestClient`
- `ILeverageRestClient`
- `IOrderBookRestClient`
- `IOpenInterestRestClient`
- `IFuturesOrderRestClient`
- `IFeeRestClient`
- `IFuturesOrderClientIdRestClient`
- `IFuturesTriggerOrderRestClient`
- `IFuturesTpSlRestClient`
- `IBookTickerRestClient`
- `IPositionModeRestClient`

Implemented spot socket shared interfaces include `ITickerSocketClient`, `ITradeSocketClient`, `IBookTickerSocketClient`, `IBalanceSocketClient`, `ISpotOrderSocketClient`, `IKlineSocketClient`, and `IOrderBookSocketClient`.

Implemented USD futures socket shared interfaces include `ITickersSocketClient`, `ITickerSocketClient`, `ITradeSocketClient`, `IBookTickerSocketClient`, `IBalanceSocketClient`, `IKlineSocketClient`, `IFuturesOrderSocketClient`, `IPositionSocketClient`, and `IOrderBookSocketClient`.

