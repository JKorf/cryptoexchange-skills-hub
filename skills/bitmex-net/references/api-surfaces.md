# BitMEX.Net API Surfaces

Use this reference when choosing where a BitMEX endpoint belongs.

## REST Clients

```csharp
var client = new BitMEXRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.ExchangeApi.ExchangeData` | Server time, active symbols, symbol metadata, intervals, indexes, symbol volumes, trades, klines, stats, settlements, book ticker history, order books, insurance, funding history, announcements, assets, asset networks, liquidations |
| `client.ExchangeApi.Account` | User events, account info, fees, deposit address, margin status, quote ratios, trading volume, balances, balance history, balance summary, account transfers, withdrawals, cancel withdrawal, isolated margin, risk limits, margin transfers, saved addresses, address book, API key info |
| `client.ExchangeApi.Trading` | Execution history, order placement/edit/cancel/cancel-all, cancel-all-after, user executions, user trades, positions, cross/isolated leverage |
| `client.ExchangeApi.SharedClient` | SharedApis REST interfaces across spot and derivatives modes |

Do not use exchange roots from other libraries such as `SpotApi`, `UsdFuturesApi`, `FuturesApiV2`, `SpotApiV3`, `CoinFuturesApi`, or `PerpetualFuturesApi`.

## Socket Clients

```csharp
var socket = new BitMEXSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.ExchangeApi` | Trade, kline, book ticker, aggregated book ticker, settlement, order book, incremental order book, symbol, order, balance, user trade, and position updates |
| `socket.ExchangeApi.SharedClient` | SharedApis socket interfaces |

Websocket subscription methods return `WebSocketResult<UpdateSubscription>`.

## Common Method Names

Market data commonly uses:

- `GetActiveSymbolsAsync`
- `GetSymbolsAsync`
- `GetTradesAsync`
- `GetKlinesAsync`
- `GetOrderBookAsync`
- `GetFundingHistoryAsync`
- `GetLiquidationsAsync`
- `GetAssetsAsync`

Account and trading commonly use:

- `GetAccountInfoAsync`
- `GetBalancesAsync`
- `GetMarginStatusAsync`
- `PlaceOrderAsync`
- `GetOrdersAsync`
- `EditOrderAsync`
- `CancelOrderAsync`
- `CancelAllOrdersAsync`
- `CancelAllAfterAsync`
- `GetUserTradesAsync`
- `GetPositionsAsync`
- `SetCrossMarginLeverageAsync`
- `SetIsolatedMarginLeverageAsync`

Sockets commonly use:

- `SubscribeToTradeUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToBookTickerUpdatesAsync`
- `SubscribeToOrderBookUpdatesAsync`
- `SubscribeToIncrementalOrderBookUpdatesAsync`
- `SubscribeToOrderUpdatesAsync`
- `SubscribeToBalanceUpdatesAsync`
- `SubscribeToUserTradeUpdatesAsync`
- `SubscribeToPositionUpdatesAsync`

Confirm exact overloads from the local library source or `../BitMEX.Net/Examples/ai-friendly/` before generating non-trivial code.

## SharedApis Surfaces

Use `.SharedClient` when writing exchange-agnostic code:

```csharp
var sharedRest = new BitMEXRestClient().ExchangeApi.SharedClient;
var sharedSocket = new BitMEXSocketClient().ExchangeApi.SharedClient;
```

Call `SharedClient.Discover()` before relying on optional shared features.

Implemented REST shared interfaces include:

- `IAssetsRestClient`
- `IBalanceRestClient`
- `IDepositRestClient`
- `IFeeRestClient`
- `IKlineRestClient`
- `IOrderBookRestClient`
- `IRecentTradeRestClient`
- `ITradeHistoryRestClient`
- `IWithdrawalRestClient`
- `IWithdrawRestClient`
- `ISpotSymbolRestClient`
- `ISpotTickerRestClient`
- `ISpotOrderRestClient`
- `IFundingRateRestClient`
- `IFuturesSymbolRestClient`
- `IFuturesTickerRestClient`
- `ILeverageRestClient`
- `IOpenInterestRestClient`
- `IFuturesOrderRestClient`
- `ISpotOrderClientIdRestClient`
- `IFuturesOrderClientIdRestClient`
- `IFuturesTriggerOrderRestClient`
- `ISpotTriggerOrderRestClient`
- `IFuturesTpSlRestClient`
- `IBookTickerRestClient`

Implemented socket shared interfaces include `ISpotOrderSocketClient`, `IUserTradeSocketClient`, `IBalanceSocketClient`, `IBookTickerSocketClient`, `IOrderBookSocketClient`, `ITickerSocketClient`, `ITradeSocketClient`, `IFuturesOrderSocketClient`, and `IPositionSocketClient`.

