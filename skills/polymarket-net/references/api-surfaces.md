# Polymarket.Net API Surfaces

Use this file when choosing native Polymarket client roots, endpoint groups, result types, credentials, socket streams, or token-id patterns.

## Package And Client Types

- NuGet package: `Polymarket.Net`
- REST client: `PolymarketRestClient`
- Socket client: `PolymarketSocketClient`
- REST interface: `IPolymarketRestClient`
- Socket interface: `IPolymarketSocketClient`
- Credentials: `PolymarketCredentials`
- L1 credential: `PolymarketL1Credential`
- L2 credential: `HMACPassCredential`
- DI extension: `services.AddPolymarket(...)`
- Local order book factory: `IPolymarketOrderBookFactory`
- User client provider: `IPolymarketUserClientProvider`

## Client Roots

REST roots:

- `client.ClobApi`
- `client.GammaApi`
- `client.DataApi`

Socket root:

- `socket.ClobApi`

Do not use exchange roots such as `SpotApi`, `FuturesApi`, `UsdFuturesApi`, `CoinFuturesApi`, `UnifiedApi`, or `ExchangeApi`.

## CLOB REST Endpoint Groups

`client.ClobApi.ExchangeData`:

- `GetServerTimeAsync()`
- `GetGeographicRestrictionsAsync()`
- `GetSamplingSimplifiedMarketsAsync(cursor)`
- `GetSamplingMarketsAsync(cursor)`
- `GetSimplifiedMarketsAsync(cursor)`
- `GetMarketsAsync(cursor)`
- `GetMarketAsync(id)`
- `GetPriceAsync(tokenId, side)`
- `GetPricesAsync(tokens)`
- `GetMidpointPriceAsync(tokenId)`
- `GetMidpointPricesAsync(tokenIds)`
- `GetPriceHistoryAsync(market, startTime, endTime, interval, fidelity)`
- `GetBidAskSpreadsAsync(tokenId)`
- `GetBidAskSpreadsAsync(tokenIds)`
- `GetOrderBookAsync(tokenId)`
- `GetOrderBooksAsync(tokenIds)`
- `GetTickSizeAsync(tokenId)`
- `GetNegativeRiskAsyncAsync(tokenId)`
- `GetFeeRateBpsAsync(tokenId)`
- `GetLastTradePriceAsync(tokenId)`
- `GetLastTradePricesAsync(tokenIds)`
- `GetMarketInfoAsync(marketId)`

`client.ClobApi.Account`:

- `CreateApiCredentialsAsync(nonce)`
- `GetApiCredentialsAsync(nonce)`
- `GetOrCreateApiCredentialsAsync(nonce)`
- `GetApiKeysAsync()`
- `DeleteApiKeyAsync()`
- `GetClosedOnlyModeAsync()`
- `GetNotificationsAsync()`
- `DropNotificationsAsync(ids)`
- `GetBalanceAllowanceAsync(assetType, tokenId)`
- `UpdateBalanceAllowanceAsync(assetType, tokenId)`
- `GetBuilderTradesAsync(builderCode, tradeId, marketId, tokenId, startTime, endTime, cursor)`

`client.ClobApi.Trading`:

- `GetOpenOrdersAsync(orderId, marketId, tokenId, cursor)`
- `GetOrderAsync(orderId)`
- `GetOrderRewardScoringAsync(orderId)`
- `GetOrdersRewardScoringAsync(orderIds)`
- `PlaceOrderAsync(tokenId, side, orderType, quantity, price, timeInForce, postOnly, clientOrderId, expiration, quantityType, tickQuantity, negativeRisk)`
- `PlaceMultipleOrdersAsync(requests)`
- `CancelOrderAsync(orderId)`
- `CancelOrdersAsync(orderIds)`
- `CancelOrdersOnMarketAsync(marketId, tokenId)`
- `CancelAllOrdersAsync()`
- `GetUserTradesAsync(tradeId, makerAddress, marketId, tokenId, startTime, endTime, cursor)`
- `PostOrderHeartbeatAsync()`

## Gamma REST Endpoint Group

`client.GammaApi` exposes human-readable discovery and metadata:

- `GetSportTeamsAsync(...)`
- `GetSportsAsync()`
- `GetSportMarketTypesAsync()`
- `GetTagsAsync(...)`
- `GetTagByIdAsync(id, includeTemplate)`
- `GetTagBySlugAsync(slug, includeTemplate)`
- `GetRelatedTagsByIdAsync(id, omitEmpty, status)`
- `GetRelatedTagsBySlugAsync(slug, omitEmpty, status)`
- `GetTagsRelatedToTagByIdAsync(id, omitEmpty, status)`
- `GetTagsRelatedToTagBySlugAsync(slug, omitEmpty, status)`
- `GetEventsKeysetPaginationAsync(...)`
- `GetEventsAsync(...)`
- `GetEventByIdAsync(id, includeChat, includeTemplate)`
- `GetEventBySlugAsync(slug, includeChat, includeTemplate)`
- `GetEventTagsAsync(id)`
- `GetMarketsAsync(...)`
- `GetMarketByIdAsync(id, includeTag)`
- `GetMarketBySlugAsync(slug, includeTag)`
- `GetMarketTagsAsync(id)`
- `GetSeriesAsync(...)`
- `GetSeriesByIdAsync(id, includeChat)`
- `SearchAsync(query, ...)`

Prefer `GammaApi` for user-facing market/event discovery and `ClobApi` for token-level trading and order books.

## Data REST Endpoint Group

`client.DataApi`:

- `GetPositionsAsync(user)`

Use `DataApi` for public position lookups by user address.

## Socket Streams

`socket.ClobApi` public streams:

- `SubscribeToPlatformUpdatesAsync(onNewMarketUpdate, onMarketResolvedUpdate)`
- `SubscribeToTokenUpdatesAsync(tokenIds, onPriceChangeUpdate, onBookUpdate, onLastTradePriceUpdate, onTickSizeUpdate, onBestBidAskUpdate)`
- `SubscribeToSportsUpdatesAsync(onSportsUpdate)`

`socket.ClobApi` private stream:

- `SubscribeToUserUpdatesAsync(onOrderUpdate, onTradeUpdate)`

Private user streams require configured L1 and L2 credentials.

## Credentials

`PolymarketCredentials` contains:

- `L1`: `PolymarketL1Credential`
- `L2`: optional `HMACPassCredential`

Construct directly:

```csharp
new PolymarketCredentials(
    new PolymarketL1Credential(SignType.Poly1271, "PRIVATE_KEY", "POLYMARKET_ADDRESS"),
    new HMACPassCredential("L2_KEY", "L2_SECRET", "L2_PASS"));
```

Or fluently:

```csharp
new PolymarketCredentials()
    .WithL1(SignType.Poly1271, "PRIVATE_KEY", "POLYMARKET_ADDRESS")
    .WithL2("L2_KEY", "L2_SECRET", "L2_PASS");
```

Use `GetOrCreateApiCredentialsAsync()` to derive/create L2 credentials from L1, then `UpdateL2Credentials(...)` on the REST or socket client.

## Token And Market Identifiers

- CLOB orders, prices, spreads, tick sizes, fee rates, and order books use `tokenId`.
- Gamma markets/events provide human-readable metadata and may expose CLOB token ids.
- `marketId` usually means condition/market id, not token id.
- Do not generate crypto exchange symbols like `BTCUSDT`, `ETH-USDT`, or futures contract ids for Polymarket CLOB methods.
- Prices are between `0` and `1`.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Websocket subscriptions: `WebSocketResult<UpdateSubscription>`
- Batch order methods: `HttpResult<CallResult<PolymarketOrderResult>[]>`
- Internal helper utilities can return `CallResult<T>`

Always check `Success` before using `Data`. For `PlaceOrderAsync`, also check `order.Data.Success` and `order.Data.Error`.

## Local Order Books And User Clients

DI registers:

- `IPolymarketOrderBookFactory`
- `IPolymarketUserClientProvider`

Create local order books with:

```csharp
var orderBook = orderBookFactory.CreateClob("TOKEN_ID");
```

The user client provider manages separate credentials per user:

```csharp
userClientProvider.InitializeUserClient("alice", credentials);
var aliceRest = userClientProvider.GetRestClient("alice");
```
