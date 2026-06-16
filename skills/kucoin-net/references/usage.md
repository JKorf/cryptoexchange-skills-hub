# Kucoin.Net Usage

## Public Spot Ticker

```csharp
using Kucoin.Net.Clients;

var client = new KucoinRestClient();
var ticker = await client.SpotApi.ExchangeData.GetTickerAsync("BTC-USDT");

if (!ticker.Success)
{
    Console.WriteLine($"Failed to get ticker: {ticker.Error}");
    return;
}

Console.WriteLine(ticker.Data.LastPrice);
```

## Authenticated Spot Account

```csharp
using Kucoin.Net;
using Kucoin.Net.Clients;

var client = new KucoinRestClient(options =>
{
    options.ApiCredentials = new KucoinCredentials("API_KEY", "API_SECRET", "API_PASSPHRASE");
});

var accounts = await client.SpotApi.Account.GetAccountsAsync();
if (!accounts.Success)
{
    Console.WriteLine(accounts.Error);
    return;
}
```

## Spot Limit Order

```csharp
using Kucoin.Net.Enums;

var order = await client.SpotApi.Trading.PlaceOrderAsync(
    symbol: "BTC-USDT",
    side: OrderSide.Buy,
    type: NewOrderType.Limit,
    quantity: 0.001m,
    price: 50000m,
    timeInForce: TimeInForce.GoodTillCanceled);
```

## SharedApis

```csharp
using Kucoin.Net.Clients;
using CryptoExchange.Net.SharedApis;

ISpotTickerRestClient tickers = new KucoinRestClient().SpotApi.SharedClient;
var result = await tickers.GetSpotTickerAsync(
    new GetTickerRequest(new SharedSymbol(TradingMode.Spot, "BTC", "USDT")));
```

## Local Examples

When available, read:

- `C:\Projects\Kucoin.Net\Examples\ai-friendly\01-spot-quickstart.cs`
- `C:\Projects\Kucoin.Net\Examples\ai-friendly\02-futures.cs`
- `C:\Projects\Kucoin.Net\Examples\ai-friendly\03-websocket.cs`
- `C:\Projects\Kucoin.Net\Examples\ai-friendly\04-multi-exchange.cs`
- `C:\Projects\Kucoin.Net\Examples\ai-friendly\05-error-handling.cs`
