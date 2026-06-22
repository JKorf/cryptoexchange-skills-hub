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

## SharedApis Surfaces

Use `.SharedClient` when writing exchange-agnostic code:

```csharp
var spotShared = new BingXRestClient().SpotApi.SharedClient;
var futuresShared = new BingXRestClient().PerpetualFuturesApi.SharedClient;
var spotSocketShared = new BingXSocketClient().SpotApi.SharedClient;
var futuresSocketShared = new BingXSocketClient().PerpetualFuturesApi.SharedClient;
```

Common shared interfaces include:

- `ISpotTickerRestClient`
- `ISpotOrderRestClient`
- `IBalanceRestClient`
- `IFuturesOrderRestClient`
- `IPositionRestClient`
- `ITickerSocketClient`
- `IOrderBookSocketClient`
