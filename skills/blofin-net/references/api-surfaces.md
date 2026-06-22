# BloFin.Net API Surfaces

Use this reference when choosing where a BloFin endpoint belongs.

## REST Clients

```csharp
var client = new BloFinRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.AccountApi` | General account balances, transfers, account config, API key info, withdrawal history, deposit history |
| `client.AccountApi.SharedClient` | SharedApis account REST interfaces for withdrawals and deposits |
| `client.FuturesApi.ExchangeData` | Futures symbols, tickers, order book, recent trades, index/mark price, funding rates/history, klines, index/mark klines, position tiers |
| `client.FuturesApi.Account` | Futures balances, margin mode, position mode, leverage reads and updates |
| `client.FuturesApi.Trading` | Futures positions, order placement, batch orders, TP/SL orders, trigger orders, cancellations, open/closed orders, close position, order lookup, user trades, price limits, position history |
| `client.FuturesApi.SharedClient` | SharedApis REST interfaces for futures workflows |

Do not use exchange roots from other libraries such as `ExchangeApi`, `SpotApi`, `UsdFuturesApi`, `FuturesApiV2`, `SpotApiV3`, `CoinFuturesApi`, or `PerpetualFuturesApi`.

## Socket Clients

```csharp
var socket = new BloFinSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.FuturesApi` | Futures trades, klines, index price klines, mark price klines, order book, tickers, funding rates, private positions, private orders, private trigger orders, private balances |
| `socket.FuturesApi.SharedClient` | SharedApis futures socket interfaces |

Websocket subscription methods return `WebSocketResult<UpdateSubscription>`.

## Common Method Names

General account commonly uses:

- `GetAccountBalancesAsync`
- `TransferAsync`
- `GetAccountConfigAsync`
- `GetApiKeyInfoAsync`
- `GetTransferHistoryAsync`
- `GetWithdrawalHistoryAsync`
- `GetDepositHistoryAsync`

Futures market data commonly uses:

- `GetSymbolsAsync`
- `GetTickersAsync`
- `GetOrderBookAsync`
- `GetRecentTradesAsync`
- `GetIndexMarkPriceAsync`
- `GetFundingRateAsync`
- `GetFundingRateHistoryAsync`
- `GetKlinesAsync`
- `GetIndexPriceKlinesAsync`
- `GetMarkPriceKlinesAsync`
- `GetPositionTiersAsync`

Futures account and trading commonly uses:

- `GetBalancesAsync`
- `GetMarginModeAsync`
- `SetMarginModeAsync`
- `GetPositionModeAsync`
- `SetPositionModeAsync`
- `GetLeverageAsync`
- `SetLeverageAsync`
- `GetPositionsAsync`
- `PlaceOrderAsync`
- `PlaceMultipleOrdersAsync`
- `PlaceTpSlOrderAsync`
- `PlaceTriggerOrderAsync`
- `CancelOrderAsync`
- `CancelOrdersAsync`
- `CancelTpSlOrdersAsync`
- `CancelTriggerOrderAsync`
- `GetOpenOrdersAsync`
- `GetOpenTpSlOrdersAsync`
- `GetOpenTriggerOrdersAsync`
- `ClosePositionAsync`
- `GetOrderAsync`
- `GetTpSlOrderAsync`
- `GetClosedOrdersAsync`
- `GetClosedTpSlOrdersAsync`
- `GetClosedTriggerOrdersAsync`
- `GetUserTradesAsync`
- `GetPriceLimitsAsync`
- `GetPositionHistoryAsync`

Sockets commonly use:

- `SubscribeToTradeUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToIndexPriceKlineUpdatesAsync`
- `SubscribeToMarkPriceKlineUpdatesAsync`
- `SubscribeToOrderBookUpdatesAsync`
- `SubscribeToTickerUpdatesAsync`
- `SubscribeToFundingRateUpdatesAsync`
- `SubscribeToPositionUpdatesAsync`
- `SubscribeToOrderUpdatesAsync`
- `SubscribeToTriggerOrderUpdatesAsync`
- `SubscribeToBalanceUpdatesAsync`
- `SubscribeToInverseBalanceUpdatesAsync`

Confirm exact overloads from the local library source or `../BloFin.Net/Examples/ai-friendly/` before generating non-trivial code.

## SharedApis Surfaces

Use `.SharedClient` when writing exchange-agnostic code:

```csharp
var accountShared = new BloFinRestClient().AccountApi.SharedClient;
var futuresShared = new BloFinRestClient().FuturesApi.SharedClient;
var socketShared = new BloFinSocketClient().FuturesApi.SharedClient;
```

Call `SharedClient.Discover()` before relying on optional shared features.

Implemented account REST shared interfaces are:

- `IWithdrawalRestClient`
- `IDepositRestClient`

Implemented futures REST shared interfaces include:

- `IBookTickerRestClient`
- `IKlineRestClient`
- `IOrderBookRestClient`
- `IRecentTradeRestClient`
- `IFundingRateRestClient`
- `IFuturesSymbolRestClient`
- `IFuturesTickerRestClient`
- `IIndexPriceKlineRestClient`
- `IMarkPriceKlineRestClient`
- `IBalanceRestClient`
- `IPositionModeRestClient`
- `ILeverageRestClient`
- `IFuturesOrderRestClient`
- `IFuturesOrderClientIdRestClient`
- `IFuturesTpSlRestClient`
- `IFuturesTriggerOrderRestClient`
- `IPositionHistoryRestClient`

Implemented futures socket shared interfaces are:

- `IBookTickerSocketClient`
- `IKlineSocketClient`
- `IOrderBookSocketClient`
- `ITickerSocketClient`
- `ITradeSocketClient`
- `IBalanceSocketClient`
- `IFuturesOrderSocketClient`
- `IPositionSocketClient`
