# Weex.Net API Surfaces

Use this file to select Weex client roots, endpoint groups, streams, SharedApis, and result types.

## Package And Clients

- Package: `Weex.Net`
- REST: `WeexRestClient`, `IWeexRestClient`
- Socket: `WeexSocketClient`, `IWeexSocketClient`
- Credentials: `WeexCredentials(key, secret, passphrase)`
- DI: `services.AddWeex(...)`
- Factories: `IWeexOrderBookFactory`, `IWeexTrackerFactory`
- Multi-user clients: `IWeexUserClientProvider`

## Roots

- `client.SpotApi`
- `client.FuturesApi`
- `socket.SpotApi`
- `socket.FuturesApi`

Each API exposes `SharedClient`.

## Spot REST

`SpotApi.ExchangeData`:

- `GetServerTimeAsync()`
- `GetAssetsAsync()`
- `GetExchangeInfoAsync(symbols, symbolStatus)`
- `GetPricesAsync(symbols)`
- `GetTickersAsync(symbols)`
- `GetRecentTradesAsync(symbol, limit)`
- `GetKlinesAsync(symbol, interval)`
- `GetOrderBookAsync(symbol, limit)`
- `GetBookTickersAsync(symbols)`

`SpotApi.Account`:

- `GetTradingSymbolsAsync()`
- `GetAccountInfoAsync()`
- `GetAccountBillsAsync(...)`
- `GetFundingBillsAsync(...)`
- `GetTransferHistoryAsync(...)`

`SpotApi.Trading`:

- `PlaceOrderAsync(symbol, side, orderType, quantity, price, timeInForce, clientOrderId)`
- `CancelOrderAsync(orderId, clientOrderId)`
- `CancelAllSymbolOrdersAsync(symbol)`
- `CancelOrdersAsync(orderIds, clientOrderIds)`
- `GetOrderAsync(orderId, clientOrderId)`
- `GetOpenOrdersAsync(symbol)`
- `GetOrderHistoryAsync(symbol, startTime, endTime, page, limit)`
- `GetUserTradesAsync(symbol, orderId, startTime, endTime, limit)`

## Futures REST

`FuturesApi.ExchangeData`:

- exchange info, order book, tickers, book tickers, trades
- regular/index/mark klines and kline history
- price, open interest, funding rate/history, trading symbols

`FuturesApi.Account`:

- balances, fees, account/symbol configuration, bills
- `SetMarginModeAsync(...)`
- `SetLeverageAsync(...)`
- `AdjustIsolatedMarginAsync(...)`
- `SetAutoAppendMarginAsync(...)`

`FuturesApi.Trading`:

- positions and position lookup
- regular order placement/cancellation/history/trades
- `ClosePositionsAsync(symbol)`
- conditional order placement/cancellation/history
- TP/SL placement and editing

Regular `PlaceOrderAsync` uses `OrderType`. `PlaceConditionalOrderAsync` uses `FuturesOrderType`.

## Socket Streams

Spot public:

- ticker, book ticker, order book, kline, trade

Spot private:

- account, order, user trade

Futures public:

- ticker, kline, order book, trade

Futures private:

- account, position, order, user trade

All subscription methods return `WebSocketResult<UpdateSubscription>`.

## SharedApis

Spot REST implements assets, balance, book ticker, deposit, fee, kline, order book, recent trade, withdrawal, spot symbol/ticker/order, and client-order-id interfaces.

Deposit and withdrawal SharedApis provide history views based on account bills. Deposit address retrieval is unavailable, and the current native surface does not initiate deposits, withdrawals, or transfers.

Futures REST implements balance, book ticker, fee, kline, order book, recent trade, funding rate, futures symbol/ticker/order/trigger order, index/mark kline, leverage, and open-interest interfaces.

Spot socket implements balance, book ticker, kline, ticker, trade, user trade, and spot-order interfaces.

Futures socket implements balance, kline, ticker, trade, user trade, futures-order, and position interfaces.

Use `SharedClient.Discover()` for capability metadata.

## Environment And Symbols

- Built-in environment: `WeexEnvironment.Live`
- Custom environment: `WeexEnvironment.CreateCustom(...)`
- No built-in testnet is exposed by the current source.
- Spot and futures commonly use compact symbols such as `ETHUSDT`.

## Result Types

- REST: `HttpResult<T>` or `HttpResult`
- Socket: `WebSocketResult<UpdateSubscription>`
- Shared helpers: `ExchangeCallResult<T>`

Check `Success` before using `Data`.
