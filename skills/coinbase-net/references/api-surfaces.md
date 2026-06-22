# Coinbase.Net API Surfaces

Use this reference when choosing where a Coinbase endpoint belongs.

## REST Clients

```csharp
var client = new CoinbaseRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.AdvancedTradeApi.ExchangeData` | Advanced Trade server time, products, product metadata, order books, klines, trade history, book tickers, fiat/crypto assets, exchange rates, buy/sell/spot prices |
| `client.AdvancedTradeApi.Account` | Accounts, portfolios, portfolio transfers and edits, perpetual/futures balances and settings, fee info, API key info, payment methods, converts, withdrawals, deposits, transactions, deposit addresses |
| `client.AdvancedTradeApi.Trading` | Order placement, cancellation, batch cancellation, order editing, order lookup, open/closed order lists, fills, close position, futures positions, perpetual positions |
| `client.AdvancedTradeApi.SharedClient` | SharedApis REST interfaces for spot, perpetual linear, and delivery linear workflows |
| `client.ExchangeApi.ExchangeData` | Coinbase Exchange server time, assets, and symbols |

`ExchangeApi` is real in Coinbase.Net, but it is not the primary trading API. Prefer `AdvancedTradeApi` for Coinbase Advanced Trade workflows and only use `ExchangeApi` for Coinbase Exchange market-data endpoints.

## Socket Clients

```csharp
var socket = new CoinbaseSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.AdvancedTradeApi` | Advanced Trade heartbeat, trades, klines, tickers, batched tickers, symbol updates, order book, private user updates, futures balance updates |
| `socket.AdvancedTradeApi.SharedClient` | SharedApis socket interfaces for spot and Coinbase futures/perpetual workflows |
| `socket.ExchangeApi` | Coinbase Exchange heartbeat, exchange info, ticker, batched ticker, and order book streams |

Websocket subscription methods return `WebSocketResult<UpdateSubscription>`.

## Common Method Names

Advanced Trade market data commonly uses:

- `GetServerTimeAsync`
- `GetSymbolsAsync`
- `GetSymbolAsync`
- `GetOrderBookAsync`
- `GetKlinesAsync`
- `GetTradeHistoryAsync`
- `GetBookTickerAsync`
- `GetBookTickersAsync`
- `GetFiatAssetsAsync`
- `GetCryptoAssetsAsync`
- `GetExchangeRatesAsync`
- `GetBuyPriceAsync`
- `GetSellPriceAsync`
- `GetSpotPriceAsync`

Advanced Trade account commonly uses:

- `GetAccountsAsync`
- `GetAccountAsync`
- `GetPortfoliosAsync`
- `GetPortfolioAsync`
- `TransferPortfolioFundsAsync`
- `GetPerpetualPortfolioSummaryAsync`
- `GetPerpetualBalancesAsync`
- `SetPerpetualMultiAssetCollateralModeAsync`
- `GetFuturesBalanceSummaryAsync`
- `GetFeeInfoAsync`
- `GetApiKeyInfoAsync`
- `GetPaymentMethodsAsync`
- `CreateConvertQuoteAsync`
- `CommitConvertTradeAsync`
- `GetWithdrawalsAsync`
- `WithdrawAsync`
- `WithdrawCryptoAsync`
- `DepositAsync`
- `GetDepositsAsync`
- `GetTransactionsAsync`
- `CreateDepositAddressAsync`

Advanced Trade trading commonly uses:

- `PlaceOrderAsync`
- `CancelOrderAsync`
- `CancelOrdersAsync`
- `EditOrderAsync`
- `GetOrderAsync`
- `GetOrdersAsync`
- `GetUserTradesAsync`
- `ClosePositionAsync`
- `GetFuturesPositionsAsync`
- `GetFuturesPositionAsync`
- `GetPerpetualPositionsAsync`
- `GetPerpetualPositionAsync`

Advanced Trade sockets commonly use:

- `SubscribeToHeartbeatUpdatesAsync`
- `SubscribeToTradeUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToTickerUpdatesAsync`
- `SubscribeToBatchedTickerUpdatesAsync`
- `SubscribeToSymbolUpdatesAsync`
- `SubscribeToOrderBookUpdatesAsync`
- `SubscribeToUserUpdatesAsync`
- `SubscribeToFuturesBalanceUpdatesAsync`

Exchange API market data commonly uses:

- `ExchangeApi.ExchangeData.GetServerTimeAsync`
- `ExchangeApi.ExchangeData.GetAssetsAsync`
- `ExchangeApi.ExchangeData.GetSymbolsAsync`

Exchange API sockets commonly use:

- `ExchangeApi.SubscribeToHeartbeatUpdatesAsync`
- `ExchangeApi.SubscribeToExchangeInfoUpdatesAsync`
- `ExchangeApi.SubscribeToTickerUpdatesAsync`
- `ExchangeApi.SubscribeToBatchedTickerUpdatesAsync`
- `ExchangeApi.SubscribeToOrderBookUpdatesAsync`

Confirm exact overloads from the local library source or `../Coinbase.Net/Examples/ai-friendly/` before generating non-trivial code.

## SharedApis Surfaces

Use `.SharedClient` when writing exchange-agnostic code:

```csharp
var sharedRest = new CoinbaseRestClient().AdvancedTradeApi.SharedClient;
var sharedSocket = new CoinbaseSocketClient().AdvancedTradeApi.SharedClient;
```

Call `SharedClient.Discover()` before relying on optional shared features.

Supported shared trading modes are `TradingMode.Spot`, `TradingMode.PerpetualLinear`, and `TradingMode.DeliveryLinear`.

Implemented REST shared interfaces include:

- `IAssetsRestClient`
- `IBalanceRestClient`
- `IDepositRestClient`
- `IOrderBookRestClient`
- `IRecentTradeRestClient`
- `ITradeHistoryRestClient`
- `IWithdrawalRestClient`
- `IWithdrawRestClient`
- `ISpotSymbolRestClient`
- `ISpotTickerRestClient`
- `ISpotOrderRestClient`
- `IFuturesSymbolRestClient`
- `IFuturesTickerRestClient`
- `IOpenInterestRestClient`
- `IFuturesOrderRestClient`
- `IKlineRestClient`
- `IFeeRestClient`
- `ISpotTriggerOrderRestClient`
- `IFuturesTriggerOrderRestClient`
- `IBookTickerRestClient`

Implemented socket shared interfaces are:

- `IKlineSocketClient`
- `ITickerSocketClient`
- `ITradeSocketClient`
- `ISpotOrderSocketClient`
- `IFuturesOrderSocketClient`
- `IPositionSocketClient`
