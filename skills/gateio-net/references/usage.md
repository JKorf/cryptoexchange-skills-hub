# GateIo.Net Usage

Use these snippets as compact patterns for Gate.io-specific code. They assume:

```csharp
using GateIo.Net;
using GateIo.Net.Clients;
using GateIo.Net.Enums;
using GateIo.Net.Objects;
using CryptoExchange.Net.Objects;
```

## Public Spot Market Data

```csharp
var client = new GateIoRestClient();

var tickerResult = await client.SpotApi.ExchangeData.GetTickersAsync("ETH_USDT");
if (!tickerResult.Success)
{
    Console.WriteLine($"Ticker failed: {tickerResult.Error}");
    return;
}

var ticker = tickerResult.Data.First();
Console.WriteLine($"ETH/USDT last: {ticker.LastPrice}");
```

## Authenticated Client

```csharp
var client = new GateIoRestClient(options =>
{
    options.ApiCredentials = new GateIoCredentials(
        Environment.GetEnvironmentVariable("GATEIO_API_KEY")!,
        Environment.GetEnvironmentVariable("GATEIO_API_SECRET")!);
});
```

Use `GateIoCredentials("API_KEY", "API_SECRET")`. GateIo.Net does not require an API passphrase.

## Balances

```csharp
var balances = await client.SpotApi.Account.GetBalancesAsync();
if (!balances.Success)
{
    Console.WriteLine($"Balances failed: {balances.Error}");
    return;
}

foreach (var balance in balances.Data.Where(x => x.Available + x.Locked > 0))
    Console.WriteLine($"{balance.Asset}: {balance.Available} available, {balance.Locked} locked");
```

## Spot Limit Order

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "ETH_USDT",
    side: OrderSide.Buy,
    type: NewOrderType.Limit,
    quantity: 0.01m,
    price: 2000m,
    timeInForce: TimeInForce.GoodTillCancel);

if (!order.Success)
{
    Console.WriteLine($"Order failed: {order.Error}");
    return;
}

Console.WriteLine($"Placed order {order.Data.Id}");
```

For market spot buys, Gate.io uses quote-asset quantity. Be explicit about that in generated code.

## Spot Trigger Order

```csharp
var trigger = await client.SpotApi.Trading.PlaceTriggerOrderAsync(
    symbol: "ETH_USDT",
    orderSide: OrderSide.Buy,
    orderType: NewOrderType.Limit,
    triggerType: TriggerType.EqualOrHigher,
    triggerPrice: 2100m,
    expiration: TimeSpan.FromHours(1),
    quantity: 0.01m,
    accountType: TriggerAccountType.Normal,
    timeInForce: TimeInForce.GoodTillCancel,
    orderPrice: 2101m);
```

Check the returned `HttpResult` before using the trigger order id.

## Transfers And Withdrawals

```csharp
var transfer = await client.SpotApi.Account.TransferAsync(
    asset: "USDT",
    from: AccountType.Spot,
    to: AccountType.Futures,
    quantity: 25m,
    settleAsset: "usdt");

var depositAddress = await client.SpotApi.Account.GenerateDepositAddressAsync("USDT");
```

Generate withdrawal code only when explicitly requested. Include address, network, memo/tag, and amount validation.

## Perpetual Futures Metadata

```csharp
const string settlementAsset = "usdt";
const string contract = "ETH_USDT";

var contractInfo = await client.PerpetualFuturesApi.ExchangeData.GetContractAsync(
    settlementAsset,
    contract);

if (!contractInfo.Success)
{
    Console.WriteLine($"Contract lookup failed: {contractInfo.Error}");
    return;
}

Console.WriteLine($"{contract} multiplier: {contractInfo.Data.Multiplier}");
```

Futures order quantities are integer contract counts. Do not assume a `decimal` base-asset quantity.

## Perpetual Futures Leverage And Order

```csharp
const string settlementAsset = "usdt";
const string contract = "ETH_USDT";

var leverage = await client.PerpetualFuturesApi.Trading.UpdatePositionLeverageAsync(
    settlementAsset,
    contract,
    leverage: 5);

if (!leverage.Success)
{
    Console.WriteLine($"Set leverage failed: {leverage.Error}");
    return;
}

var order = await client.PerpetualFuturesApi.Trading.PlaceOrderAsync(
    settlementAsset: settlementAsset,
    contract: contract,
    orderSide: OrderSide.Buy,
    quantity: 1,
    price: 0m,
    timeInForce: TimeInForce.ImmediateOrCancel);

if (!order.Success)
{
    Console.WriteLine($"Futures order failed: {order.Error}");
    return;
}
```

Gate.io uses `price: 0m` with `TimeInForce.ImmediateOrCancel` as the market-style futures pattern.

## Close A Futures Position

```csharp
var position = await client.PerpetualFuturesApi.Trading.GetPositionAsync("usdt", "ETH_USDT");
if (!position.Success || position.Data.Size == 0)
    return;

var closeOrder = await client.PerpetualFuturesApi.Trading.PlaceOrderAsync(
    settlementAsset: "usdt",
    contract: "ETH_USDT",
    orderSide: position.Data.Size > 0 ? OrderSide.Sell : OrderSide.Buy,
    quantity: (int)Math.Abs(position.Data.Size),
    price: 0m,
    reduceOnly: true,
    timeInForce: TimeInForce.ImmediateOrCancel);
```

Use `reduceOnly: true` for close examples to avoid flipping the position.

## Public Websocket Subscription

```csharp
var socket = new GateIoSocketClient();

var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "ETH_USDT",
    update => Console.WriteLine(update.Data.LastPrice));

if (!sub.Success)
{
    Console.WriteLine($"Subscribe failed: {sub.Error}");
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Subscription methods return `WebSocketResult<UpdateSubscription>`.

## Futures Websocket Subscription

```csharp
var sub = await socket.PerpetualFuturesApi.SubscribeToTickerUpdatesAsync(
    "usdt",
    "ETH_USDT",
    update =>
    {
        var first = update.Data.FirstOrDefault();
        if (first != null)
            Console.WriteLine(first.LastPrice);
    });
```

Most futures socket subscriptions take `settlementAsset` before contract.

## Socket Order Request

```csharp
var socket = new GateIoSocketClient(options =>
{
    options.ApiCredentials = new GateIoCredentials("API_KEY", "API_SECRET");
});

var order = await socket.SpotApi.PlaceOrderAsync(
    symbol: "ETH_USDT",
    side: OrderSide.Buy,
    type: NewOrderType.Limit,
    quantity: 0.01m,
    price: 2000m,
    timeInForce: TimeInForce.GoodTillCancel);

if (!order.Success)
{
    Console.WriteLine($"Socket order failed: {order.Error}");
    return;
}
```

Socket order request methods return `QueryResult<T>`, not `HttpResult<T>`.

## SharedApis Spot Ticker

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new GateIoRestClient().SpotApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USDT");

var result = await tickers.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

Use native GateIo APIs for Gate.io-only details such as Alpha, rebate, unified account, and exchange-specific trigger parameters.

## Dependency Injection

```csharp
services.AddGateIo(options =>
{
    options.ApiCredentials = new GateIoCredentials("API_KEY", "API_SECRET");
});
```

Inject `IGateIoRestClient` and `IGateIoSocketClient`, or specific registered interfaces already used in the application.

## Error Handling

```csharp
var result = await client.SpotApi.ExchangeData.GetTickersAsync("ETH_USDT");
if (!result.Success)
{
    if (result.Error?.IsTransient == true)
    {
        // Retry with bounded backoff.
    }

    Console.WriteLine($"{result.Error?.Code}: {result.Error?.Message}");
    return;
}
```

REST failures, API validation errors, and rate limits are returned on `.Error`. Do not rely on exceptions for normal API failures.
