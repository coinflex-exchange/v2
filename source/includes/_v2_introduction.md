# Change Log

**2021-01-18**

* Updated rate limits to reflect new Rest and unauthenticated websocket rate limits

**2020-12-14**

* Added new websocket API Place Batch Orders command
* Added new websocket API Cancel Batch Orders command
* Added new websocket API Modify Batch Orders command

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
