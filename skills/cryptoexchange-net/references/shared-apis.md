# Shared API V2 Reference

Use this reference for advanced Shared API V2 work: capability selection, operation options, WebSocket interactions, multi-exchange execution, symbols, quantities, pagination, and rate-limit admission.

Shared APIs provide portable request and response contracts, but an exchange package supplies the implementation. Use an exchange-specific client when the required endpoint, parameter, order type, or response field is not represented by the selected capability.

## Contents

- Capability and client model
- Access patterns and dependency injection
- Results, errors, and cancellation
- Capability options and parameter rules
- Runtime capability lookup
- Multi-exchange execution with CryptoClients.Net
- Symbols, assets, and catalogs
- Quantities and market-data models
- WebSocket subscriptions and operations
- Orders and authenticated operations
- Pagination
- Rate-limit admission
- Portability boundaries and source verification

## Capability And Client Model

V2 defines one interface for each operation or subscription. The important layers are:

| Layer | Example | Purpose |
| --- | --- | --- |
| Transport-independent operation | `IPlaceSpotOrder` | Lets an exchange aggregate select its preferred implementation |
| REST operation | `IPlaceSpotOrderRest` | Exposes the HTTP-specific `HttpResult<T>` |
| WebSocket operation | `IPlaceSpotOrderSocket` | Sends a request over WebSocket and exposes `QueryResult<T>` |
| WebSocket subscription | `ISubscribeTickerSocket` | Starts an ongoing update stream |
| API surface | `restClient.SpotApi.SharedApi` | Compile-time view of capabilities supported by one API and transport |
| Exchange aggregate | `IBinanceSharedApiClient` | Groups all Shared API surfaces for one exchange |
| Capability descriptor | `SharedCapabilities.Tickers.GetTicker.Rest` | Typed key used for runtime lookup |
| Capability resolution | `SharedCapabilityResolution<T>` | Contains the implementation, options, exchange, and transport |

Use a typed API surface when the exchange, market, and transport are known:

```csharp
using Binance.Net.Clients;
using CryptoExchange.Net.SharedApis;

using var restClient = new BinanceRestClient();
IGetTickerRest ticker = restClient.SpotApi.SharedApi;

var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");
var result = await ticker.GetTickerAsync(new GetTickerRequest(symbol), ct);

if (!result.Success)
{
    Console.WriteLine($"[{ticker.Exchange}] {result.Error}");
    return;
}

Console.WriteLine($"{result.Data.Symbol}: {result.Data.LastPrice}");
```

The same portable service can receive another exchange's `IGetTickerRest` implementation without changing its request or response model.

## Choosing An Access Pattern

Choose the narrowest entry point that matches what the application knows:

- Known exchange, API, and transport: use the API surface's typed `SharedApi` property.
- Known exchange with several surfaces: inject its aggregate, such as `IBinanceSharedApiClient`.
- Portable service requiring one operation: inject the operation interface, such as `IGetOrderBookRest`.
- Runtime exchange or transport selection: use capability lookup on an exchange aggregate.
- Selection across many exchanges: inject `IExchangeSharedApiClient` from CryptoClients.Net.

Avoid runtime lookup when a typed surface or aggregate property already expresses the intended implementation at compile time.

## Dependency Injection And Lifetimes

Exchange package registration adds its normal clients, exchange aggregate, Shared API surfaces, and supported capability interfaces:

```csharp
services.AddBinance(options =>
{
    options.Rest.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
    options.Socket.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
});
```

Typical consumers are:

```csharp
public sealed class BinanceWorkflow(IBinanceSharedApiClient sharedApi)
{
    private readonly IGetTickerRest _spotTicker = sharedApi.SpotRest;
}

public sealed class PortableWorkflow(IGetTickerRest ticker)
{
    // Use ticker for one portable operation.
}
```

Capability registrations and exchange aggregates are transient. They resolve through the package's configured underlying clients. A transient socket capability normally wraps the package's singleton socket client; resolving the capability does not itself create a new connection-owning socket client.

When multiple packages or API surfaces can register the same capability, inject `IEnumerable<T>` and select by `Exchange`, `Transport`, and the capability's supported trading modes. Do not rely on whichever single registration the container happens to return.

## Results, Errors, And Cancellation

Result types reflect the selected interaction:

| Interaction | Result |
| --- | --- |
| REST operation | `HttpResult<T>` |
| WebSocket request-response operation | `QueryResult<T>` |
| WebSocket subscription | `WebSocketResult<UpdateSubscription>` |
| Transport-independent operation | `IExchangeCallResult<T>` |
| Local symbol-support helper | `ExchangeCallResult<T>` |

Always check `Success` before using `Data`. Expected validation, authentication, network, exchange, cancellation, and rate-limit failures are returned in `Error` rather than thrown. A successful result does not imply that every nullable model property was supplied by the exchange.

Pass the caller's cancellation token through every operation. Cancellation requested during a library operation is normally represented by a failed result containing a cancellation error. Application-level waits can still throw `OperationCanceledException`, so handle those where appropriate.

Keep transport-specific interfaces when HTTP or WebSocket metadata matters. Use a transport-independent interface when the aggregate's preferred transport should remain an implementation detail.

## Capability Options And Parameter Rules

Each capability exposes a typed options property, such as `GetTickerOptions`, `PlaceSpotOrderOptions`, or `SubscribeTickerOptions`.

Common `CapabilityOptions` information includes:

- `Description` and `RequestNotes`
- `NeedsAuthentication`
- `SupportedTradingModes`
- `SupportsMultipleSymbols` and `MaxSymbolCount`
- `RequestParameterRules`
- `ExchangeParameterRules`
- Operation-specific limits and behavior

Inspect the options for the exact implementation being used:

```csharp
var options = ticker.GetTickerOptions;

Console.WriteLine(options.Description);
Console.WriteLine(string.Join(", ", options.SupportedTradingModes));
Console.WriteLine($"Authenticated operation: {options.NeedsAuthentication}");
```

`RequestParameterRules` describe portable request properties. Each rule has:

- `DefaultSupport`: the status defined by the shared request contract.
- `Support`: the final status after exchange-specific overrides.
- A `RequestParameterSupport` value of `Required`, `Optional`, or `NotSupported`.

`ExchangeParameterRules` describe exchange-only values supplied through the request's `ExchangeParameters`. Their requirement is `Required` or `Optional`.

Do not assume that a nullable request property is supported. A non-null value for a `NotSupported` parameter can be rejected or ignored for compatibility depending on the current library version. Consult the rules before relying on it.

The legacy `RequiredOptionalParameters`, `RequiredExchangeParameters`, and `OptionalExchangeParameters` properties are compatibility views. Generate new code against `RequestParameterRules` and `ExchangeParameterRules`.

Default exchange-specific values can be configured on an `ISharedApi` surface with `SetDefaultExchangeParameter` and cleared with `ResetDefaultExchangeParameters`. A value supplied directly on a request overrides the default.

## Representative Capability Groups

Capability names describe one operation. This list is representative rather than exhaustive; inspect the exchange package's typed `SharedApi` interface or the current interface reference for exact support.

Market data:

- `IGetTickerRest` and `IGetAllTickersRest`
- `IGetBookTickerRest` and `IGetOrderBookRest`
- `IGetRecentTradesRest` and `IGetTradeHistoryRest`
- `IGetKlinesRest`, `IGetMarkPriceKlinesRest`, and `IGetIndexPriceKlinesRest`
- `IGetMarkPriceRest`, `IGetIndexPriceRest`, and `IGetOpenInterestRest`
- `IGetSpotSymbolsRest` and `IGetFuturesSymbolsRest`

Orders and positions:

- `IPlaceSpotOrderRest`, `IGetSpotOrderRest`, and `ICancelSpotOrderRest`
- `IGetOpenSpotOrdersRest` and `IGetClosedSpotOrdersRest`
- Separate client-order-id, trigger-order, edit, and multiple-order capabilities
- Equivalent futures-order capabilities
- `IGetPositionsRest`, `ICloseFullPositionRest`, and `IGetPositionHistoryRest`
- `IGetLeverageRest`, `ISetLeverageRest`, `IGetPositionModeRest`, and `ISetPositionModeRest`

Account and funding:

- `IGetBalancesRest`, `IGetFeesRest`, and `IGetLedgerRest`
- `IGetDepositAddressesRest` and `IGetDepositHistoryRest`
- `IGetWithdrawalHistoryRest` and `IWithdrawRest`
- `IGetTransferHistoryRest` and `ITransferRest`
- Funding-rate, funding-info, and user-funding-history capabilities

Subscriptions:

- `ISubscribeTickerSocket` and `ISubscribeAllTickersSocket`
- `ISubscribeBookTickerSocket` and order-book subscription variants
- `ISubscribeTradesSocket` and `ISubscribeKlinesSocket`
- `ISubscribeBalancesSocket`, `ISubscribePositionsSocket`, and user-trade subscriptions
- `ISubscribeSpotOrdersSocket` and `ISubscribeFuturesOrdersSocket`

WebSocket request-response operations, where implemented:

- `IPlaceSpotOrderSocket` and `ICancelSpotOrderSocket`
- `IPlaceFuturesOrderSocket` and `ICancelFuturesOrderSocket`

## Runtime Capability Lookup

Use a typed exchange aggregate when the desired operation, trading mode, or transport is chosen at runtime:

```csharp
var match = sharedClient.GetCapability(
    SharedCapabilities.Orders.Futures.PlaceOrder.Rest,
    TradingMode.PerpetualLinear);

if (match is null)
{
    Console.WriteLine("Linear futures order placement over REST is unavailable.");
    return;
}

Console.WriteLine($"{match.Exchange} / {match.Transport}");
Console.WriteLine(string.Join(", ", match.Options.SupportedTradingModes));
```

Lookup methods on `ISharedApiClientBase`:

- `GetCapability(reference, tradingMode)` returns one preferred match or null.
- `GetCapability<T>(tradingMode, transport)` selects by interface type.
- `GetCapabilities(...)` returns every matching surface on that exchange aggregate.
- `Discover()` returns summary metadata for inspection and diagnostics.

`SharedCapabilities` values are typed descriptors. For example, `SharedCapabilities.Tickers.GetTicker.Rest` identifies the REST ticker capability and lets the compiler infer `IGetTickerRest`; it is not itself a client and does not guarantee support.

Specify `TradingMode` whenever an exchange exposes multiple matching markets. Specify `SharedTransport` or use a transport-specific descriptor when REST versus WebSocket behavior matters.

## Multi-Exchange Execution With CryptoClients.Net

`IExchangeSharedApiClient` is the V2 entry point for dynamic selection across exchange packages.

Use:

- `GetClient(exchange)` for one exchange aggregate.
- `GetCapability` for one preferred capability on one exchange.
- `GetCapabilities` for one preferred matching implementation per exchange.
- `GetImplementations` for all matching surfaces and transports, including several from one exchange.
- Typed properties such as `Binance`, `Kraken`, and `OKX` when the exchange is known.

Execute independent calls concurrently and retain each resolution alongside its result:

```csharp
using CryptoExchange.Net;
using CryptoExchange.Net.SharedApis;

var request = new GetTickerRequest(
    new SharedSymbol(TradingMode.Spot, "BTC", "USDT"));

var matches = sharedApis.GetCapabilities(
    SharedCapabilities.Tickers.GetTicker.Rest,
    TradingMode.Spot);

var tasks = matches.Select(async match =>
    (Match: match,
     Result: await match.Capability.GetTickerAsync(request, ct)));

await foreach (var item in tasks.ParallelEnumerateAsync())
{
    if (!item.Result.Success)
    {
        Console.WriteLine($"[{item.Match.Exchange}] {item.Result.Error}");
        continue;
    }

    Console.WriteLine(
        $"[{item.Match.Exchange}] {item.Result.Data.LastPrice}");
}
```

Use `Task.WhenAll` when all results are needed before processing. Use `ParallelEnumerateAsync` when completed operations should be handled immediately. One exchange failure should not hide successful results from other exchanges.

## Symbols, Assets, And Catalogs

The portable `SharedSymbol` constructor accepts a trading mode, base asset, quote asset, and optional delivery time:

```csharp
var spot = new SharedSymbol(TradingMode.Spot, "ETH", "USDT");
var linearPerpetual = new SharedSymbol(
    TradingMode.PerpetualLinear,
    "ETH",
    "USDT");
```

The string constructor sets an exchange-native `SymbolName` and bypasses normal formatting. Use it only when the exact native symbol is intentionally required.

`SharedSymbol.UsdOrStable` requests the package's preferred USD or stablecoin quote. It does not guarantee that the market exists. Use `SupportsSpotSymbolAsync` or `SupportsFuturesSymbolAsync` and check the returned `ExchangeCallResult<bool>` when availability must be confirmed.

Use `IGetSpotSymbolsRest` or `IGetFuturesSymbolsRest` to retrieve metadata. A successful call populates that capability instance's catalog:

```csharp
IGetSpotSymbolsRest symbols = restClient.SpotApi.SharedApi;

var result = await symbols.GetSpotSymbolsAsync(
    new GetSymbolsRequest(),
    ct);

if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

var catalog = symbols.SpotSymbolCatalog!;
```

Do not read `SpotSymbolCatalog` or `FuturesSymbolCatalog` before the corresponding successful request. Treat a catalog as a local snapshot for one API and environment, not a guaranteed exhaustive list.

`SharedSpotSymbol` and `SharedFuturesSymbol` classify both assets with `SharedAssetType` and nullable `SharedAssetSubType` values. Valid combinations include:

- `Crypto` with optional `StableCoin`
- `TradFi` with optional `Equity` or `Commodity`
- `Fiat` without a subtype

Treat `Unspecified` or a null subtype as unknown. `GetSymbolsRequest` can filter by trading mode and base/quote type and subtype; invalid type/subtype combinations return a validation error.

## Quantities And Market-Data Models

Use `SharedQuantity` to specify an order quantity explicitly:

- `SharedQuantity.Base(value)` for base-asset quantity.
- `SharedQuantity.Quote(value)` for quote-asset quantity.
- `SharedQuantity.Contracts(value)` for contract count.

Before placing an order, inspect `IPlaceSpotOrderRest.SpotSupportedOrderQuantity` or `IPlaceFuturesOrderRest.FuturesSupportedOrderQuantity`. Support can differ by side and by limit versus market order.

Response models use `SharedOrderQuantity` where several representations can exist. Read `QuantityInBaseAsset`, `QuantityInQuoteAsset`, or `QuantityInContracts`; do not assume a unit. `WithCalculatedQuantities(price, contractSize)` returns a copy with derivable values and does not turn calculated values into exchange-reported values.

For order books, inspect `SharedOrderBook.QuantityType` before interpreting entry quantities.

Ticker operations return `SharedTicker` for both spot and futures. The request's `SharedSymbol.TradingMode` distinguishes a single market. For `IGetAllTickersRest`, inspect `GetAllTickersOptions.RequestParameterRules` to determine whether `GetTickersRequest.TradingMode` is required, optional, or unsupported for that API.

Funding rates, mark prices, index prices, and open interest have dedicated capabilities and models. Do not assume that they are populated on `SharedTicker` or synthesize them with extra requests unless the workflow explicitly requires that behavior.

## WebSocket Subscriptions And Operations

Subscriptions are ongoing streams. Retain and close the returned `UpdateSubscription`:

```csharp
using var socketClient = new BinanceSocketClient();
ISubscribeTickerSocket ticker = socketClient.SpotApi.SharedApi;

var result = await ticker.SubscribeToTickerUpdatesAsync(
    new SubscribeTickerRequest(
        new SharedSymbol(TradingMode.Spot, "BTC", "USDT")),
    update => Console.WriteLine(update.Data.LastPrice),
    ct);

if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

try
{
    await Task.Delay(Timeout.Infinite, ct);
}
catch (OperationCanceledException) when (ct.IsCancellationRequested)
{
}
finally
{
    await result.Data.CloseAsync();
}
```

Keep handlers fast and move expensive processing to a queue, channel, or background service.

WebSocket operations such as `IPlaceSpotOrderSocket` are one request and one response over a socket connection. They return `QueryResult<T>` and do not create a subscription. Treat authenticated order operations as live trading actions even though they use the same socket client as subscriptions.

## Orders And Authenticated Operations

Order placement, lookup, cancellation, histories, user trades, positions, leverage, and position mode are separate capabilities. Depend only on the operations the workflow requires.

Before placing an order:

- Confirm that the capability supports the intended trading mode.
- Inspect parameter rules for order type, time-in-force, client order id, reduce-only behavior, and other optional fields.
- Inspect supported quantity representations.
- Use shared enums and models rather than exchange-native types.
- Prefer explicit limit-order examples unless immediate execution is requested.
- Check `Authenticated` and configure credentials on the underlying exchange client.
- Use a test environment where the exchange package provides one.

If a required parameter is unsupported or the exchange behavior must be exact, use the exchange-specific client instead of approximating it through Shared APIs.

## Pagination

Paginated REST operations return `NextPageRequest`. Pass that object unchanged back to the same operation with the original request:

```csharp
var request = new GetClosedOrdersRequest(symbol, limit: 100);
var page = await orders.GetClosedSpotOrdersAsync(request, ct: ct);

while (page.Success && page.NextPageRequest is not null)
{
    page = await orders.GetClosedSpotOrdersAsync(
        request,
        nextPageToken: page.NextPageRequest,
        ct: ct);
}
```

Do not modify a continuation or reuse it with another operation, API surface, environment, or request context.

`ExchangeHelpers.ExecutePages(...)` yields one result per page until there is no continuation, an error occurs, or cancellation is requested:

```csharp
await foreach (var page in ExchangeHelpers.ExecutePages(
    orders.GetClosedSpotOrdersAsync,
    request,
    ct))
{
    if (!page.Success)
        break;

    await ProcessPageAsync(page.Data, ct);
}
```

Inspect the operation's paginated options for `SupportsAscending`, `SupportsDescending`, `TimePeriodFilterSupport`, `MaxLimit`, and `MaxAge`. Multi-page retrieval still consumes rate-limit capacity; always propagate cancellation.

## Rate-Limit Admission

Shared API surfaces implement rate-limit admission scoping through `ISharedApi`. Use it when one operation should be rejected before the exchange rate limit reaches full utilization:

```csharp
using CryptoExchange.Net;
using CryptoExchange.Net.RateLimiting;

var result = await ticker.WithRateLimitAdmissionAsync(
    RateLimitAdmission.WithMaxUtilizationRatio(0.8),
    client => client.GetTickerAsync(request, ct));
```

The admission scope applies only to requests started inside the callback on the same async flow. It does not reorder queued requests. Coalesced public GET requests can reuse an already-running request without consuming another rate-limit slot, so a new admission decision is unnecessary in that case.

## Portability Boundary

Use an exchange-specific client when:

- No Shared API capability represents the endpoint.
- A required parameter is `NotSupported` or absent from the shared request model.
- The response requires exchange-specific fields.
- The workflow depends on exact margin, account, trigger-order, position-mode, or listen-key semantics.
- Production execution requires exchange-specific validation or guarantees.

It is valid to combine Shared API and native API calls in one application. Do not pass exchange-native request types, enums, models, or symbol strings into a Shared API operation.

## Source Verification

When a sibling checkout is available, use these sources of truth:

- `../CryptoExchange.Net/CryptoExchange.Net/SharedApis/V2/` for operation and subscription interfaces.
- `../CryptoExchange.Net/CryptoExchange.Net/SharedApis/CapabilityReferences/` for runtime descriptors.
- `../CryptoExchange.Net/CryptoExchange.Net/SharedApis/Options/` and `Parameters/` for capability and parameter metadata.
- The exchange package's `Interfaces/Clients/**/**SharedApi.cs` files for its compile-time surface.
- `../CryptoClients.Net/CryptoClients.Net/Interfaces/IExchangeSharedApiClient.cs` for multi-exchange lookup.

Use `https://cryptoexchange.jkorf.dev/docs/shared-api` for the maintained conceptual documentation and interface reference.
