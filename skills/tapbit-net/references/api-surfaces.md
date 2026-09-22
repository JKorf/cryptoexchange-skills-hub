# Tapbit.Net API Surfaces

## Package And Scope

- NuGet package: `Tapbit.Net`
- Assumed first release: `1.0.0`
- REST client: `TapbitRestClient`, `ITapbitRestClient`
- API root: `client.SpotApi`
- Credentials: `TapbitCredentials(key, secret)`
- Built-in environment: `TapbitEnvironment.Live`
- Targets: .NET 8, 9, 10, .NET Standard 2.0, and .NET Standard 2.1

Tapbit.Net 1.0.0 is spot REST only. There is no public socket, futures, margin, position, leverage, funding, deposit, withdrawal, transfer, or user-trade API.

## REST Client Map

```csharp
var client = new TapbitRestClient();
```

| Surface | Use for |
| --- | --- |
| `client.SpotApi.ExchangeData` | Server time, symbols, books, tickers, klines, public trades, assets and networks |
| `client.SpotApi.Account` | Spot balances |
| `client.SpotApi.Trading` | Limit orders, batch orders, cancellation, and order queries |
| `client.SpotApi.SharedApi` | Shared spot REST interfaces |

## Exchange Data

`client.SpotApi.ExchangeData` provides:

- `GetServerTimeAsync()`
- `GetSymbolAsync(symbol)`
- `GetSymbolsAsync()`
- `GetOrderBookAsync(symbol, depth)` where depth is 5, 10, 50, or 100
- `GetTickerAsync(symbol)`
- `GetTickersAsync()`
- `GetKlinesAsync(symbol, interval, startTime, endTime)`
- `GetRecentTradesAsync(symbol)`
- `GetAssetsAsync(asset)` where the asset filter is optional

Kline intervals are one, three, five, fifteen, or thirty minutes; one, two, four, six, or twelve hours; one day; one week; or one month.

`TapbitSymbol` includes base/quote assets, price and quantity precision, maker/taker fee rates, minimum quantity, minimum notional, and allowed price fluctuation. Use it before production trading.

## Account

`client.SpotApi.Account` provides:

- `GetBalancesAsync()`
- `GetBalanceAsync(asset)`

Both require credentials. `TapbitBalance` exposes `Asset`, `Available`, `FrozenBalance`, and `TotalBalance`.

## Trading

`client.SpotApi.Trading` provides:

- `PlaceOrderAsync(symbol, side, quantity, price)`
- `PlaceMultipleOrdersAsync(IEnumerable<TapbitOrderRequest>)`
- `CancelOrderAsync(long orderId)`
- `CancelOrdersAsync(IEnumerable<long> orderIds)`
- `GetOpenOrdersAsync(symbol, long? fromId)`
- `GetClosedOrdersAsync(symbol, long? fromId)`
- `GetOrderAsync(long orderId)`

Placement is limit-only even though returned historical order models can contain an `OrderType`. Do not generate a market-order placement overload.

Batch placement and cancellation return `HttpResult<CallResult<TapbitOrderId>[]>`; outer success does not imply every inner item succeeded.

## Symbols And Environments

Native symbols use slash-separated uppercase form, for example `BTC/USDT`. Use:

```csharp
var symbol = TapbitExchange.FormatSymbol(
    "BTC", "USDT", TradingMode.Spot);
```

Only `TapbitEnvironment.Live` is built in. `TapbitEnvironment.CreateCustom(name, spotRestAddress)` creates a deliberate custom REST environment. There is no built-in testnet.

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.SpotApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `ITapbitSharedApiClient`. It exposes:

- `SpotRest` as `ITapbitRestClientSpotSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `ITapbitSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.

## Dependency Injection And User Clients

`services.AddTapbit(...)` registers:

- `ITapbitRestClient`
- `ITapbitUserClientProvider`
- `ITapbitTrackerFactory` and `ITrackerFactory`
- the implemented shared REST interfaces

`ITapbitUserClientProvider` caches one REST client per user identifier. Use `InitializeUserClient`, `GetRestClient`, `ClearUserClients`, or `Clear` as appropriate when credentials change.

## Tracker Support

`ITapbitTrackerFactory` does not support kline or public-trade trackers. Capability checks return false and creation throws.

The factory does support `CreateUserSpotDataTracker(...)` for REST-polled balances and orders. Its `SpotUserDataTrackerConfig` must:

- contain at least one `TrackedSymbols` value
- set `TrackTrades = false`

There is no native websocket stream behind this tracker.

## Result Types

- Native REST: `HttpResult<T>`
- Batch items: `CallResult<TapbitOrderId>` inside an outer `HttpResult`
- Shared cache/capability helpers: `ExchangeCallResult<T>`

Always check `Success` before using `Data`, including every nested batch item.
