# Binance.Net Usage

## Public Spot Ticker

```csharp
using Binance.Net.Clients;

var client = new BinanceRestClient();
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTCUSDT");

if (!ticker.Success)
{
    Console.WriteLine($"Failed to get ticker: {ticker.Error}");
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

## Authenticated Spot Account

```csharp
using Binance.Net;
using Binance.Net.Clients;
using Binance.Net.Objects;

var client = new BinanceRestClient(options =>
{
    options.ApiCredentials = new BinanceCredentials("API_KEY", "API_SECRET");
});

var account = await client.SpotApi.Account.GetAccountInfoAsync();
if (!account.Success)
{
    Console.WriteLine(account.Error);
    return;
}
```

## Spot Limit Order

```csharp
using Binance.Net.Enums;

var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTCUSDT",
    side: OrderSide.Buy,
    type: SpotOrderType.Limit,
    quantity: 0.001m,
    price: 50000m,
    timeInForce: TimeInForce.GoodTillCanceled);
```

## SharedApis

```csharp
using Binance.Net.Clients;
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new BinanceRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")));
```

## Local Examples

When available, read:

- `C:\Projects\Binance.Net\Examples\ai-friendly\01-spot-quickstart.cs`
- `C:\Projects\Binance.Net\Examples\ai-friendly\02-futures.cs`
- `C:\Projects\Binance.Net\Examples\ai-friendly\03-websocket.cs`
- `C:\Projects\Binance.Net\Examples\ai-friendly\04-multi-exchange.cs`
- `C:\Projects\Binance.Net\Examples\ai-friendly\05-error-handling.cs`
