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

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.ExchangeApi.SharedApi`
- `socketClient.ExchangeApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `IBitfinexSharedApiClient`. It exposes:

- `Rest` as `IBitfinexRestClientExchangeSharedApi`
- `Socket` as `IBitfinexSocketClientExchangeSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `IBitfinexSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.
