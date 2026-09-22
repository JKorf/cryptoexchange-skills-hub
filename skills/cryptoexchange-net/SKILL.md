---
name: cryptoexchange-net
description: Build exchange-agnostic C#/.NET integrations with CryptoExchange.Net Shared API V2, including strict operation capabilities, REST and WebSocket transports, exchange aggregates, runtime capability lookup, dependency injection, shared symbols and models, and multi-exchange workflows. Use when code must work across exchange packages or select exchanges, markets, capabilities, or transports dynamically. Prefer an exchange-specific skill when portability is not required.
---

# CryptoExchange.Net Shared API V2

## Purpose

Use this skill for application code that consumes common functionality from multiple CryptoExchange.Net-based exchange packages.

`CryptoExchange.Net` defines shared contracts and infrastructure; it does not connect to an exchange by itself. Install one or more exchange packages, such as `Binance.Net`, `JK.OKX.Net`, or `KrakenExchange.Net`, or use `CryptoClients.Net` for broad exchange coverage.

Choose the narrowest suitable API:

1. Use an exchange-specific client when the workflow needs exchange-only endpoints, parameters, order types, or response fields.
2. Use a typed Shared API surface when the exchange, market, and transport are known.
3. Inject one capability interface when application logic should depend on a single portable operation.
4. Use an exchange Shared API aggregate when one known exchange exposes several relevant surfaces.
5. Use `CryptoClients.Net` and `IExchangeSharedApiClient` when exchanges are selected dynamically.

Read `references/shared-apis.md` for advanced capability lookup, parameter rules, multi-exchange execution, symbols, quantities, subscriptions, pagination, and rate-limit admission. Read `references/ecosystem.md` when selecting repositories or NuGet package IDs. Read `references/safety.md` before generating account, trading, leverage, transfer, withdrawal, or credential code.

## V2 Rules

Shared API V2 uses one interface per operation or subscription. Generate V2 APIs for new code:

- Access a typed implementation through an API surface's `SharedApi` property.
- Use capability interfaces such as `IGetTickerRest`, `IPlaceSpotOrderRest`, and `ISubscribeTickerSocket`.
- Use the corresponding options property to inspect supported trading modes, authentication requirements, request parameter rules, exchange parameter rules, limits, and notes.
- Treat support as capability-specific. Support for one operation does not imply support for another operation on the same API.
- Use an exchange aggregate and capability lookup only when the implementation must be selected at runtime.

Do not generate legacy `.SharedClient` access, broad interfaces such as `ISpotTickerRestClient`, or methods such as `GetSpotTickerAsync` unless the user explicitly asks to maintain or migrate V1 code.

Common V1-to-V2 mappings include:

| V1 | V2 |
| --- | --- |
| `api.SharedClient` | `api.SharedApi` |
| `ISpotTickerRestClient.GetSpotTickerAsync` | `IGetTickerRest.GetTickerAsync` |
| `IFuturesTickerRestClient.GetFuturesTickerAsync` | `IGetTickerRest.GetTickerAsync` |
| `ISpotOrderRestClient` | Separate place, get, cancel, history, and trade capabilities |
| `IFuturesOrderRestClient` | Separate order and position capabilities |
| `ITickerSocketClient` | `ISubscribeTickerSocket` |
| Socket order-management clients | Separate place and cancel socket capabilities |

## REST Quick Start

Use the API's typed `SharedApi` surface when the exchange, market, and transport are known:

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

Move portable logic behind the narrow capability it needs:

```csharp
using CryptoExchange.Net.Objects;
using CryptoExchange.Net.SharedApis;

public sealed class TickerService(IGetTickerRest ticker)
{
    public Task<HttpResult<SharedTicker>> GetAsync(
        SharedSymbol symbol,
        CancellationToken ct = default)
        => ticker.GetTickerAsync(new GetTickerRequest(symbol), ct);
}
```

When several implementations may be registered, inject `IEnumerable<IGetTickerRest>` and select by `Exchange` and `GetTickerOptions.SupportedTradingModes` rather than resolving an arbitrary single instance.

## Capability Options

Inspect the operation's options before relying on optional functionality:

```csharp
if (!ticker.GetTickerOptions.SupportedTradingModes.Contains(TradingMode.Spot))
    return;

var requestRules = ticker.GetTickerOptions.RequestParameterRules;
var exchangeRules = ticker.GetTickerOptions.ExchangeParameterRules;
```

Do not infer support from nullable request properties alone. Parameter rules communicate whether a parameter is supported, required, optional, or exchange-specific for that implementation. Add exchange-specific values through the request's `ExchangeParameters` only when portable parameters cannot represent the required behavior.

## Transport-Specific And Transport-Independent Capabilities

Transport-specific interfaces expose the native result shape:

- REST capabilities such as `IGetTickerRest` return `HttpResult<T>`.
- WebSocket request capabilities such as `IPlaceSpotOrderSocket` return `QueryResult<T>`.
- Subscription capabilities such as `ISubscribeTickerSocket` return `WebSocketResult<UpdateSubscription>`.

Some operations also have a transport-independent base interface, such as `IPlaceSpotOrder`. It returns `IExchangeCallResult<T>` and is useful when the exchange aggregate should choose its preferred transport. Use `...Rest` or `...Socket` when transport semantics or result metadata matter.

Always check `Success` before reading `Data`. Expected validation, authentication, network, exchange, cancellation, and rate-limit failures are returned through `Error`; do not use exceptions as the normal failure path.

## Exchange Aggregates And Runtime Lookup

Each supporting exchange package registers a typed aggregate such as `IBinanceSharedApiClient`. Its properties expose that exchange's REST and WebSocket Shared API surfaces at compile time. Prefer those properties when the desired surface is known.

Use `GetCapability` when the operation, trading mode, or transport is chosen dynamically:

```csharp
var match = sharedClient.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest,
    TradingMode.Spot);

if (match is null)
    return;

var result = await match.Capability.GetTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")),
    ct);
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A resolution provides `Capability`, `Options`, `Exchange`, and `Transport`.

Use:

- `GetCapability` for one preferred match.
- `GetCapabilities` for all matching surfaces on one exchange aggregate.
- `Discover` for summary metadata intended for inspection or diagnostics.

Specify a `TradingMode` when several markets can implement the same capability. Specify a transport when REST versus WebSocket behavior matters.

## CryptoClients.Net

Use `IExchangeSharedApiClient` for capability selection across exchange packages:

- `GetClient(exchange)` returns one exchange aggregate.
- `GetCapability` returns one preferred capability for one exchange.
- `GetCapabilities` returns one preferred match per exchange.
- `GetImplementations` returns every matching API surface and transport, including multiple implementations from one exchange.
- Typed properties such as `Binance`, `Kraken`, and `OKX` expose exchange aggregates directly.

Execute independent cross-exchange calls concurrently. Map resolutions to tasks and use `Task.WhenAll`, or use `ParallelEnumerateAsync` when results should be consumed as they complete. Preserve the association between each result and its `SharedCapabilityResolution` so failures can be attributed to the correct exchange and transport.

## Symbols, Quantities, And Catalogs

Use `SharedSymbol` instead of native symbol strings in Shared API requests:

```csharp
var spot = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");
var linearPerpetual = new SharedSymbol(TradingMode.PerpetualLinear, "BTC", "USDT");
```

The implementation converts the shared symbol to the exchange's native format. Use native symbol strings only with exchange-specific APIs.

Use `IGetSpotSymbolsRest` and `IGetFuturesSymbolsRest` for symbol metadata. Their `SpotSymbolCatalog` and `FuturesSymbolCatalog` properties are populated only after a successful corresponding symbol request. Treat `SharedAssetType.Unspecified` and a null subtype as unknown rather than assuming cryptocurrency.

Shared order and order-book quantities can represent base-asset, quote-asset, or contract quantities. Read the appropriate `SharedOrderQuantity` property and inspect the capability options instead of assuming units.

## WebSocket Subscriptions

Use a strict subscription capability and always close successful subscriptions:

```csharp
using var socketClient = new BinanceSocketClient();
ISubscribeTickerSocket tickerStream = socketClient.SpotApi.SharedApi;

var result = await tickerStream.SubscribeToTickerUpdatesAsync(
    new SubscribeTickerRequest(
        new SharedSymbol(TradingMode.Spot, "BTC", "USDT")),
    update => Console.WriteLine(update.Data.LastPrice),
    ct);

if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

await result.Data.CloseAsync();
```

Keep handlers fast and move expensive work to a queue, channel, or background processor. Treat WebSocket query capabilities that place or cancel orders as live trading operations, not subscriptions.

## Dependency Injection

Register the exchange package using its current extension method, for example:

```csharp
services.AddBinance(options =>
{
    options.Rest.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
    options.Socket.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
});
```

The package registration adds its normal clients, typed Shared API aggregate, API surfaces, and supported capability interfaces. Inject the exchange aggregate for several known surfaces, or inject the narrow capability needed by a portable service. Shared capability registrations are transient but resolve through the package's configured REST and socket clients; this does not imply a new socket connection-owning client for every capability resolution.

## Portability Boundary

Switch to an exchange-specific client when:

- The required endpoint has no Shared API capability.
- A required request parameter or order type is unsupported by the selected capability.
- The response needs exchange-specific fields.
- The workflow depends on exact margin, account, trigger-order, position-mode, or listen-key semantics.
- Production behavior requires exchange-specific validation or execution details.

It is valid to use Shared APIs for common workflows and native APIs for exchange-specific functionality in the same application. Do not pass native request types, enums, models, or symbol strings into Shared API calls.

## Source Verification

When a sibling checkout is available, verify exact current contracts before generating non-trivial code:

- `../CryptoExchange.Net/CryptoExchange.Net/SharedApis/V2/` for capability interfaces.
- `../CryptoExchange.Net/CryptoExchange.Net/SharedApis/CapabilityReferences/` for `SharedCapabilities` descriptors.
- The selected exchange package's `Interfaces/Clients/**/**SharedApi.cs` files for its compile-time capability surface.
- `../CryptoClients.Net/CryptoClients.Net/Interfaces/IExchangeSharedApiClient.cs` for multi-exchange lookup.

Use the official Shared API documentation at `https://cryptoexchange.jkorf.dev/docs/shared-api` for conceptual guidance and examples.
