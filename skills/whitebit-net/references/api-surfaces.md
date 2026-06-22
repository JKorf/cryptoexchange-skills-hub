# WhiteBit.Net API Surfaces

## Package And Clients

- Package: `WhiteBit.Net`
- REST: `WhiteBitRestClient`, `IWhiteBitRestClient`
- Socket: `WhiteBitSocketClient`, `IWhiteBitSocketClient`
- Credentials: `WhiteBitCredentials(key, secret)`
- DI: `services.AddWhiteBit(...)`
- Factories: `IWhiteBitOrderBookFactory`, `IWhiteBitTrackerFactory`
- Multi-user clients: `IWhiteBitUserClientProvider`

## V4 Root

REST groups:

- `V4Api.ExchangeData`
- `V4Api.Account`
- `V4Api.Trading`
- `V4Api.CollateralTrading`
- `V4Api.SubAccount`
- `V4Api.Convert`
- `V4Api.Codes`
- `V4Api.SharedClient`

Socket surface: `socket.V4Api` and `socket.V4Api.SharedClient`.

## Exchange Data

- server time, symbols, status, tickers, assets
- order book and recent trades
- deposit/withdrawal information
- collateral and futures symbols
- funding history

## Account

- main, spot, and collateral balances
- deposit addresses, fiat deposit URL, withdrawals, transfers
- deposit/withdrawal history and settings
- collateral summaries and account funding history
- account leverage, trading fees, hedge mode

## Spot Trading

- `PlaceSpotOrderAsync(...)`
- batch spot orders
- cancel one/multiple/all
- open/closed orders, user trades, order trades
- edit order
- kill switch and status

## Collateral Trading

- `PlaceOrderAsync(...)`
- open positions and position history
- open conditional orders
- OCO placement/cancellation
- conditional and OTO cancellation

Collateral trading covers perpetual futures and spot margin products.

## Other REST Groups

`SubAccount`: create, edit, delete, list, block/unblock, transfer, balances, history.

`Convert`: estimate, confirm, history.

`Codes`: create/apply WhiteBit codes and inspect histories.

## Socket Requests And Streams

Socket request/response methods return `QueryResult<T>` and include trade history, last price, ticker, klines, order book, balances, open/closed orders, and user trades.

Subscriptions return `WebSocketResult<UpdateSubscription>` and include:

- public trades, book ticker, last price, ticker, klines, order book
- private spot/margin balances, open/closed orders, user trades
- positions, borrow updates, and margin-position events

## SharedApis

REST supports spot symbols/tickers/orders/trigger orders, futures symbols/tickers/orders/trigger orders/TP-SL, balances, assets, deposits, withdrawals, transfers, fees, order books, trades, book tickers, funding rates, leverage, open interest, and position history.

Socket supports balance, book ticker, kline, ticker, trade, user trade, spot order, futures order, and position interfaces.

Use `SharedClient.Discover()` for runtime capability metadata.

## Symbols And Environment

- Spot: `ETH_USDT`
- Perpetual collateral: `ETH_PERP`
- Built-in environment: `WhiteBitEnvironment.Live`
- Custom environment: `WhiteBitEnvironment.CreateCustom(...)`
- No built-in testnet in the current source.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Socket requests: `QueryResult<T>`
- Socket subscriptions: `WebSocketResult<UpdateSubscription>`
- Batch methods can contain nested `CallResult<T>` values
- Shared helpers: `ExchangeCallResult<T>`

Always check `Success` before using `Data`.

## Local Order Books

`IWhiteBitOrderBookFactory` exposes `V4`, `Create(SharedSymbol, ...)`, and `CreateV4(symbol, ...)`.
