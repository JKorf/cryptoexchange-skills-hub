# BitMEX.Net API Surfaces

Use this reference when choosing where a BitMEX endpoint belongs.

## REST Clients

```csharp
var client = new BitMEXRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.ExchangeApi.ExchangeData` | Server time, active symbols, symbol metadata, intervals, indexes, symbol volumes, trades, klines, stats, settlements, book ticker history, order books, insurance, funding history, announcements, assets, asset networks, liquidations |
| `client.ExchangeApi.Account` | User events, account info, fees, deposit address, margin status, quote ratios, trading volume, balances, balance history, balance summary, account transfers, withdrawals, cancel withdrawal, isolated margin, risk limits, margin transfers, saved addresses, address book, API key info |
| `client.ExchangeApi.Trading` | Execution history, order placement/edit/cancel/cancel-all, cancel-all-after, user executions, user trades, positions, cross/isolated leverage |
| `client.ExchangeApi.SharedApi` | SharedApis REST interfaces across spot and derivatives modes |

Do not use exchange roots from other libraries such as `SpotApi`, `UsdFuturesApi`, `FuturesApiV2`, `SpotApiV3`, `CoinFuturesApi`, or `PerpetualFuturesApi`.

## Socket Clients

```csharp
var socket = new BitMEXSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.ExchangeApi` | Trade, kline, book ticker, aggregated book ticker, settlement, order book, incremental order book, symbol, order, balance, user trade, and position updates |
| `socket.ExchangeApi.SharedApi` | SharedApis socket interfaces |

Websocket subscription methods return `WebSocketResult<UpdateSubscription>`.

## Common Method Names

Market data commonly uses:

- `GetActiveSymbolsAsync`
- `GetSymbolsAsync`
- `GetTradesAsync`
- `GetKlinesAsync`
- `GetOrderBookAsync`
- `GetFundingHistoryAsync`
- `GetLiquidationsAsync`
- `GetAssetsAsync`

Account and trading commonly use:

- `GetAccountInfoAsync`
- `GetBalancesAsync`
- `GetMarginStatusAsync`
- `PlaceOrderAsync`
- `GetOrdersAsync`
- `EditOrderAsync`
- `CancelOrderAsync`
- `CancelAllOrdersAsync`
- `CancelAllAfterAsync`
- `GetUserTradesAsync`
- `GetPositionsAsync`
- `SetCrossMarginLeverageAsync`
- `SetIsolatedMarginLeverageAsync`

Sockets commonly use:

- `SubscribeToTradeUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToBookTickerUpdatesAsync`
- `SubscribeToOrderBookUpdatesAsync`
- `SubscribeToIncrementalOrderBookUpdatesAsync`
- `SubscribeToOrderUpdatesAsync`
- `SubscribeToBalanceUpdatesAsync`
- `SubscribeToUserTradeUpdatesAsync`
- `SubscribeToPositionUpdatesAsync`

Confirm exact overloads from the local library source or `../BitMEX.Net/Examples/ai-friendly/` before generating non-trivial code.

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.ExchangeApi.SharedApi`
- `socketClient.ExchangeApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `IBitMEXSharedApiClient`. It exposes:

- `Rest` as `IBitMEXRestClientExchangeSharedApi`
- `Socket` as `IBitMEXSocketClientExchangeSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `IBitMEXSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.
