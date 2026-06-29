# Lighter.Net API Surfaces

Use this reference when choosing where a Lighter endpoint belongs.

## REST Clients

```csharp
var client = new LighterRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.ExchangeApi.ExchangeData` | Status, system config, symbols, symbol details/tickers, order books, recent trades, klines, mark price klines, funding rates, exchange stats, announcements, assets |
| `client.ExchangeApi.Account` | API key generation, nonces, accounts, account limits, account metadata, PnL, liquidation history, funding history, deposits, transfers, withdrawals, API keys, integrator approval, leverage, margin |
| `client.ExchangeApi.Trading` | Place/edit/cancel/cancel-all orders, batch orders, open orders, closed orders, user trades |
| `client.ExchangeApi.SharedClient` | SharedApis REST interfaces for exchange-agnostic code |

## Socket Clients

```csharp
var socket = new LighterSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.ExchangeApi.ExchangeData` | Public trade, order book, book ticker, spot ticker, futures ticker, kline, and mark-price kline subscriptions |
| `socket.ExchangeApi.Account` | Account, user stats, balance subscriptions; leverage and margin websocket requests |
| `socket.ExchangeApi.Trading` | Order, user trade, position subscriptions; place/edit/cancel/cancel-all order websocket requests |
| `socket.ExchangeApi.SharedClient` | SharedApis socket interfaces for exchange-agnostic code |

## Common REST Method Names

Market data commonly uses:

- `GetStatusAsync`
- `GetSystemConfigAsync`
- `GetSymbolsAsync`
- `GetSymbolDetailsAsync`
- `GetOrderBookAsync`
- `GetRecentTradesAsync`
- `GetKlinesAsync`
- `GetFundingRatesAsync`
- `GetFundingRateHistoryAsync`

Account and transaction endpoints commonly use:

- `GetAccountsAsync`
- `GetAccountLimitsAsync`
- `GetAccountMetadataAsync`
- `GetPnlAsync`
- `GetDepositHistoryAsync`
- `GetTransferHistoryAsync`
- `GetWithdrawHistoryAsync`
- `GetApiKeysAsync`
- `SetLeverageAsync`
- `UpdateMarginAsync`

Trading commonly uses:

- `PlaceOrderAsync`
- `PlaceMultipleOrdersAsync`
- `EditOrderAsync`
- `CancelOrderAsync`
- `CancelAllOrdersAsync`
- `GetOpenOrdersAsync`
- `GetClosedOrdersAsync`
- `GetUserTradesAsync`

## Common Socket Method Names

Subscriptions commonly use:

- `SubscribeToTradeUpdatesAsync`
- `SubscribeToOrderBookUpdatesAsync`
- `SubscribeToBookTickerUpdatesAsync`
- `SubscribeToSpotTickerUpdatesAsync`
- `SubscribeToFuturesTickerUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToAccountUpdatesAsync`
- `SubscribeToBalanceUpdatesAsync`
- `SubscribeToOrderUpdatesAsync`
- `SubscribeToPositionUpdatesAsync`

Socket request methods commonly use:

- `PlaceOrderAsync`
- `PlaceMultipleOrdersAsync`
- `EditOrderAsync`
- `CancelOrderAsync`
- `CancelAllOrdersAsync`
- `SetLeverageAsync`
- `UpdateMarginAsync`

Subscription methods return `WebSocketResult<UpdateSubscription>`. Socket request methods return `QueryResult<T>`.

## SharedApis Surfaces

Use `.SharedClient` when writing exchange-agnostic code:

```csharp
var sharedRest = new LighterRestClient().ExchangeApi.SharedClient;
var sharedSocket = new LighterSocketClient().ExchangeApi.SharedClient;
```

REST shared interfaces implemented by Lighter include:

- `IKlineRestClient`
- `ISpotSymbolRestClient`
- `IFuturesSymbolRestClient`
- `ISpotTickerRestClient`
- `IFuturesTickerRestClient`
- `IBookTickerRestClient`
- `IRecentTradeRestClient`
- `IOrderBookRestClient`
- `IAssetsRestClient`
- `IDepositRestClient`
- `IWithdrawalRestClient`
- `IFeeRestClient`
- `IBalanceRestClient`
- `ISpotOrderRestClient`
- `ISpotOrderClientIdRestClient`
- `ILeverageRestClient`
- `IOpenInterestRestClient`
- `IFuturesOrderRestClient`
- `IFuturesOrderClientIdRestClient`

Socket shared interfaces implemented by Lighter include:

- `ITickerSocketClient`
- `ITickersSocketClient`
- `ITradeSocketClient`
- `IBookTickerSocketClient`
- `IKlineSocketClient`
- `IBalanceSocketClient`
- `ISpotOrderSocketClient`
- `IFuturesOrderSocketClient`
- `IUserTradeSocketClient`
- `IPositionSocketClient`

Confirm exact overloads from the local library source or `../Lighter.Net/Examples/` before generating non-trivial code.
