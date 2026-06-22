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

## SharedApis Surfaces

Use `.SharedClient` when writing exchange-agnostic code:

```csharp
var spotShared = new AsterRestClient().SpotV3Api.SharedClient;
var futuresShared = new AsterRestClient().FuturesV3Api.SharedClient;
var spotSocketShared = new AsterSocketClient().SpotV3Api.SharedClient;
var futuresSocketShared = new AsterSocketClient().FuturesV3Api.SharedClient;
```

Common shared interfaces include:

- `ISpotTickerRestClient`
- `ISpotOrderRestClient`
- `IBalanceRestClient`
- `IFuturesOrderRestClient`
- `IPositionRestClient`
- `ITickerSocketClient`
- `IOrderBookSocketClient`
