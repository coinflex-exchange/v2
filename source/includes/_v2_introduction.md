# Introduction

Welcome to CoinFLEX's v2 application programming interface (API). Please note, this is a beta version and should be considered a test net.

CoinFLEX's APIs provide our clients programmatic access to control aspects of their accounts and to place orders on CoinFLEX's trading platform. CoinFLEX provides the following APIs:

* a WebSocket API
* a REST API

Using these interfaces it is possible to place both authenticated and unauthenticated API commands for public and prvate commands respectively.

To get started please register for a TEST account at `https://v2stg.coinflex.com/user-console/register`


# API Key Management

An API key is required to make an authenticated API command.  API keys (public and corresponding secret key) can be generated via the CoinFLEX GUI within a clients account. 

By default, API Keys are read-only and can only read basic account information, such as positions, orders, and trades. They cannot be used to trade such as placing, modifying or cancelling orders.

If you wish to execute orders with your API Key, clients must select the `Can Trade` permission upon API key creation.

API keys are also only bound to a single sub-account, defined upon creation. This means that an API key will only ever interact and return account information for a single sub-account.


# Change Log
2020-07-25
- Added GET /v2/publictrades/{marketCode}
- Added GET /v2/trades/{marketCode}

2020-06-16
- Added general guidance for getting a login/password to create tokens
- Updated some addresses for REST API

2020-06-15
- First version of beta API endpoints - Websocket and REST

# Rate Limit

CoinFLEX's application programming interface (API) allows our clients to access and control their accounts or view our market data using custom-written software. To protect the performance of the system, we impose certain limits:

Type                    |                            Limit|
------------------------|---------------------------------|
Rest API                |                  100 per second |
Websocket API           |                  200 per second |
