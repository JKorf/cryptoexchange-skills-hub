# Aster.Net API Surfaces

Use this reference when choosing where an Aster endpoint belongs.

## REST Clients

```csharp
var client = new AsterRestClient();
```

Use V3 by default.

| Surface | Use for |
| --- | --- |
| `client.SpotV3Api.ExchangeData` | Spot V3 symbols, tickers, klines, trades, order book, exchange info |
| `client.SpotV3Api.Account` | Spot V3 account info, balances, user streams, transfers, commission rates |
| `client.SpotV3Api.Trading` | Spot V3 order placement, cancellation, lookup, open/closed orders |
| `client.FuturesV3Api.ExchangeData` | Futures V3 symbols, tickers, klines, trades, order book, exchange info |
| `client.FuturesV3Api.Account` | Futures V3 account state, leverage, margin type, account settings |
| `client.FuturesV3Api.Trading` | Futures V3 order placement, cancellation, lookup, positions |

Compatibility surfaces:

| Surface | Use for |
| --- | --- |
| `client.SpotApi` | Spot V1 compatibility endpoints |
| `client.FuturesApi` | Futures V1 compatibility endpoints |

Do not use V1 surfaces unless the user explicitly asks for V1 or the target codebase already uses V1.

## Socket Clients

```csharp
var socket = new AsterSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.SpotV3Api` | Spot V3 ticker, kline, trade, order book, book ticker streams |
| `socket.FuturesV3Api` | Futures V3 ticker, kline, mark price, trade, order book streams |
| `socket.SpotApi` | Spot V1 compatibility streams |
| `socket.FuturesApi` | Futures V1 compatibility streams |

## Common Method Names

Market data commonly uses:

- `GetTickerAsync`
- `GetKlinesAsync`
- `GetOrderBookAsync`
- `GetRecentTradesAsync`
- `GetExchangeInfoAsync`

Spot trading commonly uses:

- `PlaceOrderAsync`
- `CancelOrderAsync`
- `GetOrderAsync`
- `GetOpenOrdersAsync`

Futures trading commonly uses:

- `SetLeverageAsync`
- `SetMarginTypeAsync`
- `PlaceOrderAsync`
- `CancelOrderAsync`
- `GetPositionsAsync`

Sockets commonly use:

- `SubscribeToTickerUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToOrderBookUpdatesAsync`
- `SubscribeToPartialOrderBookUpdatesAsync`
- `SubscribeToMarkPriceUpdatesAsync` on futures

Confirm exact overloads from the local library source or `../Aster.Net/Examples/ai-friendly/` before generating non-trivial code.

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.SpotApi.SharedApi`
- `restClient.FuturesApi.SharedApi`
- `socketClient.SpotApi.SharedApi`
- `socketClient.FuturesApi.SharedApi`
- `restClient.SpotV3Api.SharedApi`
- `restClient.FuturesV3Api.SharedApi`
- `socketClient.SpotV3Api.SharedApi`
- `socketClient.FuturesV3Api.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `IAsterSharedApiClient`. It exposes:

- `SpotRest` as `IAsterRestClientSpotSharedApi`
- `FuturesRest` as `IAsterRestClientFuturesSharedApi`
- `SpotV3Rest` as `IAsterRestClientSpotV3SharedApi`
- `FuturesV3Rest` as `IAsterRestClientFuturesV3SharedApi`
- `SpotSocket` as `IAsterSocketClientSpotSharedApi`
- `FuturesSocket` as `IAsterSocketClientFuturesSharedApi`
- `SpotV3Socket` as `IAsterSocketClientSpotV3SharedApi`
- `FuturesV3Socket` as `IAsterSocketClientFuturesV3SharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `IAsterSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.

## Futures V3 Strategy Orders

`client.FuturesV3Api.Trading` includes chase and multi-leg strategy workflows:

- `PlaceChaseOrderAsync(...)`
- `PlaceStrategyOrderAsync(StrategyType, IEnumerable<AsterStrategyOrderRequest>, ...)`
- `EditStrategyOrderAsync(...)`
- `GetOpenStrategyOrderAsync(...)` and `GetClosedStrategyOrderAsync(...)`

Strategy operations can create or change futures exposure. Read the safety reference and inspect nested order results rather than treating outer success as proof that every leg succeeded.
