# V2.0 REST API

**TEST** site

* `https://api-test-v2.coinflex-cn.com/ `

**LIVE** site

* 'COMING SOON'

For clients who do not wish to take advantage of CoinFLEX's native WebSocket API, CoinFLEX offers a RESTful API that implements much of the same functionality.

##Authentication 

> **Request format**

```json
{

}
```

Public market data methods do not require authentication, however private methods require a *Signature* to be sent in the header of the request.  These private REST methods  use HMAC SHA256 signatures. 

The HMAC SHA256 signature is a keyed HMAC SHA256 operation using a clients API Secret as the key and a message string as the value for the HMAC operation.  

The message string is made up of the following formula:-

`msgString = Timestamp + "\n" + Nonce + "\n" + Verb + "\n" + URL + "\n" + Path +"\n" + Body`

Component | Required | Example | Description| 
-------------------------- |--------- |------- |------- | 
Timestamp | Yes | 2020-04-30T15:20:30 | YYYY-MM-DDThh:mm:ss
Nonce | Yes | 123 | User generated
Verb | Yes| 'GET' | 
URL | Yes | 'api-test-v2.coinflex-cn.com' |
Path | Yes | '/v2/positions | REST method being called
Body | No | instrumentID=BTC-USD-SWAP-LIN | Optional and dependent on the REST method being called

The constructed message string should look like:-

  `2020-04-30T15:20:30\n
  123\n
  GET\n
  api-test-v2.coinflex-cn.com\n
  /v2/positions\n
  instrumentID=BTC-USD-SWAP-LIN`

Note the newline characters after each component in the message string. 
If *Body* is ommitted it's treated as an empty string.

You must use HmacSHA256 to get the hash with the two parameters: the generated text and your secret key.
Then encode the hash value with Base-64. Mark down the output text as the signature.


##GET `/v2/balances`
> **Request**
```json
GET/v2/balances
```
> **Response**

```json
{
    “event”: "balances",
    "accountId":"<Your Account ID>",
    "timestamp":"1593616622",
    "tradeType":"LINEAR"|"INVERSE",
    "data":[
    {   
        "instrumentId": "BTC",
        "total":"4468.823"              
        "available": "4468.823",        
        "reserved": "0",
        "quantityLastUpdated":"1593616622"
    },
    .....
    {
        "instrumentId": "EOS",
        "total":"1585.890"              
        "available": "325.890",         
        "reserved": "1260",
        "quantityLastUpdated":"1593616500"
    }
    ]
}
```

Requires authentication. GET coin balanaces in your account. 

Parameters |Parameter Types | Description| 
-------------------------- | -----|--------- |
accountId | STRING    | Account ID|
timestamp | STRING    | Timestamp of this response|
tradeType | STRING    | Define this account is trading linear or inverse derivatives|
instrumentId | STRING |Token symbol, e.g. 'BTC' |
total| STRING| Total balance|
available |STRING|Available balance|
reserved|STRING|Reserved balance (unavailable)|


##GET `/v2/balances/<instrumentId>`
> **RESPONSE**

```json
{
    "event": "balancesById",
    "accountId":"<Your Account ID>",
    "timestamp":"1593616622",
    "tradeType":"LINEAR"|"INVERSE",
    "data":[
    {   
        "instrumentId": "BTC",
        "total":"4468.823"              
        "available": "4468.823",        
        "reserved": "0",
        "quantityLastUpdated":"1593616500"
    }
    ]
}
```

Requires authentication. GET balance for a specific coin.

Parameters |Type | Description| 
-------------------------- | -----|--------- |
accountId | STRING    | Account ID|
timestamp | STRING    | Timestamp of this response|
tradeType | STRING    | Define this account is trading linear or inverse derivatives|
instrumentId | STRING |Token symbol, e.g. 'BTC' |
total| STRING| Total balance|
available |STRING|Available balance|
reserved|STRING|Reserved balance (unavailable)|

##GET  `/v2/positions/`

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

GET positions. These are derivative contract positions and not coins.

Example Request:  /v2/markets/protected/positions

##GET  `/v2/positions/<instrumentId>`
Get position for a certain derivative contract.

Example Request: /v2/markets/protected/positions/ETH-USD-PERP-LIN

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

##GET `/v2/all/markets`
Get a list of all markets currently listed on coinflex

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

##GET `/v2/all/assets/`
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

##GET `/v2/trades`
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
##GET  `/v2/orders`
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
