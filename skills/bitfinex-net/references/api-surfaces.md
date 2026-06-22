# Bitfinex.Net API Surfaces

Use this reference when choosing where a Bitfinex endpoint belongs.

## REST Clients

```csharp
var client = new BitfinexRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.ExchangeApi.ExchangeData` | Platform status, assets, symbols, tickers, funding tickers, trade history, order books, raw books, klines, margin info, derivatives status, liquidation data, public funding statistics |
| `client.ExchangeApi.Account` | Wallet balances, base/symbol margin info, movements, ledgers, alerts, available balance, user info, 30-day summary and fees, deposit addresses, wallet transfers, withdrawals, login history, API key permissions |
| `client.ExchangeApi.Trading` | Open/closed orders, order trades, user trades, position history, order placement, order cancellation, position claim/increase, current positions, position snapshots |
| `client.GeneralApi.Funding` | Active funding offers, funding offer history, submit/cancel funding offers, loans, credits, funding trade history, funding info, auto-renew status, keep funding |

Bitfinex uses a single `ExchangeApi` root rather than product-specific roots. Spot, margin, funding-market, and derivatives market data methods are under `ExchangeApi.ExchangeData`; socket subscriptions are under `socket.ExchangeApi`.

## Socket Clients

```csharp
var socket = new BitfinexSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.ExchangeApi` | Ticker, funding ticker, order book, funding order book, raw books, trades, klines, liquidation updates, derivatives updates, authenticated user updates, websocket order commands, websocket funding commands |

Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Websocket query/command methods on `socket.ExchangeApi`, such as websocket order placement and funding offer commands, return `QueryResult<T>`.

## Common Method Names

Market data commonly uses:

- `GetTickerAsync`
- `GetFundingTickerAsync`
- `GetKlinesAsync`
- `GetOrderBookAsync`
- `GetRawOrderBookAsync`
- `GetTradeHistoryAsync`
- `GetSymbolsAsync`
- `GetFuturesSymbolsAsync`

Account and trading commonly use:

- `GetBalancesAsync`
- `GetOpenOrdersAsync`
- `GetClosedOrdersAsync`
- `PlaceOrderAsync`
- `CancelOrderAsync`
- `GetPositionsAsync`
- `GetPositionHistoryAsync`

Funding commonly uses:

- `GetActiveFundingOffersAsync`
- `SubmitFundingOfferAsync`
- `CancelFundingOfferAsync`
- `GetFundingLoansAsync`
- `GetFundingCreditsAsync`
- `GetFundingInfoAsync`

Sockets commonly use:

- `SubscribeToTickerUpdatesAsync`
- `SubscribeToFundingTickerUpdatesAsync`
- `SubscribeToOrderBookUpdatesAsync`
- `SubscribeToFundingOrderBookUpdatesAsync`
- `SubscribeToTradeUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToUserUpdatesAsync`

Confirm exact overloads from the local library source or `../Bitfinex.Net/Examples/ai-friendly/` before generating non-trivial code.

## SharedApis Surfaces

Use `.SharedClient` when writing exchange-agnostic code:

```csharp
var sharedSpot = new BitfinexRestClient().ExchangeApi.SharedClient;
var sharedSocket = new BitfinexSocketClient().ExchangeApi.SharedClient;
```

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
- `ITradeHistoryRestClient`
- `IWithdrawalRestClient`
- `IWithdrawRestClient`
- `IFeeRestClient`
- `ISpotTriggerOrderRestClient`
- `IBookTickerRestClient`
- `ITransferRestClient`

Implemented socket shared interfaces include:

- `ITickerSocketClient`
- `ITradeSocketClient`
- `IBookTickerSocketClient`
- `IBalanceSocketClient`
- `ISpotOrderSocketClient`
- `IKlineSocketClient`
- `IUserTradeSocketClient`
