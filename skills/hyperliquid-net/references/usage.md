# HyperLiquid.Net Usage

Use these snippets as compact patterns for HyperLiquid-specific code. They assume:

```csharp
using HyperLiquid.Net;
using HyperLiquid.Net.Clients;
using HyperLiquid.Net.Enums;
using CryptoExchange.Net.Objects;
```

## Public Spot Market Data

```csharp
var client = new HyperLiquidRestClient();

var prices = await client.SpotApi.ExchangeData.GetPricesAsync();
if (!prices.Success)
{
    Console.WriteLine($"Prices failed: {prices.Error}");
    return;
}

Console.WriteLine($"HYPE/USDC mid: {prices.Data["HYPE/USDC"]}");
```

## Authenticated Client

```csharp
var client = new HyperLiquidRestClient(options =>
{
    options.ApiCredentials = new HyperLiquidCredentials(
        Environment.GetEnvironmentVariable("HYPERLIQUID_PUBLIC_ADDRESS")!,
        Environment.GetEnvironmentVariable("HYPERLIQUID_PRIVATE_KEY")!);
});
```

The first credential value is the public address; the second is the ECDSA private key.

## Builder Fee Check

```csharp
var approved = await client.SpotApi.Account.GetApprovedBuilderFeeAsync();
if (!approved.Success)
{
    Console.WriteLine($"Builder fee check failed: {approved.Error}");
    return;
}

Console.WriteLine($"Approved builder fee: {approved.Data}");
```

To disable the default 1 bps builder fee:

```csharp
var client = new HyperLiquidRestClient(options =>
{
    options.ApiCredentials = new HyperLiquidCredentials("PUBLIC_ADDRESS", "PRIVATE_KEY");
    options.BuilderFeePercentage = 0;
});
```

## Spot Balances

```csharp
var balances = await client.SpotApi.Account.GetBalancesAsync();
if (!balances.Success)
{
    Console.WriteLine($"Balances failed: {balances.Error}");
    return;
}

foreach (var balance in balances.Data.Where(x => x.Total > 0))
    Console.WriteLine($"{balance.Asset}: total={balance.Total}, hold={balance.Hold}");
```

## Spot Limit Order

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "HYPE/USDC",
    side: OrderSide.Buy,
    orderType: OrderType.Limit,
    quantity: 1m,
    price: 10m,
    timeInForce: TimeInForce.GoodTillCanceled);

if (!order.Success)
{
    Console.WriteLine($"Order failed: {order.Error}");
    return;
}

Console.WriteLine($"Placed order {order.Data.OrderId}");
```

Cancel cleanup:

```csharp
var cancel = await client.SpotApi.Trading.CancelOrderAsync("HYPE/USDC", order.Data.OrderId);
```

## Futures Metadata And Price

```csharp
var tickers = await client.FuturesApi.ExchangeData.GetExchangeInfoAndTickersAsync();
if (!tickers.Success)
{
    Console.WriteLine($"Futures tickers failed: {tickers.Error}");
    return;
}

var ethTicker = tickers.Data.Tickers.Single(x => x.Symbol == "ETH");
Console.WriteLine($"ETH mid: {ethTicker.MidPrice}");
```

## Futures Leverage And Market Order

```csharp
const string symbol = "ETH";

var leverage = await client.FuturesApi.Trading.SetLeverageAsync(
    symbol: symbol,
    leverage: 5,
    marginType: MarginType.Cross);

if (!leverage.Success)
{
    Console.WriteLine($"Set leverage failed: {leverage.Error}");
    return;
}

var prices = await client.FuturesApi.ExchangeData.GetPricesAsync();
if (!prices.Success)
    return;

var order = await client.FuturesApi.Trading.PlaceOrderAsync(
    symbol: symbol,
    side: OrderSide.Buy,
    orderType: OrderType.Market,
    quantity: 0.01m,
    price: prices.Data[symbol]);
```

Market orders still require a recent price for max slippage calculation.

## Close A Futures Position

```csharp
var account = await client.FuturesApi.Account.GetAccountInfoAsync();
if (!account.Success)
    return;

var position = account.Data.Positions
    .Select(x => x.Position)
    .FirstOrDefault(x => x.Symbol == "ETH" && x.PositionQuantity != 0);

if (position == null)
    return;

var close = await client.FuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETH",
    side: position.PositionQuantity > 0 ? OrderSide.Sell : OrderSide.Buy,
    orderType: OrderType.Market,
    quantity: Math.Abs(position.PositionQuantity ?? 0),
    price: position.AverageEntryPrice ?? 3000m,
    reduceOnly: true);
```

Use `reduceOnly: true` for close-position examples to avoid flipping exposure.

## Trigger Or TP/SL Order

```csharp
var stop = await client.FuturesApi.Trading.PlaceOrderAsync(
    symbol: "ETH",
    side: OrderSide.Sell,
    orderType: OrderType.StopMarket,
    quantity: 0.01m,
    price: 3000m,
    reduceOnly: true,
    triggerPrice: 2950m,
    tpSlType: TpSlType.StopLoss);
```

Use `tpSlGrouping` when the user needs grouped TP/SL behavior.

## Transfers And Withdrawals

```csharp
var internalTransfer = await client.SpotApi.Account.TransferInternalAsync(
    direction: TransferDirection.SpotToFutures,
    quantity: 25m);

var spotTransfer = await client.SpotApi.Account.TransferSpotAsync(
    destinationAddress: "0x0000000000000000000000000000000000000000",
    asset: "HYPE",
    quantity: 1m);
```

Generate USD transfer, withdrawal, staking, vault, or validator-delegation code only when explicitly requested.

## Public Websocket Subscription

```csharp
var socket = new HyperLiquidSocketClient();

var sub = await socket.FuturesApi.ExchangeData.SubscribeToSymbolUpdatesAsync(
    "ETH",
    update => Console.WriteLine(update.Data.MidPrice));

if (!sub.Success)
{
    Console.WriteLine($"Subscribe failed: {sub.Error}");
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Subscription methods return `WebSocketResult<UpdateSubscription>`.

## Socket Query

```csharp
var socket = new HyperLiquidSocketClient();

var prices = await socket.FuturesApi.ExchangeData.GetPricesAsync();
if (!prices.Success)
{
    Console.WriteLine($"Socket query failed: {prices.Error}");
    return;
}
```

Socket query and order methods return `QueryResult<T>`, not `HttpResult<T>`.

## Authenticated Websocket Subscription

```csharp
var socket = new HyperLiquidSocketClient(options =>
{
    options.ApiCredentials = new HyperLiquidCredentials("PUBLIC_ADDRESS", "PRIVATE_KEY");
});

var sub = await socket.FuturesApi.Trading.SubscribeToOrderUpdatesAsync(
    address: null,
    onMessage: update =>
    {
        foreach (var order in update.Data)
            Console.WriteLine($"{order.Order.OrderId}: {order.Status}");
    });
```

When `address` is `null`, authenticated streams use the address from credentials.

## SharedApis Futures Ticker

```csharp
using CryptoExchange.Net.SharedApis;

IFuturesTickerRestClient tickers = new HyperLiquidRestClient().FuturesApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.PerpetualLinear, "ETH", "USDC");

var result = await tickers.GetFuturesTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

Use native HyperLiquid APIs for builder fees, vaults, staking, HIP-3 DEX, and detailed TWAP/trigger behavior.

## Dependency Injection

```csharp
services.AddHyperLiquid(options =>
{
    options.ApiCredentials = new HyperLiquidCredentials("PUBLIC_ADDRESS", "PRIVATE_KEY");
});
```

Inject `IHyperLiquidRestClient` and `IHyperLiquidSocketClient`, or specific registered interfaces already used in the application.

## Error Handling

```csharp
var result = await client.FuturesApi.ExchangeData.GetPricesAsync();
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
