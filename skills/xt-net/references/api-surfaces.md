# XT.Net API Surfaces

## Package And Clients

- Package: `XT.Net`
- REST: `XTRestClient`, `IXTRestClient`
- Socket: `XTSocketClient`, `IXTSocketClient`
- Credentials: `XTCredentials(key, secret)`
- DI: `services.AddXT(...)`
- Factories: `IXTOrderBookFactory`, `IXTTrackerFactory`
- Multi-user clients: `IXTUserClientProvider`

## Product Roots

REST:

- `SpotApi.ExchangeData`, `SpotApi.Account`, `SpotApi.Trading`, `SpotApi.SharedClient`
- `UsdtFuturesApi.ExchangeData`, `UsdtFuturesApi.Account`, `UsdtFuturesApi.Trading`, `UsdtFuturesApi.SharedClient`
- `CoinFuturesApi.ExchangeData`, `CoinFuturesApi.Account`, `CoinFuturesApi.Trading`, `CoinFuturesApi.SharedClient`

Socket:

- `SpotApi` and `SpotApi.SharedClient`
- `FuturesApi` and `FuturesApi.SharedClient`

The REST futures products are separate; the websocket futures surface is combined.

## Spot REST

Exchange data covers server time, symbols, currencies, tickers, order books, trades, and klines.

Account covers balances, deposit addresses/history, withdrawals/history, transfers, and websocket tokens.

Trading includes:

- `PlaceOrderAsync(...)`
- `GetOrderAsync(long orderId)` and client-order-id lookup
- cancel, edit, batch get/cancel, and cancel-all
- open/closed orders and user trades

Spot placement requires `TimeInForce` and `BusinessType`. Market buys use `quoteQuantity`; market sells use `quantity`.

## Futures REST

Both USDT-M and Coin-M roots expose market data, account, and trading groups. Capabilities include symbols, tickers, order books, trades, klines, funding rates, open interest, balances, leverage, positions, regular orders, trigger orders, and TP/SL workflows.

Futures order placement requires `symbol`, `OrderSide`, `OrderType`, `quantity`, and `PositionSide`. Limit orders add price and normally `TimeInForce`.

`CloseAllPositionsAsync()` affects every open futures position in the selected REST product/account scope.

## Socket Streams

Spot public streams include trades, klines, snapshots and incremental order books, and tickers. Spot private streams include balance, order, and user-trade updates.

Futures public streams include trades, klines, tickers, aggregate tickers, index/mark prices, snapshot and incremental order books, and funding rates. Futures private streams include balances, positions, orders, user trades, and notifications.

Private stream overloads accept an explicit token/listen key. Overloads without one require credentials and acquire/maintain the key internally.

## SharedApis

Spot REST implements assets, balances, deposits, withdrawals, withdraw, transfers, klines, order books, recent trades, tickers, symbols, orders, fees, and book tickers.

Futures REST implements balances, klines, order books, recent trades, funding rates, symbols, tickers, leverage, open interest, orders, fees, trigger orders, TP/SL, and book tickers.

Spot socket implements balances, klines, order books, tickers, trades, user trades, and spot orders. Futures socket implements balances, klines, order books, tickers, trades, user trades, futures orders, and positions.

Call `SharedClient.Discover()` to inspect supported interfaces at runtime.

## Factories And Providers

`IXTOrderBookFactory` exposes `Spot`, `UsdtFutures`, and `CoinFutures`, plus `CreateSpot`, `CreateUsdtFutures`, `CreateCoinFutures`, and `Create(SharedSymbol, ...)`.

`IXTTrackerFactory` exposes spot and USDT-futures user-data tracker creation. Coin-futures tracker methods are not currently active in the interface.

`IXTUserClientProvider` initializes and caches credentialed REST/socket clients by user identifier.

## Symbols And Environment

- Spot native symbol: `eth_usdt`
- Futures native symbol: `ETH_USDT`
- Built-in environment: `XTEnvironment.Live`
- Custom environment: `XTEnvironment.CreateCustom(...)`
- No built-in testnet in the current source

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Socket subscriptions: `WebSocketResult<UpdateSubscription>`
- Shared helpers: `ExchangeCallResult<T>`

Always check `Success` before using `Data`.
