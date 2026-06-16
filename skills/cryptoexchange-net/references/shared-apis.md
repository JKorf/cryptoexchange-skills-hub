# SharedApis Reference

Use `CryptoExchange.Net.SharedApis` for exchange-agnostic code.

## Typical Interfaces

- `ISpotTickerRestClient`: spot ticker data
- `ISpotOrderRestClient`: spot order placement, cancellation, and lookup
- `IFuturesOrderRestClient`: futures order placement, cancellation, and lookup
- `IBalanceRestClient`: account balances
- `IPositionRestClient`: futures positions
- `ITickerSocketClient`: ticker websocket updates
- `IOrderBookSocketClient`: order book websocket updates

## Pattern

```csharp
using Binance.Net.Clients;
using OKX.Net.Clients;
using CryptoExchange.Net.SharedApis;

var symbol = new SharedSymbol(TradingMode.Spot, "BTC", "USDT");

ISpotTickerRestClient binance = new BinanceRestClient().SpotApi.SharedClient;
ISpotTickerRestClient okx = new OKXRestClient().UnifiedApi.SharedClient;

var requests = new[]
{
    ("Binance", binance.GetSpotTickerAsync(new GetTickerRequest(symbol))),
    ("OKX", okx.GetSpotTickerAsync(new GetTickerRequest(symbol)))
};

var results = await Task.WhenAll(requests.Select(x => x.Item2));

foreach (var item in requests.Zip(results))
{
    var name = item.First.Item1;
    var result = item.Second;
    if (!result.Success)
    {
        Console.WriteLine($"{name}: {result.Error}");
        continue;
    }

    Console.WriteLine($"{name} {result.Data.Symbol}: {result.Data.LastPrice}");
}
```

## Guidance

- Prefer `SharedSymbol` for shared API calls.
- Do not mix shared request types with exchange-specific endpoint methods.
- Prefer `Task.WhenAll` for independent calls across exchanges.
- Preserve cancellation tokens when adding production examples.
- Use exchange-specific clients when a shared interface does not expose the requested behavior.

## Known Client Surface Examples

- Binance: `new BinanceRestClient().SpotApi.SharedClient`
- OKX: `new OKXRestClient().UnifiedApi.SharedClient`
- Bybit: `new BybitRestClient().V5Api.SharedClient`
- Kraken: `new KrakenRestClient().SpotApi.SharedClient`
- Coinbase: `new CoinbaseRestClient().AdvancedTradeApi.SharedClient`
