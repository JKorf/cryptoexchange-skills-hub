# DeepCoin.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../DeepCoin.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using DeepCoin.Net;
using DeepCoin.Net.Clients;
using DeepCoin.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Spot Ticker

```csharp
var client = new DeepCoinRestClient();

var tickers = await client.ExchangeApi.ExchangeData.GetTickersAsync(SymbolType.Spot);
if (!tickers.Success)
{
    Console.WriteLine($"Failed to get tickers: {tickers.Error}");
    return;
}

var ticker = tickers.Data.FirstOrDefault(x => x.Symbol == "ETH-USDT");
if (ticker == null)
{
    Console.WriteLine("ETH-USDT was not returned by DeepCoin spot tickers.");
    return;
}

Console.WriteLine($"ETH-USDT last price: {ticker.LastPrice}");
```

## DeepCoin Symbols

Use DeepCoin-native symbols for exchange-specific clients:

- Spot: `ETH-USDT`, `BTC-USDT`
- Swap/futures: `ETH-USDT-SWAP`, `BTC-USDT-SWAP`

Use `SharedSymbol` values only for `CryptoExchange.Net.SharedApis`:

```csharp
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USDT");
```

## Symbols, Klines, Order Book, Funding

```csharp
var spotSymbols = await client.ExchangeApi.ExchangeData.GetSymbolsAsync(SymbolType.Spot);
var swapSymbols = await client.ExchangeApi.ExchangeData.GetSymbolsAsync(SymbolType.Swap);

var candles = await client.ExchangeApi.ExchangeData.GetKlinesAsync(
    "ETH-USDT",
    KlineInterval.OneMinute,
    limit: 50);

var book = await client.ExchangeApi.ExchangeData.GetOrderBookAsync("ETH-USDT", depth: 25);
var funding = await client.ExchangeApi.ExchangeData.GetFundingRateAsync(ProductGroup.USDTMargined, "ETH-USDT-SWAP");
```

## Authenticated Balances

```csharp
var client = new DeepCoinRestClient(options =>
{
    options.ApiCredentials = new DeepCoinCredentials("API_KEY", "API_SECRET", "API_PASS");
});

var balances = await client.ExchangeApi.Account.GetBalancesAsync(SymbolType.Spot);
if (!balances.Success)
{
    Console.WriteLine($"Failed to get balances: {balances.Error}");
    return;
}

foreach (var balance in balances.Data.Where(b => b.Balance > 0))
    Console.WriteLine($"{balance.Asset}: {balance.AvailableBalance} available, {balance.FrozenBalance} frozen");
```

## Spot Limit Order

```csharp
var order = await client.ExchangeApi.Trading.PlaceOrderAsync(
    symbol: "ETH-USDT",
    side: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 0.01m,
    price: 2000m,
    tradeMode: TradeMode.Spot);

if (!order.Success)
{
    Console.WriteLine($"Failed to place order: {order.Error}");
    return;
}

var status = await client.ExchangeApi.Trading.GetOpenOrderAsync("ETH-USDT", order.Data.OrderId);
var cancel = await client.ExchangeApi.Trading.CancelOrderAsync("ETH-USDT", order.Data.OrderId);
```

For quote-currency sizing on supported spot orders, pass `quantityType: QuantityType.QuoteAsset`.

## Swap Leverage And Order

Swap/futures position and leverage calls affect account risk. Only generate write operations when explicitly requested.

```csharp
const string symbol = "ETH-USDT-SWAP";

var leverage = await client.ExchangeApi.Account.SetLeverageAsync(
    symbol,
    leverage: 5m,
    tradeMode: TradeMode.Cross,
    positionType: PositionType.Split);

if (!leverage.Success)
{
    Console.WriteLine($"Failed to set leverage: {leverage.Error}");
    return;
}

var openOrder = await client.ExchangeApi.Trading.PlaceOrderAsync(
    symbol: symbol,
    side: OrderSide.Buy,
    orderType: OrderType.Market,
    quantity: 0.1m,
    tradeMode: TradeMode.Cross,
    positionSide: PositionSide.Long);
```

## Swap Close Position

Use the opposite order side, same position side, the position id, and `reduceOnly: true` when closing a swap position.

```csharp
var positions = await client.ExchangeApi.Trading.GetPositionsAsync(SymbolType.Swap, symbol);
if (!positions.Success)
    return;

var position = positions.Data.FirstOrDefault(p => p.Size != 0 && p.PositionSide == PositionSide.Long);
if (position != null)
{
    var closeOrder = await client.ExchangeApi.Trading.PlaceOrderAsync(
        symbol: symbol,
        side: OrderSide.Sell,
        orderType: OrderType.Market,
        quantity: Math.Abs(position.Size),
        tradeMode: TradeMode.Cross,
        positionSide: PositionSide.Long,
        closePosId: position.PositionId,
        reduceOnly: true);
}
```

## Listen Key And Private User Stream

```csharp
var rest = new DeepCoinRestClient(options =>
{
    options.ApiCredentials = new DeepCoinCredentials("API_KEY", "API_SECRET", "API_PASS");
});

var socket = new DeepCoinSocketClient(options =>
{
    options.ApiCredentials = new DeepCoinCredentials("API_KEY", "API_SECRET", "API_PASS");
});

var listenKey = await rest.ExchangeApi.Account.StartUserStreamAsync();
if (!listenKey.Success)
{
    Console.WriteLine($"Failed to start user stream: {listenKey.Error}");
    return;
}

var userSub = await socket.ExchangeApi.SubscribeToUserDataUpdatesAsync(
    listenKey.Data.ListenKey,
    onOrderMessage: update => Console.WriteLine($"Order updates: {update.Data.Length}"),
    onBalanceMessage: update => Console.WriteLine($"Balance updates: {update.Data.Length}"),
    onPositionMessage: update => Console.WriteLine($"Position updates: {update.Data.Length}"));

if (userSub.Success)
    await socket.UnsubscribeAsync(userSub.Data);
```

The no-listen-key overload can acquire and renew a listen key automatically.

## Public Websocket Subscriptions

```csharp
var socket = new DeepCoinSocketClient();

var tickerSub = await socket.ExchangeApi.SubscribeToSymbolUpdatesAsync(
    "ETH-USDT",
    update => Console.WriteLine($"{update.Data.Symbol}: {update.Data.LastPrice}"));

if (!tickerSub.Success)
{
    Console.WriteLine($"Failed to subscribe ticker: {tickerSub.Error}");
    return;
}

await socket.UnsubscribeAsync(tickerSub.Data);
```

Keep handlers fast; push expensive processing to a queue or channel.

## Order Book Websocket Subscription

```csharp
var bookSub = await socket.ExchangeApi.SubscribeToOrderBookUpdatesAsync(
    "ETH-USDT",
    update =>
    {
        var bestBid = update.Data.Bids.FirstOrDefault();
        var bestAsk = update.Data.Asks.FirstOrDefault();
        Console.WriteLine($"Book seq {update.Data.SequenceNumber}: bid={bestBid?.Price} ask={bestAsk?.Price}");
    });
```

## SharedApis REST

Use this when the user wants exchange-agnostic code. Call discovery before relying on optional features.

```csharp
using DeepCoin.Net.Clients;
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient spot = new DeepCoinRestClient().ExchangeApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USDT");

var result = await spot.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (result.Success)
    Console.WriteLine($"[{spot.Exchange}] {result.Data.Symbol}: {result.Data.LastPrice}");
```

## SharedApis Websocket

```csharp
var socket = new DeepCoinSocketClient();
ITickerSocketClient tickers = socket.ExchangeApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USDT");

var sub = await tickers.SubscribeToTickerUpdatesAsync(
    new SubscribeTickerRequest(symbol),
    update => Console.WriteLine($"[{tickers.Exchange}] {update.Data.Symbol}: {update.Data.LastPrice}"));

if (!sub.Success)
{
    Console.WriteLine($"Subscribe failed: {sub.Error}");
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Shared socket interfaces do not expose `UnsubscribeAsync`; keep the concrete socket client and call `socket.UnsubscribeAsync(sub.Data)`.

## Error Handling And Retry

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Shared non-I/O helpers can return `ExchangeCallResult<T>`.

```csharp
async Task<HttpResult<T>> WithRetry<T>(
    Func<Task<HttpResult<T>>> call,
    int maxAttempts = 3)
{
    HttpResult<T> last = default!;

    for (var attempt = 1; attempt <= maxAttempts; attempt++)
    {
        last = await call();
        if (last.Success)
            return last;

        if (last.Error?.IsTransient != true)
            return last;

        await Task.Delay(TimeSpan.FromMilliseconds(250 * Math.Pow(2, attempt)));
    }

    return last;
}

var tickers = await WithRetry(() => client.ExchangeApi.ExchangeData.GetTickersAsync(SymbolType.Spot));
```

Common handling guidance:

- Retry only transient network, server, or rate-limit errors.
- Do not retry validation errors, permission errors, insufficient balance, bad passphrase/signature, bad symbol, or rejected orders blindly.
- Use `ct: cancellationToken` on REST methods when the calling application has cancellation.
- Use `options.RequestTimeout = TimeSpan.FromSeconds(10)` for per-request timeout configuration.

## Dependency Injection

```csharp
using DeepCoin.Net;

services.AddDeepCoin(options =>
{
    options.ApiCredentials = new DeepCoinCredentials("API_KEY", "API_SECRET", "API_PASS");
});
```

Inject `IDeepCoinRestClient` and `IDeepCoinSocketClient`, or follow the consuming project's existing interface pattern. `AddDeepCoin` registers shared REST and socket interfaces from `ExchangeApi.SharedClient`.

## Local Examples

When available, read:

- `../DeepCoin.Net/Examples/ai-friendly/01-spot-quickstart.cs`
- `../DeepCoin.Net/Examples/ai-friendly/02-futures.cs`
- `../DeepCoin.Net/Examples/ai-friendly/03-websocket.cs`
- `../DeepCoin.Net/Examples/ai-friendly/04-multi-exchange.cs`
- `../DeepCoin.Net/Examples/ai-friendly/05-error-handling.cs`
