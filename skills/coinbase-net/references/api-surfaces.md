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
| `client.AdvancedTradeApi.SharedApi` | SharedApis REST interfaces for spot, perpetual linear, and delivery linear workflows |
| `client.ExchangeApi.ExchangeData` | Coinbase Exchange server time, assets, and symbols |

`ExchangeApi` is real in Coinbase.Net, but it is not the primary trading API. Prefer `AdvancedTradeApi` for Coinbase Advanced Trade workflows and only use `ExchangeApi` for Coinbase Exchange market-data endpoints.

## Socket Clients

```csharp
var socket = new CoinbaseSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.AdvancedTradeApi` | Advanced Trade heartbeat, trades, klines, tickers, batched tickers, symbol updates, order book, private user updates, futures balance updates |
| `socket.AdvancedTradeApi.SharedApi` | SharedApis socket interfaces for spot and Coinbase futures/perpetual workflows |
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

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.AdvancedTradeApi.SharedApi`
- `socketClient.AdvancedTradeApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `ICoinbaseSharedApiClient`. It exposes:

- `AdvancedTradeRest` as `ICoinbaseRestClientAdvancedTradeSharedApi`
- `AdvancedTradeSocket` as `ICoinbaseSocketClientAdvancedTradeSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `ICoinbaseSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.
