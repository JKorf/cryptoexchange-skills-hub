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

## SharedApis Surfaces

Use `.SharedClient` when writing exchange-agnostic code:

```csharp
var sharedSpot = new BinanceRestClient().SpotApi.SharedClient;
var sharedSocket = new BinanceSocketClient().SpotApi.SharedClient;
```

Common shared interfaces include:

- `ISpotTickerRestClient`
- `ISpotOrderRestClient`
- `IBalanceRestClient`
- `IFuturesOrderRestClient`
- `IPositionRestClient`
- `ITickerSocketClient`
- `IOrderBookSocketClient`
