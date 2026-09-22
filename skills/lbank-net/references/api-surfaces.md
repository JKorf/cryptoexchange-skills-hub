# LBank.Net API Surfaces

Use this file when choosing LBank client roots, native methods, socket streams, shared interfaces, result types, factories, or environments.

## Package and Client Types

- NuGet package: `LBank.Net`
- REST client: `LBankRestClient`
- Socket client: `LBankSocketClient`
- Credentials: `LBankCredentials`
- REST interface: `ILBankRestClient`
- Socket interface: `ILBankSocketClient`
- DI extension: `services.AddLBank(...)`
- Local order book factory: `ILBankOrderBookFactory`
- Tracker factory: `ILBankTrackerFactory`
- User client provider: `ILBankUserClientProvider`

## Client Roots

- REST: `client.SpotApi`
- Socket: `socket.SpotApi`

Do not use `UnifiedApi`, `FuturesApi`, `DerivativesApi`, or `MarginApi`.

## REST Exchange Data

`client.SpotApi.ExchangeData`:

- `GetServerTimeAsync()`
- `GetAvailableSymbolsAsync()`
- `GetSymbolsAsync(symbol)`
- `GetAssetsAsync(asset)`
- `GetOrderBookAsync(symbol, limit)`
- `GetPriceAsync(symbol)`
- `GetBookTickerAsync(symbol)`
- `GetTickersAsync(symbol)`
- `GetLeveragedTokenTickersAsync(symbol)`
- `GetTradesAsync(symbol, limit, afterTime)`
- `GetKlinesAsync(symbol, interval, limit, afterTime)`

`GetPriceAsync`, `GetTickersAsync`, and `GetLeveragedTokenTickersAsync` return arrays. REST order-book depth is at most 200; recent-trade limit is at most 500; kline limit is at most 2000.

## REST Account and Wallet

`client.SpotApi.Account`:

- `GetApiKeyInfoAsync()`
- `GetUserAssetsAsync()`
- `GetAccountInfoAsync()`
- `WithdrawAsync(address, asset, quantity, fee, network, memo, notes, name, clientOrderId, internalTransfer)`
- `GetDepositHistoryAsync(asset, status, startTime, endTime)`
- `GetWithdrawHistoryAsync(asset, status, clientOrderId, startTime, endTime)`
- `GetDepositAddressAsync(asset, network)`
- `GetAssetDetailsAsync(asset)`
- `GetTradeFeeAsync(symbol)`
- `StartUserStreamAsync()`
- `KeepAliveUserStreamAsync(listenKey)`
- `StopUserStreamAsync(listenKey)`

All account methods require credentials. `WithdrawAsync` mutates the live account.

## REST Trading

`client.SpotApi.Trading`:

- `PlaceOrderAsync(symbol, orderType, quantity, price, clientOrderId, receiveWindow)`
- `GetOrderAsync(symbol, orderId, clientOrderId)`
- `CancelOrderAsync(symbol, orderId, clientOrderId)`
- `CancelAllOrdersAsync(symbol)`
- `GetOrdersAsync(symbol, page, pageSize, status)`
- `GetOpenOrdersAsync(symbol, page, pageSize)`
- `GetUserTradesAsync(symbol, fromId, startTime, endTime, limit)`

Use exactly one of `orderId` and `clientOrderId` for order lookup or cancellation.

`OrderType` combines side and behavior:

- `BuyLimit`, `SellLimit`
- `BuyMarket`, `SellMarket`
- `BuyMaker`, `SellMaker`
- `BuyIoc`, `SellIoc`
- `BuyFok`, `SellFok`

For a market buy, `quantity` is in the quote asset. The user-trade limit is at most 100; order page size is at most 200.

## Socket Streams

`socket.SpotApi`:

- `SubscribeToTradeUpdatesAsync(symbol, handler)`
- `SubscribeToKlineUpdatesAsync(symbol, StreamKlineInterval, handler)`
- `SubscribeToOrderBookUpdatesAsync(symbol, depth, handler)`
- `SubscribeToTickerUpdatesAsync(symbol, handler)`
- `SubscribeToOrderUpdatesAsync(listenKey, handler)`
- `SubscribeToBalanceUpdatesAsync(listenKey, handler)`

Socket order-book depth must be `10`, `50`, or `100`. Pass `null` as the private-stream listen key only when the socket client has credentials and should manage the key.

## Native Symbols and Models

- Native symbol format: lowercase `base_quote`
- Example: `eth_usdt`
- Native ticker wrapper: `LBankSymbolTicker.Ticker.LastPrice`
- Socket ticker: `LBankTickerUpdate.LastPrice`
- Order book sides: `Bids` and `Asks`
- Order book entry properties: `Price` and `Quantity`
- REST candle enum: `KlineInterval`
- Socket candle enum: `StreamKlineInterval`

Some individual upstream endpoints document compact uppercase symbol inputs. Follow the selected method's interface documentation when it explicitly differs; default native integration examples to lowercase underscore-separated symbols.

## Shared API V2

Strict capabilities are exposed through `SharedApi` on the supported native client roots:

- `restClient.SpotApi.SharedApi`
- `socketClient.SpotApi.SharedApi`

Each V2 interface represents one operation, such as `IGetTickerRest`, `IPlaceSpotOrderRest`, or `ISubscribeTickerSocket`. Depend on the narrowest capability required by the workflow instead of a legacy topic client.

The exchange aggregate is `ILBankSharedApiClient`. It exposes:

- `SpotRest` as `ILBankRestClientSpotSharedApi`
- `SpotSocket` as `ILBankSocketClientSpotSharedApi`

Use an aggregate property for compile-time discovery. Use runtime lookup only when the capability, trading mode, or transport is selected dynamically:

```csharp
var match = SharedApi.GetCapability(
    SharedCapabilities.Tickers.GetTicker.Rest);

if (match is not null)
    Console.WriteLine($"{match.Exchange} / {match.Transport}");
```

`SharedCapabilities.Tickers.GetTicker.Rest` is a typed descriptor, not an implementation or guarantee of support. A match contains the capability implementation and its options. Use `GetCapabilities` for all matching surfaces and `Discover` for summary metadata.

The library's DI registration registers `ILBankSharedApiClient` and its supported strict capability interfaces. Inject a narrow capability when only one operation is needed. If several exchanges are registered, inject `IEnumerable<TCapability>` and select by exchange and supported trading mode.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Websocket subscription: `WebSocketResult<UpdateSubscription>`

Always check `Success` before accessing `Data`. Non-generic `HttpResult` has no response data.

## Factories, Trackers, and User Clients

`ILBankOrderBookFactory`:

- `Spot`
- `Create(SharedSymbol, options)`
- `CreateSpot(symbol, options)`

`ILBankTrackerFactory` supports the standard Spot kline and trade tracker factory members plus:

- `CreateUserSpotDataTracker(userIdentifier, credentials, config, environment, exchangeParameters)`
- `CreateUserSpotDataTracker(config, exchangeParameters)`

Use `ILBankUserClientProvider` for cached per-user REST/socket clients. Clear cached clients when credentials change.

## Environments and Socket Version

- Built in: `LBankEnvironment.Live`
- Custom: `LBankEnvironment.CreateCustom(name, spotRestAddress, spotSocketStreamsAddress, spotSocketStreamsV3Address)`
- Public socket default: `LBankSocketOptions.UseV3 = true`

No built-in testnet environment exists.
