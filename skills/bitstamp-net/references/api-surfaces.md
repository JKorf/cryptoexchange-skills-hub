# Bitstamp.Net API Surfaces

Use this reference when choosing where a Bitstamp endpoint belongs.

## REST Clients

```csharp
var client = new BitstampRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.ExchangeApi.ExchangeData` | Spot and derivative symbols, assets, all tickers, single ticker, hour ticker, klines, order book, trades, EUR/USD conversion, funding rates, funding history, margin tiers, collateral assets |
| `client.ExchangeApi.Account` | Account balances, withdrawal fees, trading fees, user transactions, account tradable symbols, max trade quantity, withdrawals, deposits, margin info, leverage settings |
| `client.ExchangeApi.Trading` | Limit/market orders, cancellation, order history, order lookup, replace order, open orders, derivative user trades, open positions, position status/history, close positions, settlement transactions, collateral updates |
| `client.ExchangeApi.SharedClient` | SharedApis REST interfaces across spot and derivative modes |

Do not use exchange roots from other libraries such as `SpotApi`, `UsdFuturesApi`, `FuturesApiV2`, `SpotApiV3`, `CoinFuturesApi`, or `PerpetualFuturesApi`.

## Socket Clients

```csharp
var socket = new BitstampSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.ExchangeApi` | Spot trade updates, full order book updates, order book snapshots, derivative funding rate updates, private order updates, private user trade updates |
| `socket.ExchangeApi.SharedClient` | SharedApis spot socket trade and order book interfaces |

Websocket subscription methods return `WebSocketResult<UpdateSubscription>`.

## Common Method Names

Market data commonly uses:

- `GetSymbolsAsync`
- `GetAssetsAsync`
- `GetAllTickersAsync`
- `GetTickerAsync`
- `GetKlinesAsync`
- `GetOrderBookAsync`
- `GetTradesAsync`
- `GetFundingRateAsync`
- `GetMarginTiersAsync`

Account and trading commonly uses:

- `GetAccountBalancesAsync`
- `GetAccountBalanceAsync`
- `GetAllFeesAsync`
- `GetFeesAsync`
- `GetUserTransactionsAsync`
- `GetLeverageSettingsAsync`
- `SetLeverageAsync`
- `PlaceLimitOrderAsync`
- `PlaceMarketOrderAsync`
- `CancelOrderAsync`
- `GetOpenOrdersAsync`
- `GetOpenPositionsAsync`
- `ClosePositionsAsync`
- `UpdatePositionCollateralAsync`

Sockets commonly use:

- `SubscribeToTradeUpdatesAsync`
- `SubscribeToFullOrderBookUpdatesAsync`
- `SubscribeToOrderBookSnapshotUpdatesAsync`
- `SubscribeToFundingRateUpdatesAsync`
- `SubscribeToOrderUpdatesAsync`
- `SubscribeToUserTradeUpdatesAsync`

Confirm exact overloads from the local library source or `../Bitstamp.Net/Examples/ai-friendly/` before generating non-trivial code.

## SharedApis Surfaces

Use `.SharedClient` when writing exchange-agnostic code:

```csharp
var sharedRest = new BitstampRestClient().ExchangeApi.SharedClient;
var sharedSocket = new BitstampSocketClient().ExchangeApi.SharedClient;
```

Call `SharedClient.Discover()` before relying on optional shared features.

Implemented REST shared interfaces include:

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
- `IFundingRateRestClient`
- `IFuturesSymbolRestClient`
- `IFuturesTickerRestClient`
- `ILeverageRestClient`
- `IOpenInterestRestClient`
- `IPositionHistoryRestClient`
- `IFuturesOrderRestClient`

REST shared clients support `TradingMode.Spot` and `TradingMode.PerpetualLinear`. Socket shared clients support `TradingMode.Spot`.

Implemented socket shared interfaces are `ITradeSocketClient` and `IOrderBookSocketClient`.
