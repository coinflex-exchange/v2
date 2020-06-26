# V2.0 REST API

**TEST** site
* `https://api-test-v2.coinflex-cn.com/ `

**LIVE** site
* 'COMING SOON'

For clients who do not wish to take advantage of CoinFLEX's native WebSocket API, CoinFLEX offers a RESTful API that implements much of the same functionality.

##Authentication 

The public market data methods do not require authentication, however private methods require a *Signature* to be sent in the header of the request.  These private REST methods  use HMAC SHA256 signatures. 

The HMAC SHA256 signature is a keyed HMAC SHA256 operation using a clients API Secret as the key and a predifined message string as the value for the HMAC operation.  The message string is made up of two parts:-
* a query string made up of the following formula:-

Formula Component | Type | Example | 
-------------------------- | -----|--------- |
Timestamp | YYYY-MM-DDThh:mm:ss | 2020-04-30T15:20:30 |
Nonce | INTEGER | 123 |
Verb | STRING | 'GET' |
URL | STRING | 'api-test-v2.coinflex-cn.com' |
Path | STRING | '/v2/positions |

The final constructed query string should look like:-

  `2020-04-30T15:20:30\n
  123\n
  GET\n
  api-test-v2.coinflex-cn.com\n
  /v2/positions\n `

Note the newline characters after each component in the query string above.

* the REST request body, for example:- 
  * {instrumentId": "BTC"}

The final message string which needs to be signed is the concatenation of both the query string and the request body.


> **Request format**

```json
{
  "email":"<email>",
  "password":"<password>"
}
```

> **RESPONSE**

```json
{
    "event": null,
    "success": true,
    "message": null,
    "code": "0000",
    "data": {
        "token": "...",
        "email": "xxxx@xxx.com",
        "accountName": "xxxx@xxx.com",
        "mainLogin": true
    }
}
```

##GET `/v2/account/protected/balances/`

GET coin balanaces in your account

Note: x-cf-token is required in HTTP header, which is the token returned by /v2/account/auth/trading/login

> **Request format**

```json
{
    “event”: null,
    “success”: true,
    “message”: null,
    “code”: “0000”,
    “data”:[
    {
        "instrumentId": "BTC",
        "available": 4468.823,
        "reserved": 0
    },
    {
        "instrumentId": "EOS",
        "available": 325.890,
        "reserved": 1260
    }
    ]
}
```


Parameters |Parameter Types | Description| 
-------------------------- | -----|--------- |
instrumentId | String | | Token symbol, e.g. 'BTC' |
available |number| Amount available|
reserved|number|Amount on hold (unavailable)|

##GET `/v2/account/protected/balances/<instrumentId>`

GET balance for a specific coin

Note: x-cf-token is required in HTTP header.

> **Request format**

Parameters |Parameter Types | Description| 
-------------------------- | -----|--------- |
instrumentId | String | | Token symbol, e.g. 'BTC' |

> **RESPONSE**

```json
{
    “event”: null,
    “success”: true,
    “message”: null,
    “code”: “0000”,
    “data”:
    {
        "instrumentId": "EOS",
        "available": 325.890,
        "reserved": 1260
    }
}
```


Parameters |Parameter Types | Description| 
-------------------------- | -----|--------- |
instrumentId | String | Token symbol, e.g. 'BTC' |
available|number|Amount available|
reserved|number|Amount on hold (unavailable)|

##GET  `/v2/account/protected/positions/`

GET positions. These are derivative contract positions and not coins.

Example Request:  /v2/markets/protected/positions

Note: x-cf-token is required in HTTP header.
> **RESPONSE**

```json
{
    “event”: null,
    “success”: true,
    “message”: null,
    “code”: “0000”,
    “data”: [
        {
            “instrumentId”: “BTC-USD-200626-LIN”,
            “type”: “FUTURE”,
            “isInverse”: true,
            “quantity”: 2.000000000,
            “lastUpdated”: 1590768853991,
            “contractValCurrency”: null,
            “entryPrice”: 7800.000000000,
            “positionPl”: 0.0
            “estLiquidationPx”: 0.0
        }
    ]
}
```


**ADD estLiquidationPx. Definition here:**
(https://coinflex.atlassian.net/wiki/spaces/CFV2/pages/759398602/Estimated+Liquidation+Price+calculation)

##GET  `/v2/account/protected/<instrumentId>`
Get position for a certain derivative contract.

Example Request: /v2/markets/protected/positions/ETH-USD-PERP-LIN

Note: x-cf-token is required in HTTP header.

**Request**

Parameters |Parameter Types | Description| 
-------------------------- | -----|--------- |
instrumentId | String | Contract ID, e.g. 'ETH-USD-PERP-LIN' |


> **RESPONSE**

```json
   {
    “event”: null,
    “success”: true,
    “message”: null,
    “code”: “0000”,
    “data”: 
        {
            “instrumentId”: “BTC-USD-200626-LIN”,
            “type”: “FUTURE”,
            “isInverse”: true,
            “quantity”: 2.000000000,
            “lastUpdated”: 1590768853991,
            “contractValCurrency”: null,
            “entryPrice”: 7800.000000000,
            “positionPl”: 0.0
            “eqtLiquidationPrice”: 1304.03
        }
}
```

##GET `/v2/markets/public/markets`
Get a list of all markets currently listed on coinflex

Change - Added “referencePair” field

> **RESPONSE**


```json
{
    "event": null,
    "success": true,
    "message": null,
    "code": "0000",
    "data": [
        {
            "marketId": 2001000000000,
            "marketCode": "BTC-USD",
            "name": "BTC/USD Spot",
            "referencePair": "BTC/USD"
            "base": "BTC",
            "counter": "USD",
            "type": "SPOT",
            "tickSize": 0.1,
            "qtyIncrement": 0.001,
            "listingDate": -2208988800000,
            "endDate": null,
            "marginCurrency": null,
            "contractValCurrency": "BTC",
            "upperPriceBound": 11000.00,
            "lowerPriceBound": 9000.00,
            "marketPrice": 10000.00,
            "marketPriceLastUpdated": null,
            "marketPriceStatus": "1",
            "markPrice": null,
            "markPriceLastUpdated": null
        }
    ]
}
```

##GET `/assets/`
Get a list of all assets available on CoinFLEX. These include coins and bookable contracts.

Example Request: /v2/markets/public/assets

> **RESPONSE**


```json
{
    "event": null,
    "success": true,
    "message": null,
    "code": "0000",
    "data": [
        {
            "instrumentId": "BTC-USD-200626-LIN",
            "name": "BTC/USD 20-06-26 Future (Linear)",
            "base": "BTC",
            "counter": "USD",
            "type": "FUTURE",
            "marginCurrency": "USD",
            "contractValCurrency": "BTC",
            "deliveryDate": null,
            "deliveryInstrument": null
        },
        {
            "instrumentId": "BTC",
            "name": "Bitcoin",
            "base": null,
            "counter": null,
            "type": "SPOT",
            "marginCurrency": null,
            "contractValCurrency": null,
            "deliveryDate": null,
            "deliveryInstrument": null
        }
    ]
}
```

##GET `/v2/ohlc/protected/trades/`
Get all trades of current user.

Login Required. Please pass x-cf-token in the HTTP header.

> **RESPONSE**


```json
{
    "event": null,
    "success": true,
    "message": null,
    "code": "0000",
    "data":[ 
    {
        "tradeId": 23534545
        "timestamp": 1588218296439236,
        "marketCode": "BTC-USD"
        "quantity": 1,
        "price": 9171,
        "total": -9171,
        "taker": true,
        "fee": 2.7513,
        "feeInstrumentId": "USD",
        "orderId": 322805624
    }
]
}
 
```
##GET  `/v2/ohlc/protected/orders/`
Get all open orders of current user.

Login Required. Please pass x-cf-token in HTTP header.

> **RESPONSE**


```json
{
    "event": null,
    "success": true,
    "message": null,
    "code": "0000",
    "data":[
   {
      "orderId": <long>,
      "clientOrderId": <long>,
      "marketCode": <String>,
      "side": <string>,
      "quantity": <BigDecimal>,
      "price": <BigDecimal>,
      "timestamp": <long>
    }
  ]
}
 
```
##POST `/auction/`
Delivered positions of current user for auction.

Login Required. Please pass x-cf-token in the HTTP header.

> **RESPONSE**

```json
{
    "event": delivery,
    "success": true,
    "message": null,
    "code": "0000",
    "data":[ 
    {
      "repoId": 12341234123256 (BTC-USD-REPO-LIN),
      "qtyDeliver": 1,
      "instrumentIdDeliver": "BTC",
      "markPrice": 9855,
    }
}
```
##POST `/deliveryWallet/`

Description: Record users' delivery

Login Required. Please pass x-cf-token in the HTTP header.

> **RESPONSE**

```json
{
    "event": delivery,
    "success": true,
    "message": null,
    "code": "0000",
    "data":[ 
    {
      "instrumentId": "BTC-USD-SWAP-LIN",
      "side": "long",
      "quantity": 2.000000000,
      "assetTransfer": 20000 (USD)
      "timestamp": 1558588585858,
    }
}
```


##GET `/deliveryWallet/`

Description: Get info in the “delivery wallet”

Login Required. Please pass x-cf-token in the HTTP header.

> **RESPONSE**

```json
{
    "event": delivery,
    "success": true,
    "message": null,
    "code": "0000",
    "data":[ 
    {
      "timestamp": 1558588585858,
      "instrumentId": "BTC-USD-SWAP-LIN",
      "side": "long",
      "quantity": 2.000000000,
      “entryPrice”: 7800.000000000,
    }
}
```


##GET `/deliver/`

Description: Get position info when user clicks “next delivery cycle”

Login Required. Please pass x-cf-token in the HTTP header.

> **RESPONSE**

```json
{
    "event": delivery,
    "success": true,
    "message": null,
    "code": "0000",
    "data":[ 
    {
      "instrumentId": "BTC-USD-SWAP-LIN",
      "side": "long",
      "markPrice": 9855,
      "base": "BTC",
      "counter": "USD",
      
    }
}
```
