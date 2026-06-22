# CryptoCom.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../CryptoCom.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using CryptoCom.Net;
using CryptoCom.Net.Clients;
using CryptoCom.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Ticker

```csharp
var client = new CryptoComRestClient();
const string symbol = "BTC_USDT";

var tickers = await client.ExchangeApi.ExchangeData.GetTickersAsync(symbol);
if (!tickers.Success)
{
    Console.WriteLine($"Failed to get ticker: {tickers.Error}");
    return;
}

var ticker = tickers.Data.FirstOrDefault();
Console.WriteLine($"{ticker?.Symbol}: {ticker?.LastPrice}");
```

## Symbols And Precision

Fetch symbols before trading. Use exact instrument names and tick sizes from the exchange.

```csharp
var symbols = await client.ExchangeApi.ExchangeData.GetSymbolsAsync();
if (!symbols.Success)
    return;

var btcusdt = symbols.Data.FirstOrDefault(s => s.Symbol == "BTC_USDT");
if (btcusdt == null)
    return;

var validQuantity = btcusdt.QuantityTickSize > 0
    ? Math.Floor(0.00123456m / btcusdt.QuantityTickSize) * btcusdt.QuantityTickSize
    : 0.00123456m;
```

## Crypto.com Symbols

Use Crypto.com-native symbols for exchange-specific clients:

- Spot: `BTC_USDT`, `ETH_USDT`
- Perpetual/derivatives: examples use `ETHUSD_PERP`
- Staking: examples use symbols such as `SOL.staked`, `ETH.staked`, or `CDCETH`

Use `SharedSymbol` values only for `CryptoExchange.Net.SharedApis`:

```csharp
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");
```

## Authenticated Balances

```csharp
var client = new CryptoComRestClient(options =>
{
    options.ApiCredentials = new CryptoComCredentials("API_KEY", "API_SECRET");
});

var balances = await client.ExchangeApi.Account.GetBalancesAsync();
if (!balances.Success)
{
    Console.WriteLine($"Failed to get balances: {balances.Error}");
    return;
}

foreach (var accountBalance in balances.Data)
{
    foreach (var balance in accountBalance.PositionBalances.Where(b => b.Quantity != 0))
        Console.WriteLine($"{balance.Asset}: {balance.Quantity}");
}
```

## Spot Limit Order

```csharp
var order = await client.ExchangeApi.Trading.PlaceOrderAsync(
    symbol: "BTC_USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    price: 50000m);

if (!order.Success)
{
    Console.WriteLine($"Failed to place order: {order.Error}");
    return;
}

var status = await client.ExchangeApi.Trading.GetOrderAsync(orderId: order.Data.OrderId);
var cancel = await client.ExchangeApi.Trading.CancelOrderAsync(orderId: order.Data.OrderId);
```

Common variations:

- Market order: `type: OrderType.Market`, omit `price`
- Trigger order: add `triggerPrice` and `triggerPriceType`
- Spot margin: pass `margin: true`
- Isolated margin: pass `isolatedMargin: true`, `leverage`, and optional `isolationId`
- Reduce-only derivatives: pass `reduceOnly: true`

## Derivative Position And Close

CryptoCom.Net uses `ExchangeApi.Trading` for derivatives; there is no separate futures root.

```csharp
const string symbol = "ETHUSD_PERP";

var positions = await client.ExchangeApi.Trading.GetPositionsAsync(symbol);
if (!positions.Success)
{
    Console.WriteLine($"Failed to get positions: {positions.Error}");
    return;
}

var close = await client.ExchangeApi.Trading.ClosePositionAsync(
    symbol: symbol,
    orderType: OrderType.Market);
```

## Staking

Staking writes can lock or move funds. Prefer read-only examples unless the user explicitly asks to stake, unstake, or convert.

```csharp
var symbols = await client.ExchangeApi.Staking.GetStakingSymbolsAsync();
var positions = await client.ExchangeApi.Staking.GetStakingPositionsAsync("SOL.staked");
var rewards = await client.ExchangeApi.Staking.GetStakingRewardHistoryAsync("SOL.staked");
```

## Account Funding Actions

Withdrawals and isolated margin transfers move funds. Only generate write calls when explicitly requested.

```csharp
var assets = await client.ExchangeApi.Account.GetAssetsAsync();
var deposits = await client.ExchangeApi.Account.GetDepositHistoryAsync("USDT");
var withdrawals = await client.ExchangeApi.Account.GetWithdrawalHistoryAsync("USDT");
var addresses = await client.ExchangeApi.Account.GetDepositAddressesAsync("USDT");
```

## Public Websocket Subscriptions

```csharp
var socket = new CryptoComSocketClient();

var tickerSub = await socket.ExchangeApi.SubscribeToTickerUpdatesAsync(
    "BTC_USDT",
    update => Console.WriteLine($"{update.Data.Symbol}: {update.Data.LastPrice}"));

if (!tickerSub.Success)
{
    Console.WriteLine($"Failed to subscribe ticker: {tickerSub.Error}");
    return;
}

await socket.UnsubscribeAsync(tickerSub.Data);
```

Keep handlers fast; push expensive processing to a queue or channel.

## Order Book And Kline Subscriptions

```csharp
var bookSub = await socket.ExchangeApi.SubscribeToOrderBookUpdatesAsync(
    "BTC_USDT",
    depth: 50,
    update =>
    {
        var bestBid = update.Data.Bids.FirstOrDefault();
        var bestAsk = update.Data.Asks.FirstOrDefault();
        Console.WriteLine($"Book bid={bestBid?.Price} ask={bestAsk?.Price}");
    });

var klineSub = await socket.ExchangeApi.SubscribeToKlineUpdatesAsync(
    "BTC_USDT",
    KlineInterval.OneMinute,
    update => Console.WriteLine($"Received {update.Data.Length} candle update(s)"));
```

## Authenticated Websocket Updates

Private streams require credentials on the socket client.

```csharp
var socket = new CryptoComSocketClient(options =>
{
    options.ApiCredentials = new CryptoComCredentials("API_KEY", "API_SECRET");
});

var orderSub = await socket.ExchangeApi.SubscribeToOrderUpdatesAsync(update =>
{
    foreach (var order in update.Data)
        Console.WriteLine($"Order {order.OrderId} {order.Symbol}: {order.Status}");
});

if (!orderSub.Success)
{
    Console.WriteLine($"Failed to subscribe order updates: {orderSub.Error}");
    return;
}

await socket.UnsubscribeAsync(orderSub.Data);
```

## Socket API Requests

Use REST request methods by default. Use socket request methods only when the user specifically wants request/response trading over the websocket connection. These return `QueryResult<T>` or `QueryResult`.

```csharp
var socketOrder = await socket.ExchangeApi.PlaceOrderAsync(
    symbol: "BTC_USDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.001m,
    price: 50000m);

if (!socketOrder.Success)
{
    Console.WriteLine($"Socket order failed: {socketOrder.Error}");
    return;
}
```

## SharedApis REST

Use this when the user wants exchange-agnostic code. Call discovery before relying on optional features.

```csharp
using CryptoCom.Net.Clients;
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient spot = new CryptoComRestClient().ExchangeApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");

var result = await spot.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (result.Success)
    Console.WriteLine($"[{spot.Exchange}] {result.Data.Symbol}: {result.Data.LastPrice}");
```

## SharedApis Websocket

```csharp
var socket = new CryptoComSocketClient();
ITickerSocketClient tickers = socket.ExchangeApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");

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

REST methods return `HttpResult<T>` or `HttpResult`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Socket API request methods return `QueryResult<T>` or `QueryResult`. Shared non-I/O helpers can return `ExchangeCallResult<T>`. Batch order methods can return nested `CallResult<T>` entries; inspect each item after the outer result succeeds.

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

var ticker = await WithRetry(() => client.ExchangeApi.ExchangeData.GetTickersAsync("BTC_USDT"));
```

Common handling guidance:

- Retry only transient network, server, or rate-limit errors.
- Do not retry validation errors, permission errors, insufficient balance, bad precision, unknown order, or rejected orders blindly.
- Use `ct: cancellationToken` on REST and socket request methods when the calling application has cancellation.
- Use `options.RequestTimeout = TimeSpan.FromSeconds(10)` for per-request timeout configuration.

## Dependency Injection

```csharp
using CryptoCom.Net;

services.AddCryptoCom(options =>
{
    options.ApiCredentials = new CryptoComCredentials("API_KEY", "API_SECRET");
});
```

Inject `ICryptoComRestClient` and `ICryptoComSocketClient`, or follow the consuming project's existing interface pattern. `AddCryptoCom` registers shared REST and socket interfaces from `ExchangeApi.SharedClient`.

## Local Examples

When available, read:

- `../CryptoCom.Net/Examples/ai-friendly/01-exchange-quickstart.cs`
- `../CryptoCom.Net/Examples/ai-friendly/02-derivatives.cs`
- `../CryptoCom.Net/Examples/ai-friendly/03-websocket.cs`
- `../CryptoCom.Net/Examples/ai-friendly/04-multi-exchange.cs`
- `../CryptoCom.Net/Examples/ai-friendly/05-error-handling.cs`
