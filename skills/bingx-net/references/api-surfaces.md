# BingX.Net API Surfaces

Use this reference when choosing where a BingX endpoint belongs.

## REST Clients

```csharp
var client = new BingXRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.SpotApi.ExchangeData` | Spot symbols, tickers, klines, trades, order book, trading rules |
| `client.SpotApi.Account` | Spot balances, deposits, withdrawals, transfers, account settings |
| `client.SpotApi.Trading` | Spot order placement, cancellation, lookup, open/history orders, OCO |
| `client.PerpetualFuturesApi.ExchangeData` | Perpetual futures symbols, tickers, klines, trades, order book, funding/mark data |
| `client.PerpetualFuturesApi.Account` | Perpetual futures account state, leverage, margin mode, account settings |
| `client.PerpetualFuturesApi.Trading` | Perpetual futures order placement, cancellation, lookup, positions |
| `client.SubAccountApi` | Subaccount permissions and related account endpoints |

There is no `FuturesApi` root. Use `PerpetualFuturesApi`.

## Socket Clients

```csharp
var socket = new BingXSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.SpotApi` | Spot ticker, price, kline, trade, order book streams |
| `socket.PerpetualFuturesApi` | Perpetual futures ticker, mark price, kline, trade, order book, user streams |

## Common Method Names

Spot market data commonly uses:

- `GetTickersAsync`
- `GetKlinesAsync`
- `GetOrderBookAsync`
- `GetRecentTradesAsync`
- `GetTradingRulesAsync`

Spot trading commonly uses:

- `PlaceOrderAsync`
- `CancelOrderAsync`
- `GetOrderAsync`
- `GetOpenOrdersAsync`
- `GetOrdersAsync`
- `PlaceOcoOrderAsync`

Perpetual futures commonly uses:

- `SetLeverageAsync`
- `SetMarginModeAsync`
- `PlaceOrderAsync`
- `CancelOrderAsync`
- `GetPositionsAsync`
- `ClosePositionAsync`

Sockets commonly use:

- `SubscribeToTickerUpdatesAsync`
- `SubscribeToPriceUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToPartialOrderBookUpdatesAsync`
- `SubscribeToIncrementalOrderBookUpdatesAsync`
- `SubscribeToMarkPriceUpdatesAsync` on perpetual futures

Confirm exact overloads from the local library source or `../BingX.Net/Examples/ai-friendly/` before generating non-trivial code.

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.SpotApi.SharedApi`
- `restClient.PerpetualFuturesApi.SharedApi`
- `socketClient.SpotApi.SharedApi`
- `socketClient.PerpetualFuturesApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `IBingXSharedApiClient`. It exposes:

- `SpotRest` as `IBingXRestClientSpotSharedApi`
- `PerpetualFuturesRest` as `IBingXRestClientPerpetualFuturesSharedApi`
- `SpotSocket` as `IBingXSocketClientSpotSharedApi`
- `PerpetualFuturesSocket` as `IBingXSocketClientPerpetualFuturesSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `IBingXSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.
