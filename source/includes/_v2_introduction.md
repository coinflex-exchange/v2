# Change Log

**2021-02-26**

* Updated REST API [GET /v2/funding-rates/{marketCode}](#rest-api-methods-public-get-v2-funding-rates-marketcode)
    * does not require signature anymore, it's public
    * change type of `limit` `startTime` `endTime` from STRING to LONG
    * change `startTime` `endTime` from datetime (e.g.: `2020-12-08 20:00:00`) to millisecond timestamp(e.g.: `1579450778000`)

**2021-02-20**

* Added new websocket API [Liquidation RFQ](#websocket-api-subscriptions-public-liquidation-rfq), a subsription channel publishing upcoming liquidations
* Added new websocket API [Market](#websocket-api-subscriptions-public-market), a subsription channel publishing market information for each order book

**2021-02-05**

* Added new REST API [GET /v2.1/orders](#rest-api-methods-private-get-v2-1-orders) to get all orders of current user
* Added new REST API [GET /v2/candles](#rest-api-methods-public-get-v2-candles) to get candlestick data for the current candle
* Added new REST API [GET /v2/funding-rates/{marketCode}](#rest-api-methods-public-get-v2-funding-rates-marketcode) to get funding rates by marketCode
* Updated REST API [GET /v2/orders](#rest-api-methods-private-get-v2-orders)
    * type of timestamp changed from INTEGER to STRING
    * changed field name from `remainQuantity` to `remainingQuantity`
    * type of `orderCreated` and `lastModified` and `lastTradeTimestamp` changed from STRING TO INTEGER

**2021-01-26**

* Correction to REST API for [GET /v2/accountinfo](#rest-api-methods-private-get-v2-accountinfo), [GET /v2/balances](#rest-api-methods-private-get-v2-balances) and [GET /v2/positions](#rest-api-methods-private-get-v2-positions)

**2021-01-18**

* Updated rate limits to reflect new Rest and unauthenticated websocket rate limits

**2020-12-14**

* Added new websocket API [Place Batch Orders](#websocket-api-order-commands-place-batch-orders)
* Added new websocket API [Cancel Batch Orders](#websocket-api-order-commands-cancel-batch-orders)
* Added new websocket API [Modify Batch Orders](#websocket-api-order-commands-modify-batch-orders)

**2020-12-10**

* Added market order and stop-limit order websocket API details

**2020-09-25**

* Added physical delivery API endpoints
* Added position WebSocket channel
* Added balance WebSocket channel
* Added GET /v2/accountinfo
* Added GET /v2/ticker
* Added rate limits
* Added guidance for maintaining connections

**2020-07-25**

* Added GET /v2/publictrades/{marketCode}
* Added GET /v2/trades/{marketCode}

**2020-06-16**

* Added general guidance for getting a login/password to create tokens
* Updated some addresses for REST API

**2020-06-15**

* First beta version of API endpoints. Websocket and REST

# Introduction

Welcome to CoinFLEX's v2 application programming interface (API). CoinFLEX's APIs provide clients programmatic access to control aspects of their accounts and to place orders on CoinFLEX's trading platform. CoinFLEX supports the following types of APIs:

* a WebSocket API
* a REST API

Using these interfaces it is possible to place both authenticated and unauthenticated API commands for public and prvate commands respectively.

To get started please register for a TEST account at `https://v2stg.coinflex.com/user-console/register`


# API Key Management

An API key is required to make an authenticated API command.  API keys (public and corresponding secret key) can be generated via the CoinFLEX GUI within a clients account. 

By default, API Keys are read-only and can only read basic account information, such as positions, orders, and trades. They cannot be used to trade such as placing, modifying or cancelling orders.

If you wish to execute orders with your API Key, clients must select the `Can Trade` permission upon API key creation.

API keys are also only bound to a single sub-account, defined upon creation. This means that an API key will only ever interact and return account information for a single sub-account.

# Rate Limit

CoinFLEX's APIs allows our clients to access and control their accounts or view our market data using custom-written software. To protect the performance of the system, we impose certain limits:

Type                           |                             Limit|
-------------------------------|----------------------------------|
Rest API                       |                   100 per second |
Rest API                       |                  2500 per 5 mins |
Rest POST v2.1/delivery/orders |                 2 per 10 seconds |
Websocket API (Auth)           |                   200 per second | 
Websocket API (No Auth)        |                     1 per second | 
