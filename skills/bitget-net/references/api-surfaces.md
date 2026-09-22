# Bitget.Net API Surfaces

Use this reference when choosing where a Bitget endpoint belongs.

## REST Clients

```csharp
var client = new BitgetRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.SpotApiV2.ExchangeData` | Spot server time, announcements, assets, symbols, VIP fee rates, tickers, order book, klines, recent trades, trade history |
| `client.SpotApiV2.Account` | Funding balances, spot balances, account info, trade fee, asset valuation, ledger, transfers, deposits, withdrawals, sub-account balances and transfers |
| `client.SpotApiV2.Trading` | Spot order placement, multiple orders, cancel/replace, cancellation, order lookup, open/closed orders, user trades, trigger orders |
| `client.SpotApiV2.Margin` | Cross and isolated spot margin balances, borrow/repay, interest, risk, max borrowable/transferable, margin orders, liquidation history |
| `client.FuturesApiV2.ExchangeData` | Futures contracts, tickers, funding rates, order book, klines, trades, open interest, index/mark price data |
| `client.FuturesApiV2.Account` | Futures balances, leverage, margin mode, position mode, ledger, ADL rank, liquidation price, openable quantity |
| `client.FuturesApiV2.Trading` | Futures positions, position history, order placement/edit/cancel, open/closed orders, user trades, close positions, trigger orders, TP/SL |
| `client.CopyTradingFuturesV2.Trader` | Copy trading lead/trader settings and information |
| `client.CopyTradingFuturesV2.Follower` | Copy trading follower traders and current orders |
| `client.UnifiedApi.Account` | UTA balances, funding balances, account config, leverage, hold mode, repayments, transfers, deposits, withdrawals, fees, account mode |
| `client.UnifiedApi.ExchangeData` | UTA server time, futures/spot/margin symbols, tickers, order books, trades, klines, funding rates, open interest, proof of reserves |
| `client.UnifiedApi.Trading` | UTA order placement/edit/cancel, open orders, user trades, positions, position history, max open available, strategy orders |

Do not use older or invented roots such as `SpotApi`, `FuturesApi`, `MarginApi`, `UsdFuturesApi`, `PerpetualFuturesApi`, `CopyTradingApi`, or `BrokerV2`.

## Socket Clients

```csharp
var socket = new BitgetSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.SpotApiV2` | Spot ticker, trade, kline, order book, margin index price, spot order/trade/balance updates, cross/isolated margin account and order updates |
| `socket.FuturesApiV2` | Futures ticker, trade, kline, order book, balance, position, user trade, order, trigger order, position history, equity, ADL updates |
| `socket.UnifiedApi` | UTA/unified websocket endpoints; verify exact subscriptions in `IBitgetSocketClientUnifiedApi` before generating code |

Websocket subscription methods return `WebSocketResult<UpdateSubscription>`.

## Common Method Names

Spot market data commonly uses:

- `GetTickersAsync`
- `GetSymbolsAsync`
- `GetOrderBookAsync`
- `GetKlinesAsync`
- `GetRecentTradesAsync`
- `GetTradesAsync`

Spot account and trading commonly use:

- `GetSpotBalancesAsync`
- `PlaceOrderAsync`
- `CancelOrderAsync`
- `GetOpenOrdersAsync`
- `GetClosedOrdersAsync`
- `GetUserTradesAsync`

Futures commonly uses:

- `GetTickerAsync`
- `GetFundingRateAsync`
- `GetBalancesAsync`
- `SetLeverageAsync`
- `SetMarginModeAsync`
- `SetPositionModeAsync`
- `GetPositionsAsync`
- `PlaceOrderAsync`
- `CancelOrderAsync`
- `ClosePositionsAsync`

Sockets commonly use:

- `SubscribeToTickerUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToOrderBookUpdatesAsync`
- `SubscribeToBalanceUpdatesAsync`
- `SubscribeToOrderUpdatesAsync`
- `SubscribeToUserTradeUpdatesAsync`
- `SubscribeToPositionUpdatesAsync`

Confirm exact overloads from the local library source or `../Bitget.Net/Examples/ai-friendly/` before generating non-trivial code.

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.SpotApiV2.SharedApi`
- `socketClient.SpotApiV2.SharedApi`
- `restClient.FuturesApiV2.SharedApi`
- `socketClient.FuturesApiV2.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `IBitgetSharedApiClient`. It exposes:

- `SpotRest` as `IBitgetRestClientSpotSharedApi`
- `FuturesRest` as `IBitgetRestClientFuturesSharedApi`
- `SpotSocket` as `IBitgetSocketClientSpotSharedApi`
- `FuturesSocket` as `IBitgetSocketClientFuturesSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `IBitgetSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.
