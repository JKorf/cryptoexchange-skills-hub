# BitMart.Net API Surfaces

Use this reference when choosing where a BitMart endpoint belongs.

## REST Clients

```csharp
var client = new BitMartRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.SpotApi.ExchangeData` | Server status/time, spot assets, symbols, symbol names, tickers, deposit/withdraw info, klines, trades, order book |
| `client.SpotApi.Account` | Funding balances, spot balances, deposit addresses, withdrawal quotas, withdrawals, deposit/withdrawal history, isolated margin accounts/transfers, fees, withdrawal addresses |
| `client.SpotApi.Margin` | Isolated margin borrow, repay, borrow/repay history, borrow info |
| `client.SpotApi.SubAccount` | Spot sub-account transfer operations, transfer history, sub-account balances, sub-account list |
| `client.SpotApi.Trading` | Spot order placement, multiple orders, cancellation, margin order placement, order lookup, client-order lookup, open/closed orders, user trades, order trades |
| `client.UsdFuturesApi.ExchangeData` | Contracts, futures order book, open interest, current/historical funding rate, klines, mark klines, leverage brackets, recent trades |
| `client.UsdFuturesApi.Account` | Futures balances, transfer history, spot/futures transfer, leverage, symbol fee rate, transaction history, position mode |
| `client.UsdFuturesApi.SubAccount` | Futures sub-account transfers, balances, transfer history |
| `client.UsdFuturesApi.Trading` | Futures order lookup, open/closed orders, positions, position risk, user trades, order placement/cancel/edit, trigger orders, trailing orders, TP/SL, cancel-all-after |

Do not use Binance/Bitget-style roots such as `SpotApiV3`, `FuturesApiV2`, `UsdFuturesApiV3`, `CoinFuturesApi`, or `PerpetualFuturesApi`.

## Socket Clients

```csharp
var socket = new BitMartSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.SpotApi` | Spot ticker, kline, partial and incremental order book, trade, order, book ticker, balance streams |
| `socket.UsdFuturesApi` | USD futures ticker/tickers, trade, kline, order book, order, balance, position streams |

Websocket subscription methods return `WebSocketResult<UpdateSubscription>`.

## Common Method Names

Spot market data commonly uses:

- `GetTickerAsync`
- `GetTickersAsync`
- `GetSymbolsAsync`
- `GetSymbolNamesAsync`
- `GetKlinesAsync`
- `GetTradesAsync`
- `GetOrderBookAsync`

Spot account, margin, and trading commonly use:

- `GetSpotBalancesAsync`
- `GetFundingBalancesAsync`
- `GetDepositAddressAsync`
- `WithdrawAsync`
- `BorrowAsync`
- `RepayAsync`
- `PlaceOrderAsync`
- `PlaceMarginOrderAsync`
- `CancelOrderAsync`
- `GetOpenOrdersAsync`
- `GetClosedOrdersAsync`
- `GetUserTradesAsync`

USD futures commonly uses:

- `GetContractsAsync`
- `GetCurrentFundingRateAsync`
- `GetBalancesAsync`
- `SetLeverageAsync`
- `SetPositionModeAsync`
- `GetPositionsAsync`
- `PlaceOrderAsync`
- `CancelOrderAsync`
- `PlaceTriggerOrderAsync`
- `PlaceTpSlOrderAsync`

Sockets commonly use:

- `SubscribeToTickerUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToOrderBookUpdatesAsync`
- `SubscribeToPartialOrderBookUpdatesAsync`
- `SubscribeToTradeUpdatesAsync`
- `SubscribeToOrderUpdatesAsync`
- `SubscribeToBalanceUpdatesAsync`
- `SubscribeToPositionUpdatesAsync`

Confirm exact overloads from the local library source or `../BitMart.Net/Examples/ai-friendly/` before generating non-trivial code.

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.SpotApi.SharedApi`
- `socketClient.SpotApi.SharedApi`
- `restClient.UsdFuturesApi.SharedApi`
- `socketClient.UsdFuturesApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `IBitMartSharedApiClient`. It exposes:

- `SpotRest` as `IBitMartRestClientSpotSharedApi`
- `UsdFuturesRest` as `IBitMartRestClientUsdFuturesSharedApi`
- `SpotSocket` as `IBitMartSocketClientSpotSharedApi`
- `UsdFuturesSocket` as `IBitMartSocketClientUsdFuturesSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `IBitMartSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.
