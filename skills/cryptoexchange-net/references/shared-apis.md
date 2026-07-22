# SharedApis Reference

Use `CryptoExchange.Net.SharedApis` when the user wants exchange-agnostic C# code. Shared APIs let the same request/model shape work across Binance, OKX, Bybit, Kraken, Coinbase, Kucoin, and other CryptoExchange.Net-based libraries that implement the relevant interface.

Use exchange-specific clients instead when the user needs an endpoint, option, order type, or model field that is not represented in the shared interface.

## Core Idea

Each exchange library exposes `.SharedClient` properties from its normal REST/socket API surfaces. Those shared clients implement common interfaces such as `ISpotTickerRestClient`, `ISpotOrderRestClient`, and `ITickerSocketClient`.

```csharp
using Binance.Net.Clients;
using OKX.Net.Clients;
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient binance = new BinanceRestClient().SpotApi.SharedClient;
ISpotTickerRestClient okx = new OKXRestClient().UnifiedApi.SharedClient;

var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");

var result = await binance.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine($"[{binance.Exchange}] {result.Error}");
    return;
}

Console.WriteLine($"[{binance.Exchange}] {result.Data.Symbol}: {result.Data.LastPrice}");
```

## Result Types

- REST shared API calls return `HttpResult<T>`.
- Websocket shared subscription calls return `WebSocketResult<UpdateSubscription>`.
- Always check `Success` before reading `Data`.
- Use `Error` for exchange/API/network/rate-limit failures.
- Use `client.Exchange` for logging instead of trying to infer the exchange from the result payload.

## Symbol Normalization

Use `SharedSymbol` for shared calls. Do not pass native exchange strings such as `BTCUSDT` or `BTC-USDT` into shared request types.

```csharp
var spot = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");
var linearPerp = new SharedSymbol(TradingMode.PerpetualLinear, "BTC", "USDT");
```

Different exchanges format symbols differently. The shared client translates `SharedSymbol` into the exchange-native symbol format internally.

For unusual asset names, aliases, or exchange-specific symbol mappings, inspect the exchange library docs/source and the shared symbol support methods on the relevant shared client.

## Asset Classification And Symbol Catalogs

`SharedSpotSymbol` and `SharedFuturesSymbol` classify both sides of a market with:

- `BaseAssetType` and `QuoteAssetType`: `SharedAssetType.Unspecified`, `Crypto`, `Fiat`, or `TradFi`
- `BaseAssetSubType` and `QuoteAssetSubType`: nullable `SharedAssetSubType.StableCoin`, `Equity`, or `Commodity`

Treat `Unspecified` and a `null` subtype as unknown classification. Do not assume an unclassified asset is cryptocurrency. Valid subtype relationships are:

- `Crypto` + `StableCoin`
- `TradFi` + `Equity`
- `TradFi` + `Commodity`
- `Fiat` without a subtype

Filter either spot or futures symbol discovery with `GetSymbolsRequest`:

```csharp
var request = new GetSymbolsRequest(
    baseAssetType: SharedAssetType.TradFi,
    baseAssetSubType: SharedAssetSubType.Equity,
    quoteAssetType: SharedAssetType.Crypto,
    quoteAssetSubType: SharedAssetSubType.StableCoin);

var result = await futuresSymbolClient.GetFuturesSymbolsAsync(request);
```

The request also accepts `tradingMode` and `exchangeParameters`. Invalid asset type/subtype combinations fail request validation.

Each shared symbol client exposes a nullable cached catalog. Populate and use a spot catalog as follows:

```csharp
ISpotSymbolRestClient symbols = new BinanceRestClient().SpotApi.SharedClient;

var result = await symbols.GetSpotSymbolsAsync(new GetSymbolsRequest());
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

var catalog = symbols.SpotSymbolCatalog!;

if (catalog.Assets.TryGetValue("USDT", out var usdt))
    Console.WriteLine($"{usdt.Name}: {usdt.Type} / {usdt.SubType}");

if (catalog.Symbols.TryGetValue("BTCUSDT", out var btcUsdt))
    Console.WriteLine($"{btcUsdt.BaseAsset}/{btcUsdt.QuoteAsset}");
```

Use the same lifecycle for futures:

```csharp
var result = await futuresSymbolClient.GetFuturesSymbolsAsync(
    new GetSymbolsRequest(tradingMode: TradingMode.PerpetualLinear));

if (result.Success)
    Console.WriteLine(futuresSymbolClient.FuturesSymbolCatalog!.Symbols.Count);
```

`SpotSymbolCatalog` and `FuturesSymbolCatalog` are separate `SharedSymbolCatalog` instances. `Exchange` identifies the exchange, `Assets` is keyed by asset name and contains `SharedAssetInfo` (`Name`, `Type`, `SubType`), and `Symbols` is keyed by the native exchange symbol name. Never read a catalog before the corresponding successful symbol request.

## REST Interface Families

Market data:

- `ISpotTickerRestClient`: spot ticker by symbol and all tickers
- `IFuturesTickerRestClient`: futures ticker by symbol and all tickers
- `IBookTickerRestClient`: best bid/ask
- `IOrderBookRestClient`: order book snapshots
- `IRecentTradeRestClient`: recent public trades
- `IKlineRestClient`: candlestick/kline data
- `ISpotSymbolRestClient`: spot symbol metadata/support checks
- `IFuturesSymbolRestClient`: futures symbol metadata/support checks

Orders:

- `ISpotOrderRestClient`: place, get, cancel, open/closed spot orders
- `IFuturesOrderRestClient`: place, get, cancel, open/closed futures orders, positions, close position
- `ISpotOrderClientIdRestClient`: spot order lookup/cancel by client order id
- `IFuturesOrderClientIdRestClient`: futures order lookup/cancel by client order id
- `ISpotTriggerOrderRestClient`: spot trigger orders
- `IFuturesTriggerOrderRestClient`: futures trigger orders
- `IFuturesTpSlRestClient`: futures take-profit/stop-loss support

Account and funding:

- `IBalanceRestClient`: account balances
- `IPositionRestClient`: positions where implemented separately from futures order client
- `IFeeRestClient`: fees
- `IDepositRestClient`: deposit addresses and deposit history
- `IWithdrawalRestClient`: withdrawal history
- `IWithdrawRestClient`: withdrawal creation
- `ITransferRestClient`: internal transfers
- `IListenKeyRestClient`: listen-key style user stream management

Futures-specific:

- `IFundingRateRestClient`: funding rate history
- `ILeverageRestClient`: get/set leverage
- `IPositionModeRestClient`: get/set position mode
- `IPositionHistoryRestClient`: position history
- `IOpenInterestRestClient`: open interest
- `IMarkPriceKlineRestClient`: mark price klines
- `IIndexPriceKlineRestClient`: index price klines

Not every exchange implements every interface. Code against the narrowest interface required by the workflow and handle unsupported-operation errors gracefully.

## Websocket Interface Families

Public streams:

- `ITickerSocketClient`: ticker updates
- `ITickersSocketClient`: all ticker updates
- `IBookTickerSocketClient`: best bid/ask updates
- `IOrderBookSocketClient`: order book updates
- `ITradeSocketClient`: public trade updates
- `IKlineSocketClient`: kline updates

Private streams:

- `ISpotOrderSocketClient`: spot order updates
- `IFuturesOrderSocketClient`: futures order updates
- `IUserTradeSocketClient`: user trade updates
- `IBalanceSocketClient`: balance updates
- `IPositionSocketClient`: position updates

Always close subscriptions on shutdown:

```csharp
ITickerSocketClient socketClient = new BinanceSocketClient().SpotApi.SharedClient;

var sub = await socketClient.SubscribeToTickerUpdatesAsync(
    new SubscribeTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")),
    update => Console.WriteLine($"[{socketClient.Exchange}] {update.Data.LastPrice}"));

if (!sub.Success)
{
    Console.WriteLine(sub.Error);
    return;
}

await sub.Data.CloseAsync();
```

When using the concrete exchange socket client, `await concreteSocket.UnsubscribeAsync(sub.Data)` is also commonly available.

## Multi-Exchange Concurrency

Use `Task.WhenAll` for independent cross-exchange calls.

```csharp
var clients = new List<ISpotTickerRestClient>
{
    new BinanceRestClient().SpotApi.SharedClient,
    new OKXRestClient().UnifiedApi.SharedClient,
    new BybitRestClient().V5Api.SharedClient,
};

var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");

var tasks = clients.Select(client => FetchTickerAsync(client, symbol)).ToArray();
var snapshots = (await Task.WhenAll(tasks)).Where(x => x != null).Cast<TickerSnapshot>();

foreach (var snapshot in snapshots.OrderByDescending(x => x.LastPrice))
{
    Console.WriteLine($"{snapshot.Exchange,-12} {snapshot.LastPrice}");
}

async Task<TickerSnapshot?> FetchTickerAsync(ISpotTickerRestClient client, SharedSymbol symbol)
{
    var result = await client.GetSpotTickerAsync(new GetTickerRequest(symbol));
    if (!result.Success)
    {
        Console.WriteLine($"[{client.Exchange}] {result.Error}");
        return null;
    }

    return new TickerSnapshot(
        client.Exchange,
        result.Data.Symbol,
        result.Data.LastPrice ?? 0m,
        result.Data.Volume);
}

record TickerSnapshot(string Exchange, string Symbol, decimal LastPrice, decimal Volume);
```

For periodic market data, prefer websocket interfaces when available instead of tight REST polling loops.

## Book Tickers And Arbitrage Scanners

Use `IBookTickerRestClient` or `IBookTickerSocketClient` for best bid/ask analysis. The 24h ticker is usually too coarse for spread calculations.

```csharp
async Task<Quote?> GetQuoteAsync(IBookTickerRestClient client, SharedSymbol symbol)
{
    var result = await client.GetBookTickerAsync(new GetBookTickerRequest(symbol));
    if (!result.Success || result.Data == null)
        return null;

    return new Quote(
        Exchange: client.Exchange,
        BidPrice: result.Data.BestBidPrice,
        AskPrice: result.Data.BestAskPrice);
}

record Quote(string Exchange, decimal BidPrice, decimal AskPrice);
```

Production arbitrage needs more than shared ticker calls: order book depth, fees, available balances, inventory on both venues, withdrawal/rebalance timing, order execution risk, and hard exposure limits.

## Orders Through SharedApis

Use `ISpotOrderRestClient` or `IFuturesOrderRestClient` when the user wants portable order code.

Guidance:

- Prefer limit orders in examples unless the user explicitly asks for market orders.
- Check exchange support for the requested order type and time-in-force.
- Use `SharedSymbol`, `SharedOrderSide`, shared request types, and shared order models.
- Do not pass Binance/OKX/Bybit-native enums or models to shared clients.
- Always check `Success` and surface `Error` to the caller.
- If a required order option is unavailable in SharedApis, switch to the exchange-specific client.

## Pagination

Some shared REST methods accept a `PageRequest? nextPageToken` parameter and return paginated data.

Pattern:

```csharp
PageRequest? nextPage = null;

do
{
    var result = await client.GetClosedSpotOrdersAsync(
        new GetClosedOrdersRequest(symbol),
        nextPage);

    if (!result.Success)
    {
        Console.WriteLine(result.Error);
        break;
    }

    foreach (var order in result.Data)
    {
        Console.WriteLine(order.Id);
    }

    nextPage = result.NextPageRequest;
}
while (nextPage != null);
```

If the exact paging behavior differs for the target method/version, inspect the shared interface in `CryptoExchange.Net/SharedApis/Interfaces/Rest`.

## Capability Checks

Before writing code that assumes a symbol or feature exists:

- Use symbol metadata interfaces such as `ISpotSymbolRestClient` or `IFuturesSymbolRestClient`.
- Check whether the specific exchange implements the interface you plan to use.
- Handle unsupported-operation responses as normal failed results.
- For a fixed exchange, inspect its `.SharedClient` implementation when uncertain.

## When To Use Native Exchange Clients

Use native clients instead of SharedApis when:

- The user asks for an exchange-specific endpoint.
- The required request option is not represented by the shared request type.
- The response needs exchange-specific fields not present on shared models.
- The workflow depends on exchange-specific order types, margin settings, listen-key details, or account modes.
- You need exact exchange behavior for production trading.

It is fine to mix approaches at the application level: use SharedApis for common cross-exchange flows and native clients for exchange-specific features. Do not mix shared request/model types inside a single method call.

## Known Client Surface Examples

- Binance REST: `new BinanceRestClient().SpotApi.SharedClient`
- Binance socket: `new BinanceSocketClient().SpotApi.SharedClient`
- OKX REST: `new OKXRestClient().UnifiedApi.SharedClient`
- OKX socket: `new OKXSocketClient().UnifiedApi.SharedClient`
- Bybit REST: `new BybitRestClient().V5Api.SharedClient`
- Bybit socket: `new BybitSocketClient().V5Api.SharedClient`
- Kraken REST: `new KrakenRestClient().SpotApi.SharedClient`
- Coinbase REST: `new CoinbaseRestClient().AdvancedTradeApi.SharedClient`

Confirm exact client roots from the exchange package when generating code for an exchange not listed here.

## Local Examples

When available in a sibling checkout, read:

- `../CryptoExchange.Net/Examples/ai-friendly/01-shared-clients-quickstart.cs`
- `../CryptoExchange.Net/Examples/ai-friendly/02-multi-exchange-tickers.cs`
- `../CryptoExchange.Net/Examples/ai-friendly/03-cross-exchange-arbitrage-skeleton.cs`
- `../Binance.Net/Examples/ai-friendly/04-multi-exchange.cs`
