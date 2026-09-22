# Pionex.Net API Surfaces

Use this file when choosing Pionex client roots, native methods, socket streams, shared interfaces, result types, factories, or environments.

## Package and Client Types

- NuGet package: `Pionex.Net`
- REST client: `PionexRestClient`
- Socket client: `PionexSocketClient`
- Credentials: `PionexCredentials`
- REST interface: `IPionexRestClient`
- Socket interface: `IPionexSocketClient`
- DI extension: `services.AddPionex(...)`
- Local order book factory: `IPionexOrderBookFactory`
- Tracker factory: `IPionexTrackerFactory`
- User client provider: `IPionexUserClientProvider`

## Client Roots

- REST: `client.SpotApi`
- Socket: `socket.SpotApi`

Do not use `UnifiedApi`, `FuturesApi`, `UsdtFuturesApi`, `MarginApi`, or `OptionsApi`.

## REST Exchange Data

`client.SpotApi.ExchangeData`:

- `GetServerTimeAsync()`
- `GetSymbolsAsync(symbols, symbolType)`
- `GetRecentTradesAsync(symbol, limit)`
- `GetOrderBookAsync(symbol, limit)`
- `GetTickersAsync(symbol, type)`
- `GetBookTickersAsync(symbol, type)`
- `GetKlinesAsync(symbol, interval, endTime, limit)`

`GetTickersAsync` and `GetBookTickersAsync` return arrays.

## REST Account

`client.SpotApi.Account`:

- `GetBalancesAsync()`
- `GetFullBalancesAsync()`

`GetBalancesAsync` returns `PionexBalance[]` with `Asset`, `Free`, and `Frozen`.

`GetFullBalancesAsync` returns wallet totals, asset prices, bot-account details, and trader-account details.

## REST Trading

`client.SpotApi.Trading`:

- `PlaceOrderAsync(symbol, side, type, quantity, quoteQuantity, price, immediateOrCancel, clientOrderId)`
- `GetOrderAsync(orderId)`
- `GetOrderByClientOrderIdAsync(clientOrderId)`
- `CancelOrderAsync(symbol, orderId)`
- `GetOpenOrdersAsync(symbol)`
- `GetOrdersAsync(symbol, startTime, endTime, limit)`
- `CancelAllOrdersAsync(symbol)`
- `GetUserTradesAsync(symbol, startTime, endTime)`
- `GetOrderTradesAsync(orderId, fromId)`

There is no test-order method.

## Socket Streams

`socket.SpotApi`:

- `SubscribeToTradeUpdatesAsync(symbol, handler)` returns updates containing `PionexTrade[]`
- `SubscribeToOrderBookUpdatesAsync(symbol, depth, handler)` returns `PionexOrderBook`
- `SubscribeToOrderUpdatesAsync(symbol, handler)` returns `PionexOrder`
- `SubscribeToUserTradeUpdatesAsync(symbol, handler)` returns `PionexUserTrade`
- `SubscribeToBalanceUpdatesAsync(handler)` returns `PionexBalance[]`

The source documents order-book subscription depth as at most 100.

Private streams authenticate through socket client credentials. Do not create a listen key.

## Native Symbols and Models

- Native symbol format: `BASE_QUOTE`
- Examples: `BTC_USDT`, `ETH_USDT`
- Native ticker current/close property: `ClosePrice`
- Native order book sides: `Bids` and `Asks`
- Native order book entry properties: `Price` and `Quantity`

Use `Pionex.Net.Enums.OrderSide`, `OrderType`, `SymbolType`, and `KlineInterval`.

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.SpotApi.SharedApi`
- `socketClient.SpotApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `IPionexSharedApiClient`. It exposes:

- `SpotRest` as `IPionexRestClientSpotSharedApi`
- `SpotSocket` as `IPionexSocketClientSpotSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `IPionexSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Websocket subscription: `WebSocketResult<UpdateSubscription>`

Always check `Success` before accessing `Data`. Non-generic `HttpResult` has no response data.

## Factories and User Clients

`IPionexOrderBookFactory`:

- `Spot`
- `Create(SharedSymbol, options)`
- `CreateSpot(symbol, options)`

`IPionexUserClientProvider`:

- `InitializeUserClient(userIdentifier, credentials, environment)`
- `GetRestClient(userIdentifier, credentials, environment)`
- `GetSocketClient(userIdentifier, credentials, environment)`
- `ClearUserClients(userIdentifier)`
- `Clear()`

## Environments

- Built in: `PionexEnvironment.Live`
- Custom: `PionexEnvironment.CreateCustom(name, restAddress, socketAddress)`

No built-in testnet environment exists.
