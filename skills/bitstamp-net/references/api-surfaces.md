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
| `client.ExchangeApi.SharedApi` | SharedApis REST interfaces across spot and derivative modes |

Do not use exchange roots from other libraries such as `SpotApi`, `UsdFuturesApi`, `FuturesApiV2`, `SpotApiV3`, `CoinFuturesApi`, or `PerpetualFuturesApi`.

## Socket Clients

```csharp
var socket = new BitstampSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.ExchangeApi` | Spot trade updates, full order book updates, order book snapshots, derivative funding rate updates, private order updates, private user trade updates |
| `socket.ExchangeApi.SharedApi` | SharedApis spot socket trade and order book interfaces |

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

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.ExchangeApi.SharedApi`
- `socketClient.ExchangeApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `IBitstampSharedApiClient`. It exposes:

- `Rest` as `IBitstampRestClientExchangeSharedApi`
- `Socket` as `IBitstampSocketClientExchangeSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `IBitstampSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.
