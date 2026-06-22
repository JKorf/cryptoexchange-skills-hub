# HTX.Net Usage

Use these snippets as compact patterns for HTX-specific code. They assume:

```csharp
using HTX.Net;
using HTX.Net.Clients;
using HTX.Net.Enums;
using CryptoExchange.Net.Objects;
```

## Public Spot Market Data

```csharp
var client = new HTXRestClient();

var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("ETHUSDT");
if (!ticker.Success)
{
    Console.WriteLine($"Ticker failed: {ticker.Error}");
    return;
}

Console.WriteLine($"ETH/USDT close: {ticker.Data.ClosePrice}");
```

## Authenticated Client

```csharp
var client = new HTXRestClient(options =>
{
    options.ApiCredentials = new HTXCredentials(
        Environment.GetEnvironmentVariable("HTX_API_KEY")!,
        Environment.GetEnvironmentVariable("HTX_API_SECRET")!);
});
```

For Ed25519 credentials:

```csharp
var credentials = new HTXCredentials().WithEd25519("API_KEY", "PRIVATE_KEY");
```

## Spot Account Id And Balances

```csharp
var accounts = await client.SpotApi.Account.GetAccountsAsync();
if (!accounts.Success)
{
    Console.WriteLine($"Accounts failed: {accounts.Error}");
    return;
}

var spotAccount = accounts.Data.FirstOrDefault();
if (spotAccount == null)
{
    Console.WriteLine("No HTX account found.");
    return;
}

var balances = await client.SpotApi.Account.GetBalancesAsync(spotAccount.Id);
if (!balances.Success)
{
    Console.WriteLine($"Balances failed: {balances.Error}");
    return;
}
```

HTX spot balances and spot order placement require an account id.

## Spot Limit Order

```csharp
var order = await client.SpotApi.Trading.PlaceOrderAsync(
    accountId: spotAccount.Id,
    symbol: "ETHUSDT",
    side: OrderSide.Buy,
    type: OrderType.Limit,
    quantity: 0.01m,
    price: 2000m);

if (!order.Success)
{
    Console.WriteLine($"Order failed: {order.Error}");
    return;
}

Console.WriteLine($"Placed order {order.Data}");
```

Cancel cleanup:

```csharp
var cancel = await client.SpotApi.Trading.CancelOrderAsync(order.Data);
```

## Spot Stop Or Conditional Order

For a stop-limit order on the normal spot order endpoint:

```csharp
var stopOrder = await client.SpotApi.Trading.PlaceOrderAsync(
    accountId: spotAccount.Id,
    symbol: "ETHUSDT",
    side: OrderSide.Buy,
    type: OrderType.StopLimit,
    quantity: 0.01m,
    price: 1990m,
    stopPrice: 2000m,
    stopOperator: Operator.GreaterThanOrEqual);
```

For HTX conditional orders, use `PlaceConditionalOrderAsync(...)` and manage them with the conditional-order methods.

## Transfers And Withdrawals

```csharp
var transfer = await client.SpotApi.Account.TransferAsync(
    fromAccount: TransferAccount.Spot,
    toAccount: TransferAccount.LinearSwap,
    asset: "USDT",
    quantity: 25m,
    marginAccount: "USDT");

var addresses = await client.SpotApi.Account.GetDepositAddressesAsync("USDT");
```

Generate withdrawal code only when explicitly requested. Include address, network, address tag, fee, and amount validation.

## USDT Futures Metadata

```csharp
const string contractCode = "ETH-USDT";

var contract = await client.UsdtFuturesApi.ExchangeData.GetContractsAsync(contractCode);
if (!contract.Success)
{
    Console.WriteLine($"Contract lookup failed: {contract.Error}");
    return;
}
```

USDT futures contract codes use hyphens, while spot symbols do not.

## USDT Futures Cross-Margin Order

```csharp
const string contractCode = "ETH-USDT";

var leverage = await client.UsdtFuturesApi.Trading.SetCrossMarginLeverageAsync(
    leverageRate: 5,
    contractCode: contractCode);

if (!leverage.Success)
{
    Console.WriteLine($"Set leverage failed: {leverage.Error}");
    return;
}

var order = await client.UsdtFuturesApi.Trading.PlaceCrossMarginOrderAsync(
    quantity: 1,
    side: OrderSide.Buy,
    leverageRate: 5,
    orderPriceType: OrderPriceType.Market,
    contractCode: contractCode,
    offset: Offset.Open);
```

Use `Offset.Close` and/or `reduceOnly: true` when closing through order placement.

## USDT Futures Isolated-Margin Order

```csharp
const string contractCode = "ETH-USDT";

await client.UsdtFuturesApi.Trading.SetIsolatedMarginLeverageAsync(
    contractCode,
    leverageRate: 5);

var order = await client.UsdtFuturesApi.Trading.PlaceIsolatedMarginOrderAsync(
    contractCode: contractCode,
    quantity: 1,
    side: OrderSide.Buy,
    leverageRate: 5,
    orderPriceType: OrderPriceType.Market,
    offset: Offset.Open);
```

Cross and isolated margin have separate method names. Do not swap them.

## Close A Futures Position

```csharp
var close = await client.UsdtFuturesApi.Trading.CloseCrossMarginPositionAsync(
    direction: OrderSide.Sell,
    contractCode: "ETH-USDT",
    orderPriceType: LightningPriceType.Market);
```

For isolated margin, use `CloseIsolatedMarginPositionAsync(contractCode, direction, ...)`.

## Futures V5 Order

```csharp
var order = await client.UsdtFuturesV5Api.Trading.PlaceOrderAsync(
    contractCode: "ETH-USDT",
    marginMode: MarginMode.Cross,
    side: OrderSide.Buy,
    type: OrderTypeV5.Market,
    quantity: 1m,
    positionSide: FuturesPositionSide.Long,
    reduceOnly: false);
```

Use `UsdtFuturesV5Api` for native V5 features. Use `UsdtFuturesApi.SharedClient` for SharedApis futures portability.

## Public Websocket Subscription

```csharp
var socket = new HTXSocketClient();

var sub = await socket.SpotApi.SubscribeToTickerUpdatesAsync(
    "ETHUSDT",
    update => Console.WriteLine(update.Data.ClosePrice));

if (!sub.Success)
{
    Console.WriteLine($"Subscribe failed: {sub.Error}");
    return;
}

await socket.UnsubscribeAsync(sub.Data);
```

Subscription methods return `WebSocketResult<UpdateSubscription>`.

## Spot Socket Query

```csharp
var klines = await socket.SpotApi.GetKlinesAsync("ETHUSDT", KlineInterval.OneMinute);
if (!klines.Success)
{
    Console.WriteLine($"Socket query failed: {klines.Error}");
    return;
}
```

Spot socket query and socket order methods return `QueryResult<T>`.

## Futures Websocket Subscription

```csharp
var sub = await socket.UsdtFuturesApi.SubscribeToTickerUpdatesAsync(
    "ETH-USDT",
    update => Console.WriteLine(update.Data.ClosePrice));
```

Authenticated futures streams distinguish margin modes:

```csharp
var orders = await socket.UsdtFuturesApi.SubscribeToOrderUpdatesAsync(
    MarginMode.Cross,
    update => Console.WriteLine(update.Data.OrderId));
```

## SharedApis Spot Ticker

```csharp
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new HTXRestClient().SpotApi.SharedClient;
var symbol = new SharedSymbol(TradingMode.Spot, "ETH", "USDT");

var result = await tickers.GetSpotTickerAsync(new GetTickerRequest(symbol));
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.LastPrice);
```

Use native HTX APIs for HTX-specific details such as spot account ids, margin loans, V5 futures, and detailed futures trigger/TP-SL behavior.

## Dependency Injection

```csharp
services.AddHTX(options =>
{
    options.ApiCredentials = new HTXCredentials("API_KEY", "API_SECRET");
});
```

Inject `IHTXRestClient` and `IHTXSocketClient`, or specific registered interfaces already used in the application.

## Error Handling

```csharp
var result = await client.SpotApi.ExchangeData.GetTickerAsync("ETHUSDT");
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
