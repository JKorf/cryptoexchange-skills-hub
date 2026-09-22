# Binance.Net API Surfaces

Use this reference when choosing where a Binance endpoint belongs.

## REST Clients

```csharp
var client = new BinanceRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.SpotApi.ExchangeData` | Spot symbols, tickers, klines, trades, order book, exchange info |
| `client.SpotApi.Account` | Spot account info, balances, user data listen keys |
| `client.SpotApi.Trading` | Spot order placement, cancellation, lookup, OCO/order-list style operations |
| `client.UsdFuturesApi.ExchangeData` | USD-M futures symbols, tickers, klines, trades, order book, exchange info |
| `client.UsdFuturesApi.Account` | USD-M futures account state, positions, leverage, margin type |
| `client.UsdFuturesApi.Trading` | USD-M futures order placement, cancellation, lookup |
| `client.CoinFuturesApi.ExchangeData` | COIN-M futures symbols, tickers, klines, trades, order book, exchange info |
| `client.CoinFuturesApi.Account` | COIN-M futures account state, positions, leverage, margin type |
| `client.CoinFuturesApi.Trading` | COIN-M futures order placement, cancellation, lookup |

## Socket Clients

```csharp
var socket = new BinanceSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.SpotApi.ExchangeData` | Spot ticker, kline, trade, order book, book ticker streams |
| `socket.SpotApi.Account` | Spot user data streams: order updates, balance updates |
| `socket.UsdFuturesApi.ExchangeData` | USD-M futures ticker, kline, trade, order book streams |
| `socket.UsdFuturesApi.Account` | USD-M futures user data streams: orders, account updates, positions |
| `socket.CoinFuturesApi.ExchangeData` | COIN-M futures ticker, kline, trade, order book streams |
| `socket.CoinFuturesApi.Account` | COIN-M futures user data streams |

## Common Method Names

Market data commonly uses:

- `GetTickerAsync`
- `GetKlinesAsync`
- `GetOrderBookAsync`
- `GetRecentTradesAsync`
- `GetExchangeInfoAsync`

Trading commonly uses:

- `PlaceOrderAsync`
- `CancelOrderAsync`
- `GetOrderAsync`
- `GetOpenOrdersAsync`

Sockets commonly use:

- `SubscribeToTickerUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToPartialOrderBookUpdatesAsync`
- `SubscribeToAggregatedTradeUpdatesAsync`
- `SubscribeToUserDataUpdatesAsync`

Confirm exact overloads from the local library source or `../Binance.Net/Examples/ai-friendly/` before generating non-trivial code.

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.SpotApi.SharedApi`
- `restClient.UsdFuturesApi.SharedApi`
- `restClient.CoinFuturesApi.SharedApi`
- `socketClient.SpotApi.SharedApi`
- `socketClient.UsdFuturesApi.SharedApi`
- `socketClient.CoinFuturesApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `IBinanceSharedApiClient`. It exposes:

- `SpotRest` as `IBinanceRestClientSpotSharedApi`
- `UsdFuturesRest` as `IBinanceRestClientUsdFuturesSharedApi`
- `CoinFuturesRest` as `IBinanceRestClientCoinFuturesSharedApi`
- `SpotSocket` as `IBinanceSocketClientSpotSharedApi`
- `UsdFuturesSocket` as `IBinanceSocketClientUsdFuturesSharedApi`
- `CoinFuturesSocket` as `IBinanceSocketClientCoinFuturesSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `IBinanceSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.
