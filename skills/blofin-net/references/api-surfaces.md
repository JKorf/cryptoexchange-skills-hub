# BloFin.Net API Surfaces

Use this reference when choosing where a BloFin endpoint belongs.

## REST Clients

```csharp
var client = new BloFinRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.AccountApi` | General account balances, transfers, account config, API key info, withdrawal history, deposit history |
| `client.AccountApi.SharedApi` | SharedApis account REST interfaces for withdrawals and deposits |
| `client.FuturesApi.ExchangeData` | Futures symbols, tickers, order book, recent trades, index/mark price, funding rates/history, klines, index/mark klines, position tiers |
| `client.FuturesApi.Account` | Futures balances, margin mode, position mode, leverage reads and updates |
| `client.FuturesApi.Trading` | Futures positions, order placement, batch orders, TP/SL orders, trigger orders, cancellations, open/closed orders, close position, order lookup, user trades, price limits, position history |
| `client.FuturesApi.SharedApi` | SharedApis REST interfaces for futures workflows |

Do not use exchange roots from other libraries such as `ExchangeApi`, `SpotApi`, `UsdFuturesApi`, `FuturesApiV2`, `SpotApiV3`, `CoinFuturesApi`, or `PerpetualFuturesApi`.

## Socket Clients

```csharp
var socket = new BloFinSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.FuturesApi` | Futures trades, klines, index price klines, mark price klines, order book, tickers, funding rates, private positions, private orders, private trigger orders, private balances |
| `socket.FuturesApi.SharedApi` | SharedApis futures socket interfaces |

Websocket subscription methods return `WebSocketResult<UpdateSubscription>`.

## Common Method Names

General account commonly uses:

- `GetAccountBalancesAsync`
- `TransferAsync`
- `GetAccountConfigAsync`
- `GetApiKeyInfoAsync`
- `GetTransferHistoryAsync`
- `GetWithdrawalHistoryAsync`
- `GetDepositHistoryAsync`

Futures market data commonly uses:

- `GetSymbolsAsync`
- `GetTickersAsync`
- `GetOrderBookAsync`
- `GetRecentTradesAsync`
- `GetIndexMarkPriceAsync`
- `GetFundingRateAsync`
- `GetFundingRateHistoryAsync`
- `GetKlinesAsync`
- `GetIndexPriceKlinesAsync`
- `GetMarkPriceKlinesAsync`
- `GetPositionTiersAsync`

Futures account and trading commonly uses:

- `GetBalancesAsync`
- `GetMarginModeAsync`
- `SetMarginModeAsync`
- `GetPositionModeAsync`
- `SetPositionModeAsync`
- `GetLeverageAsync`
- `SetLeverageAsync`
- `GetPositionsAsync`
- `PlaceOrderAsync`
- `PlaceMultipleOrdersAsync`
- `PlaceTpSlOrderAsync`
- `PlaceTriggerOrderAsync`
- `CancelOrderAsync`
- `CancelOrdersAsync`
- `CancelTpSlOrdersAsync`
- `CancelTriggerOrderAsync`
- `GetOpenOrdersAsync`
- `GetOpenTpSlOrdersAsync`
- `GetOpenTriggerOrdersAsync`
- `ClosePositionAsync`
- `GetOrderAsync`
- `GetTpSlOrderAsync`
- `GetClosedOrdersAsync`
- `GetClosedTpSlOrdersAsync`
- `GetClosedTriggerOrdersAsync`
- `GetUserTradesAsync`
- `GetPriceLimitsAsync`
- `GetPositionHistoryAsync`

Sockets commonly use:

- `SubscribeToTradeUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToIndexPriceKlineUpdatesAsync`
- `SubscribeToMarkPriceKlineUpdatesAsync`
- `SubscribeToOrderBookUpdatesAsync`
- `SubscribeToTickerUpdatesAsync`
- `SubscribeToFundingRateUpdatesAsync`
- `SubscribeToPositionUpdatesAsync`
- `SubscribeToOrderUpdatesAsync`
- `SubscribeToTriggerOrderUpdatesAsync`
- `SubscribeToBalanceUpdatesAsync`
- `SubscribeToInverseBalanceUpdatesAsync`

Confirm exact overloads from the local library source or `../BloFin.Net/Examples/ai-friendly/` before generating non-trivial code.

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.AccountApi.SharedApi`
- `restClient.FuturesApi.SharedApi`
- `socketClient.FuturesApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `IBloFinSharedApiClient`. It exposes:

- `AccountRest` as `IBloFinRestClientAccountSharedApi`
- `FuturesRest` as `IBloFinRestClientFuturesSharedApi`
- `FuturesSocket` as `IBloFinSocketClientFuturesSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `IBloFinSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.
