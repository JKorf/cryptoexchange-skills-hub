# Bitfinex.Net Usage

Use these snippets as patterns. Keep generated code async, check `Success`, and prefer the current local examples in `../Bitfinex.Net/Examples/ai-friendly/` for complete compilable programs.

## Base Imports

```csharp
using Bitfinex.Net;
using Bitfinex.Net.Clients;
using Bitfinex.Net.Enums;
using CryptoExchange.Net.Objects;
```

Add `using CryptoExchange.Net.SharedApis;` only for shared cross-exchange code.

## Public Spot Ticker

```csharp
var client = new BitfinexRestClient();
var ticker = await client.ExchangeApi.ExchangeData.GetTickerAsync("tBTCUSD");

if (!ticker.Success)
{
    Console.WriteLine($"Failed to get ticker: {ticker.Error}");
    return;
}

Console.WriteLine($"BTC/USD last price: {ticker.Data.LastPrice}");
Console.WriteLine($"24h volume: {ticker.Data.Volume}");
```

## Bitfinex Symbols

Use Bitfinex-native symbols for exchange-specific clients:

- Trading pairs: `tBTCUSD`, `tETHUSD`
- Funding currencies: `fUSD`, `fBTC`
- Derivatives: `tBTCF0:USTF0`

Use unprefixed `SharedSymbol` values only for `CryptoExchange.Net.SharedApis`:

```csharp
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USD");
```

## Authenticated Wallets

```csharp
var client = new BitfinexRestClient(options =>
{
    options.ApiCredentials = new BitfinexCredentials("API_KEY", "API_SECRET");
});

var wallets = await client.ExchangeApi.Account.GetBalancesAsync();
if (!wallets.Success)
{
    Console.WriteLine($"Failed to get wallets: {wallets.Error}");
    return;
}

foreach (var wallet in wallets.Data.Where(x => x.Total > 0))
{
    Console.WriteLine($"{wallet.Type} {wallet.Asset}: total={wallet.Total}, available={wallet.Available}");
}
```

## Spot Exchange Limit Order

Use `OrderType.ExchangeLimit` for spot exchange-wallet orders. Use non-exchange order types, such as `OrderType.Limit`, for margin orders.

```csharp
var order = await client.ExchangeApi.Trading.PlaceOrderAsync(
    symbol: "tBTCUSD",
    side: OrderSide.Buy,
    type: OrderType.ExchangeLimit,
    quantity: 0.001m,
    price: 50000m);

if (!order.Success)
{
    Console.WriteLine($"Failed to place order: {order.Error}");
    return;
}

Console.WriteLine($"Placed order {order.Data.Data.Id}: {order.Data.Data.Status}");
```

Bitfinex write endpoints often wrap the changed entity under `Data.Data`; check the concrete model from the local source when printing details.

## Order Lookup And Cleanup

```csharp
var openOrders = await client.ExchangeApi.Trading.GetOpenOrdersAsync("tBTCUSD");
if (!openOrders.Success)
{
    Console.WriteLine($"Failed to get open orders: {openOrders.Error}");
    return;
}

var cancel = await client.ExchangeApi.Trading.CancelOrderAsync(orderId: order.Data.Data.Id);
if (!cancel.Success)
{
    Console.WriteLine($"Failed to cancel order: {cancel.Error}");
    return;
}
```

## Margin Positions

```csharp
var positions = await client.ExchangeApi.Trading.GetPositionsAsync();
if (!positions.Success)
{
    Console.WriteLine($"Failed to get positions: {positions.Error}");
    return;
}

foreach (var position in positions.Data)
{
    Console.WriteLine($"{position.Symbol}: quantity={position.Quantity}, pnl={position.ProfitLoss}, leverage={position.Leverage}");
}
```

Margin endpoints can represent real borrowed exposure. Avoid generating margin order placement unless the user explicitly asks for it.

## Public Funding Ticker

```csharp
var ticker = await client.ExchangeApi.ExchangeData.GetFundingTickerAsync("fUSD");
if (!ticker.Success)
{
    Console.WriteLine($"Failed to get funding ticker: {ticker.Error}");
    return;
}

Console.WriteLine($"fUSD last funding rate: {ticker.Data.LastPrice}");
```

## Authenticated Funding Offer

```csharp
var activeOffers = await client.GeneralApi.Funding.GetActiveFundingOffersAsync("fUSD");
if (!activeOffers.Success)
{
    Console.WriteLine($"Failed to get active funding offers: {activeOffers.Error}");
    return;
}

var offer = await client.GeneralApi.Funding.SubmitFundingOfferAsync(
    fundingOrderType: FundingOrderType.Limit,
    symbol: "fUSD",
    quantity: 100m,
    rate: 0.0002m,
    period: 2);

if (!offer.Success)
{
    Console.WriteLine($"Failed to submit funding offer: {offer.Error}");
    return;
}

Console.WriteLine($"Funding offer submitted: {offer.Data.Data.Id}");
```

Add cancellation or cleanup when an example submits a live funding offer.

## Public Websocket Subscription

```csharp
var socket = new BitfinexSocketClient();

var tickerSub = await socket.ExchangeApi.SubscribeToTickerUpdatesAsync(
    "tBTCUSD",
    update =>
    {
        Console.WriteLine($"BTC/USD ticker: {update.Data.LastPrice} volume={update.Data.Volume}");
    });

if (!tickerSub.Success)
{
    Console.WriteLine($"Failed to subscribe: {tickerSub.Error}");
    return;
}

await socket.UnsubscribeAsync(tickerSub.Data);
```

Keep handlers fast; push expensive processing to a queue or channel.

## Kline Websocket Subscription

```csharp
var klineSub = await socket.ExchangeApi.SubscribeToKlineUpdatesAsync(
    "tETHUSD",
    KlineInterval.OneMinute,
    update =>
    {
        var candle = update.Data.LastOrDefault();
        if (candle != null)
            Console.WriteLine($"ETH 1m close: {candle.ClosePrice}");
    });

if (!klineSub.Success)
{
    Console.WriteLine($"Failed to subscribe klines: {klineSub.Error}");
    return;
}

await socket.UnsubscribeAsync(klineSub.Data);
```

## Funding Websocket Subscription

```csharp
var fundingSub = await socket.ExchangeApi.SubscribeToFundingTickerUpdatesAsync(
    "fUSD",
    update => Console.WriteLine($"fUSD funding ticker: {update.Data.LastPrice}"));

if (!fundingSub.Success)
{
    Console.WriteLine($"Failed to subscribe funding ticker: {fundingSub.Error}");
    return;
}

await socket.UnsubscribeAsync(fundingSub.Data);
```

## Authenticated User Updates

`SubscribeToUserUpdatesAsync` exposes multiple callbacks for authenticated order, trade, wallet, position, funding, balance, and margin events. Use the exact overload from `IBitfinexSocketClientExchangeApi` or the local examples before generating full user-stream code.

```csharp
var socket = new BitfinexSocketClient(options =>
{
    options.ApiCredentials = new BitfinexCredentials("API_KEY", "API_SECRET");
});

var userSub = await socket.ExchangeApi.SubscribeToUserUpdatesAsync(
    walletHandler: update =>
    {
        foreach (var wallet in update.Data)
            Console.WriteLine($"{wallet.Type} {wallet.Asset}: total={wallet.Total}, available={wallet.Available}");
    });

if (!userSub.Success)
{
    Console.WriteLine($"Failed to subscribe user updates: {userSub.Error}");
    return;
}

await socket.UnsubscribeAsync(userSub.Data);
```

## SharedApis REST

Use this when the user wants exchange-agnostic code. Shared symbols do not use Bitfinex `t` or `f` prefixes.

```csharp
using Bitfinex.Net.Clients;
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new BitfinexRestClient().ExchangeApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USD");

var result = await tickers.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine($"[{tickers.Exchange}] Failed: {result.Error}");
    return;
}

Console.WriteLine($"[{tickers.Exchange}] {result.Data.Symbol}: {result.Data.LastPrice}");
```

## SharedApis Websocket

```csharp
var socket = new BitfinexSocketClient();
ITickerSocketClient tickers = socket.ExchangeApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USD");

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

REST methods return `HttpResult<T>`. Websocket subscription methods return `WebSocketResult<UpdateSubscription>`. Errors are normally returned in `Error`, not thrown.

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

        if (last.Error == null || !last.Error.IsTransient)
            return last;

        await Task.Delay(TimeSpan.FromMilliseconds(250 * Math.Pow(2, attempt)));
    }

    return last;
}

var ticker = await WithRetry(() => client.ExchangeApi.ExchangeData.GetTickerAsync("tBTCUSD"));
```

Common handling guidance:

- Retry only transient network, server, or rate-limit errors.
- Do not retry validation errors, permission errors, insufficient balance, or rejected orders blindly.
- Use `ct: cancellationToken` on REST methods when the calling application has cancellation.
- Use `options.RequestTimeout = TimeSpan.FromSeconds(10)` for per-request timeout configuration.

## Dependency Injection

```csharp
using Bitfinex.Net;

services.AddBitfinex(options =>
{
    options.Rest.ApiCredentials = new BitfinexCredentials("API_KEY", "API_SECRET");
    options.Socket.ApiCredentials = new BitfinexCredentials("API_KEY", "API_SECRET");
});
```

Inject `IBitfinexRestClient` and `IBitfinexSocketClient`, or follow the consuming project's existing interface pattern. `AddBitfinex` also registers shared REST and socket interfaces from `ExchangeApi.SharedClient`.

## Local Examples

When available, read:

- `../Bitfinex.Net/Examples/ai-friendly/01-spot-quickstart.cs`
- `../Bitfinex.Net/Examples/ai-friendly/02-margin-funding.cs`
- `../Bitfinex.Net/Examples/ai-friendly/03-websocket.cs`
- `../Bitfinex.Net/Examples/ai-friendly/04-multi-exchange.cs`
- `../Bitfinex.Net/Examples/ai-friendly/05-error-handling.cs`
