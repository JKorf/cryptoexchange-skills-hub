# WhiteBit.Net API Surfaces

## Package And Clients

- Package: `WhiteBit.Net`
- REST: `WhiteBitRestClient`, `IWhiteBitRestClient`
- Socket: `WhiteBitSocketClient`, `IWhiteBitSocketClient`
- Credentials: `WhiteBitCredentials(key, secret)`
- DI: `services.AddWhiteBit(...)`
- Factories: `IWhiteBitOrderBookFactory`, `IWhiteBitTrackerFactory`
- Multi-user clients: `IWhiteBitUserClientProvider`

## V4 Root

REST groups:

- `V4Api.ExchangeData`
- `V4Api.Account`
- `V4Api.Trading`
- `V4Api.CollateralTrading`
- `V4Api.SubAccount`
- `V4Api.Convert`
- `V4Api.Codes`
- `V4Api.SharedApi`

Socket surface: `socket.V4Api` and `socket.V4Api.SharedApi`.

## Exchange Data

- server time, symbols, status, tickers, assets
- order book and recent trades
- deposit/withdrawal information
- collateral and futures symbols
- funding history

## Account

- main, spot, and collateral balances
- deposit addresses, fiat deposit URL, withdrawals, transfers
- deposit/withdrawal history and settings
- collateral summaries and account funding history
- account leverage, trading fees, hedge mode

## Spot Trading

- `PlaceSpotOrderAsync(...)`
- batch spot orders
- cancel one/multiple/all
- open/closed orders, user trades, order trades
- edit order
- kill switch and status

## Collateral Trading

- `PlaceOrderAsync(...)`
- open positions and position history
- open conditional orders
- OCO placement/cancellation
- conditional and OTO cancellation

Collateral trading covers perpetual futures and spot margin products.

## Other REST Groups

`SubAccount`: create, edit, delete, list, block/unblock, transfer, balances, history.

`Convert`: estimate, confirm, history.

`Codes`: create/apply WhiteBit codes and inspect histories.

## Socket Requests And Streams

Socket request/response methods return `QueryResult<T>` and include trade history, last price, ticker, klines, order book, balances, open/closed orders, and user trades.

Subscriptions return `WebSocketResult<UpdateSubscription>` and include:

- public trades, book ticker, last price, ticker, klines, order book
- private spot/margin balances, open/closed orders, user trades
- positions, borrow updates, and margin-position events

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.V4Api.SharedApi`
- `socketClient.V4Api.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `IWhiteBitSharedApiClient`. It exposes:

- `V4Rest` as `IWhiteBitRestClientV4SharedApi`
- `V4Socket` as `IWhiteBitSocketClientV4SharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `IWhiteBitSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.

## Symbols And Environment

- Spot: `ETH_USDT`
- Perpetual collateral: `ETH_PERP`
- Built-in environment: `WhiteBitEnvironment.Live`
- Custom environment: `WhiteBitEnvironment.CreateCustom(...)`
- No built-in testnet in the current source.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Socket requests: `QueryResult<T>`
- Socket subscriptions: `WebSocketResult<UpdateSubscription>`
- Batch methods can contain nested `CallResult<T>` values
- Shared helpers: `ExchangeCallResult<T>`

Always check `Success` before using `Data`.

## Local Order Books

`IWhiteBitOrderBookFactory` exposes `V4`, `Create(SharedSymbol, ...)`, and `CreateV4(symbol, ...)`.
