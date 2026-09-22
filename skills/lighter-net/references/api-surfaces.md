# Lighter.Net API Surfaces

Use this reference when choosing where a Lighter endpoint belongs.

## REST Clients

```csharp
var client = new LighterRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.ExchangeApi.ExchangeData` | Status, system config, symbols, symbol details/tickers, order books, recent trades, klines, mark price klines, funding rates, exchange stats, announcements, assets |
| `client.ExchangeApi.Account` | API key generation, nonces, accounts, account limits, account metadata, PnL, liquidation history, funding history, deposits, transfers, withdrawals, API keys, integrator approval, leverage, margin |
| `client.ExchangeApi.Trading` | Place/edit/cancel/cancel-all orders, batch orders, open orders, closed orders, user trades |
| `client.ExchangeApi.SharedApi` | SharedApis REST interfaces for exchange-agnostic code |

## Socket Clients

```csharp
var socket = new LighterSocketClient();
```

| Surface | Use for |
| --- | --- |
| `socket.ExchangeApi.ExchangeData` | Public trade, order book, book ticker, spot ticker, futures ticker, kline, and mark-price kline subscriptions |
| `socket.ExchangeApi.Account` | Account, user stats, balance subscriptions; leverage and margin websocket requests |
| `socket.ExchangeApi.Trading` | Order, user trade, position subscriptions; place/edit/cancel/cancel-all order websocket requests |
| `socket.ExchangeApi.SharedApi` | SharedApis socket interfaces for exchange-agnostic code |

## Common REST Method Names

Market data commonly uses:

- `GetStatusAsync`
- `GetSystemConfigAsync`
- `GetSymbolsAsync`
- `GetSymbolDetailsAsync`
- `GetOrderBookAsync`
- `GetRecentTradesAsync`
- `GetKlinesAsync`
- `GetFundingRatesAsync`
- `GetFundingRateHistoryAsync`

Account and transaction endpoints commonly use:

- `GetAccountsAsync`
- `GetAccountLimitsAsync`
- `GetAccountMetadataAsync`
- `GetPnlAsync`
- `GetDepositHistoryAsync`
- `GetTransferHistoryAsync`
- `GetWithdrawHistoryAsync`
- `GetApiKeysAsync`
- `SetLeverageAsync`
- `UpdateMarginAsync`

Trading commonly uses:

- `PlaceOrderAsync`
- `PlaceMultipleOrdersAsync`
- `EditOrderAsync`
- `CancelOrderAsync`
- `CancelAllOrdersAsync`
- `GetOpenOrdersAsync`
- `GetClosedOrdersAsync`
- `GetUserTradesAsync`

## Common Socket Method Names

Subscriptions commonly use:

- `SubscribeToTradeUpdatesAsync`
- `SubscribeToOrderBookUpdatesAsync`
- `SubscribeToBookTickerUpdatesAsync`
- `SubscribeToSpotTickerUpdatesAsync`
- `SubscribeToFuturesTickerUpdatesAsync`
- `SubscribeToKlineUpdatesAsync`
- `SubscribeToAccountUpdatesAsync`
- `SubscribeToBalanceUpdatesAsync`
- `SubscribeToOrderUpdatesAsync`
- `SubscribeToPositionUpdatesAsync`

Socket request methods commonly use:

- `PlaceOrderAsync`
- `PlaceMultipleOrdersAsync`
- `EditOrderAsync`
- `CancelOrderAsync`
- `CancelAllOrdersAsync`
- `SetLeverageAsync`
- `UpdateMarginAsync`

Subscription methods return `WebSocketResult<UpdateSubscription>`. Socket request methods return `QueryResult<T>`.

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.ExchangeApi.SharedApi`
- `socketClient.ExchangeApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `ILighterSharedApiClient`. It exposes:

- `Rest` as `ILighterRestClientExchangeSharedApi`
- `Socket` as `ILighterSocketClientExchangeSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `ILighterSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.
