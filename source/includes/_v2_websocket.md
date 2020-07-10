
# Websocket API

> **Request format**

```json
{
  "op": "<value>", 
   "args": ["<value>"]
}

OR
   
{
  "op": "<value>,
  "data": {"<key1>":"<value1>",.....}
}
```

> **Success response format**

```json
{
  "event": "<op value>", 
  "channel": "<args value>",
  "success": true
}

OR

{
  "event": "<op value>",
  "success": true
}
```

> **Failure response format**

```json
{
  "event": "error", 
  "message": "<errorMessage>",
  "code": "<code>",
  "success": false
}
```

**TEST** site

* `wss://api-test-v2.coinflex-cn.com/v2/websocket`

**LIVE** site

* `wss://v2api.coinflex.com/v2/websocket`

CoinFLEX's application programming interface (API) provides our clients programmatic access to control aspects of their accounts and to place orders on the CoinFLEX trading platform. The API is accessible via WebSocket connection to the URIs listed above. Commands, replies, and notifications all traverse the WebSocket in text frames with JSON-formatted payloads.

Websocket commands can be sent in either of the following two formats:

**For subscription based commands**

`{"op": "<value>", "args": ["<value>"]}`

`op`: can either be:

* subscribe
* unsubscribe

`args`: the value is the channel name, for example:

* order:BTC-USD-SWAP-LIN
* futures/depth:BTC-USD-SPR-QP-LIN

**All other commands**

`{"op": "<command>", "data": {"<key1>":"<value1>",.....}}`

`op`: can be:

* login
* placeorder
* cancelorder
* modifyorder

`data`: JSON string of the request object containing the required parameters


## Authentication

> **Request format**

```json
{
  "op": "login",
  "tag": <integer>,
  "data":{
          "apiKey": <string>,
          "timestamp": <string>,
          "signature": <string>
         }
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = 'API-KEY'
api_secret = 'API-SECRET'
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

msg_auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}

async def subscribe():
    async with websockets.connect('wss://api-test-v2.coinflex-cn.com/v2/websocket') as ws:
        await ws.send(json.dumps(msg_auth))
        while ws.open:
            resp = await ws.recv()
            print(resp)

asyncio.get_event_loop().run_until_complete(subscribe())
```
```javascript
var apiKey = "API-KEY";
var secretKey = "API-SECRET";
const ts = '' + Date.now();

var sign = CryptoJS.enc.Base64.stringify(CryptoJS.HmacSHA256(secretKey, ts +'GET/auth/self/verify'));
var msg = '{"op":"login","data":{"apiKey":"' + apiKey + '", "timestamp": "'+ ts + '", "signature":"'+ sign + '"}}';

var ws = new WebSocket('wss://api-test-v2.coinflex-cn.com/v2/websocket');

ws.onmessage = function (e) {
  console.log('websocket message from server : ', e.data);
};

ws.onopen = function () {
    ws.send(msg);
};
```

> **Success response format**

```json
{
  "event": "login",
  "success": true,
  "tag": "1",
  "timestamp": "1592491808"
}
```
```python
{
  "event":"login",
  "success":true,
  "tag":"1",
  "timestamp":"1592491808"
}
```
```javascript
{
  "event":"login",
  "success":true,
  "tag":"1",
  "timestamp":"1592491808"
}
```

> **Failure response format**

```json
{
  "event":"login",
  "success":false,
  "code":"<code>",
  "message":"<errorMessage>",
  "tag":"1",
  "timestamp":"1592492032"
}
```
```python
{
  "event":"login",
  "success":false,
  "code":"<code>",
  "message":"<errorMessage>",
  "tag":"1",
  "timestamp":"1592492032"
}
```
```javascript
{
  "event":"login",
  "success":false,
  "code":"<code>",
  "message":"<errorMessage>",
  "tag":"1",
  "timestamp":"1592492032"
}
```

The Websocket API consists of public and private methods. The public methods do not require authentication.  The private methods requires an authenticated websocket connection.

To autenticate a websocket connection a "login" message must be sent containing the clients signature.

To construct the signature a clients API-Secret key is required.  API keys (public and corresponding secret key) can be generated via the GUI within the clients account.  

The signature is calculated as:

* `Base64(HmacSHA256(API-Secret, timestamp + 'GET/auth/self/verify'))`

**Parameters - Login**

Parameter | Type | Required | Description | 
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | **'login'** |
tag| INTEGER| No | If given, it will be echoed in the reply. |
data | ARRAY object | Yes |
\>apiKey | STRING | Yes | Clients public API key, visible in the GUI when created. | 
\>timestamp | STRING | Yes | Current millisecond timestamp |  
\>signature | STRING | Yes | `Base64(HmacSHA256(API-Secret, timestamp + 'GET/auth/self/verify'))` |


## Place Order

### Limit Order 

> **Request format**

```json
{   
  "op":"placeorder", 
  "data":{
          "clientOrderId":1, 
          "marketCode": "BTC-USD-SWAP-LIN",
          "side":"BUY", 
          "orderType":"LIMIT", 
          "quantity":1.5, 
          "timeInForce":"GTC", 
          "price":9431.48
          },
  "tag":1       
}
```

> **Submitted response format**

```json
{
  "event":"placeorder",
  "submitted":true, 
  "tag":"1", 
  "data":{
          "clientOrderId":"1", 
          "marketCode": "BTC-USD-SWAP-LIN",
          "side":"BUY", 
          "orderType":"LIMIT", 
          "quantity":"1.5", 
          "timeInForce":"GTC", 
          "price":"9431.48"
         },
  "timestamp":"1592491945"
}
```

> **Failure response format**

```json
{
  "event":"placeorder",
  "message":"<errorMessage>",
  "code":"<code>",
  "submitted": false,
  "tag":"1",
  "timestamp":"1592491503"
}
```

Requires authentication. 
Please also subscribe to the user order channel to receive push notifications for all message udpates related to a clients orders.

**Request Parameters Specification:**

Parameters | Type | Required | Description | 
-------------------------- | -----|--------- | -------------|
clientOrderId | INTEGER | No | Customized order ID to identify your orders | 
marketCode | STRING | Yes | Market Code i.e. `BTC-USD-SWAP-LIN` |    
orderType | STRING | Yes |  LIMIT for limit orders | 
price | DECIMAL |  Yes |  Price |    Price for limit orders |
quantity |  DECIMAL | Yes | Quantity (denominated by contractValCurrency) |  
side | STRING | Yes | BUY / SELL | 
timeInForce | ENUM | No | <ul><li>`GTC` (Good-till-Cancel) - Default</li><li> `IOC` (Immediate or Cancel, i.e. Taker-only)</li><li> `FOK` (Fill or Kill, for full size)</li><li>`MAKER_ONLY` (i.e. Post-only)</li><li> `MAKER_ONLY_REPRICE` (Reprices order to the best maker price if specified price otherwise leads to a taker trade)</li></ul>
tag| INTEGER| No|Iff given and non-zero, it will be echoed in the reply.


### Market Order  &nbsp;&nbsp;*****COMING SOON*****


### Stop Limit Order  &nbsp;&nbsp;*****COMING SOON*****

Requires authentication. Please subscribe user order channel to receive the order updates.

> **Request format**

```json
{
  "op":"placeorder", 
  "data":{
          "clientOrderId":1, 
          "marketCode": "BTC-USD-SWAP-LIN", 
          "side":"BUY", 
          "orderType":"STOP", 
          "quantity":10,
          "stopPrice":100,
          "limitPrice":120
         }
}
```

**Request Parameters Specification:**

Parameters | Type | Required |Description| 
-------------------------- | -----|--------- | -------------|
clientOrderId| NUMBER | No | User-generated order ID to identify orders | 
limitPrice| NUMBER|Yes | Limit price for the stop-limit order. <p><p>The limit price must be greater or equal to the stop price for buy-side.<p><p>The limit price must be less or equal to the stop price for sell-side.|
marketCode| STRING| Yes| Market Code i.e. `BTC-USD-SWAP-LIN`| 
orderType|STRING| Yes|  `STOP` for Stop-Limit orders (no Stop-market currently)| 
quantity|NUMBER|Yes|Quantity (denominated by contractValCurrency)|
side|STRING| Yes| `BUY `/ `SELL`| 
stopPrice|NUMBER|Yes|Stop price for the stop-limit order.<p><p>Triggered by the best bid price for the SELL side stop-limit order.<p><p>Triggered by the best ask price for the BUY side stop-limit order. |


### Cancel Order 

> **Request format**

```json
{
  "op":"cancelorder",
  "data":{
          "marketCode":"BTC-USD-SWAP-LIN", 
          "orderId":12
         },
  "tag":1
}
```

Requires authentication. Please subscribe user order channel to receive the order updates.

**Request Parameters Specification:**

Parameters | TYPE | REQUIRED| Description
-------------------------- | -----|--------- | -------------|
marketCode|STRING|YES|Market Code i.e. `BTC-USD-SWAP-LIN`| 
orderId|INTEGER|YES|Order ID is generated by the server.|

> **Submitted response format**

```json
{
  "event":"cancelorder",
  "submitted":true,
  "data":{
          "marketCode":"BTC-USD-SWAP-LIN", 
          "orderId":"12"
         },
  "tag":"1",
  "timestamp":"1592491173"
}
```

> **Failure response format**

```json
{
  "event":"cancelorder",
  "message":"<errorMessage>",
  "code":"<code>",
  "submitted": false,
  "tag":"1",
  "timestamp":"1592491473"
}
```

### Modify Order

> **Request format**

```json
{
  "op":"modifyorder",
  "data":{
          "marketCode":"BTC-USD-SWAP-LIN", 
          "orderId":888,
          "side":"BUY",
          "price":9800,
          "quantity":2,
         },
  "tag":1
}
```

> **Submitted response format**

```json
{
  "event":"modifyorder",
  "submitted":true,
  "data":{
          "marketCode":"BTC-USD-SWAP-LIN", 
          "orderId":888,
          "side":"BUY",
          "price":9800,
          "quantity":2,
         },
  "tag":"1",
  "timestamp":"1592491032"
}
```

> **Failure response format**

```json
{
  "event":"modifyorder",
  "message":"<errorMessage>",
  "code":"<code>",
  "submitted": false,
  "tag":"1",
  "timestamp":"1592491173"
}
```

Requires authentication. Please subscribe user order channel to receive the order updates.

Parameters | Type | Required | Description|
-------------------------- | -----|--------- | -------------|
marketCode|string|Yes|market id| Market Code i.e. `BTC-USD-SWAP-LIN`|
orderId| integer|Yes|Order ID generated by the server|
side| string|Yes| `BUY` or `SELL`| 
price|decimal|Yes|Price for limit orders| 
quantity|decimal|Yes|  Quantity (denominated by `contractValCurrency`)| 


## Subscriptions - Private 
### User Order Channel 

> **Request format**

```json
{"op":"subscribe", 
 "args":["order:BTC-USD"]}
```
Requires authentication. Get the user's order information.

order is the channel name.  BTC-USD is the market code.  Example responses are in Webosocket Private Data Stream Section.

#### Order: OrderOpened

A new limit order has been opened.

Recipients: Authenticated users who subscribed user order channel can receive notice of their own limit orders status.


* Limit order opened:

Parameters | Type | Required 
-------------------------- | -----|--------- | 
notice | STRING | `OrderOpened`
accountId | STRING  | Account identifer   
marketCode | STRING|  Market Code i.e. `BTC-USD-SWAP-LIN`
orderId | STRING |     Order ID generated by the server
clientOrderId |  STRING |  Client order ID submitted   
price | STRING | Price submitted
quantity | STRING| Quantity submitted
side|STRING|`BUY` / `SELL`
timeInForce|STRING|Confirming user setting
status|STRING|  OPEN 
timestamp|STRING|UNIX timestamp
orderType|STRING|`LIMIT`

> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {   
            "notice":"OrderOpened",
            "accountId":<integer>, 
            "marketCode":"BTC-USD-SWAP-LIN",
            "orderId" : "123",  
            "clientOrderId": "16", 
            "price": "9600", 
            "quantity":"2" ,
            "orderType":"LIMIT" 
            "side": "BUY", 
            "timeInForce": "GTC", 
            "status": "OPEN",
            "timestamp":"1592491654"
        }
    ]
}

```


* Stop limit order opened:


> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {   
            "notice":"OrderOpened",
            "accountId":"4", 
            "marketCode":"BTC-USD-SWAP-LIN",
            "orderId":"22", 
            "clientOrderId":"16",  
            "quantity":"2", 
            "orderType":"STOP" 
            "side": "BUY", 
            "stopPrice":"9280",
            "limitPrice":"9300",
            "status": "OPEN",
            "timestamp":"15924910987"
        }
    ]
}

```


Parameters |Type| Required |
--------|-----|---|
notice | STRING| `OrderOpened`
accountId | STRING|  Account identifer   
marketCode | STRING |  Market Code i.e. `BTC-USD-SWAP-LIN`
orderId | STRING |     Order ID generated by the server
clientOrderId |  STRING | Client order ID submitted   
price |STRING | Price submitted
quantity | STRING| Quantity submitted
side|STRING|`BUY` / `SELL`
timeInForce|STRING|Confirming user setting
status|STRING|  OPEN = success
timestamp|STRING |UNIX timestamp
orderType| STRING |`STOP`
stopPrice|STRING|Stop price submitted
limitPrice|STRING|limit price3 submitted



#### Order: OrderModified

* Order modified

```json
{
    "table":"order",
    "data":[
        {   
            "notice":"OrderModified",
            "accountId":"1", 
            "marketCode":"BTC-USD-SWAP-LIN",
            "orderId":"123",
            "clientOrderId":"16", 
            "price":"9600", 
            "quantity":"1", 
            "orderType":"LIMIT" 
            "side": "BUY", 
            "timeInForce": "GTC", 
            "status": "OPEN",
            "timestamp":"1592491223"
        }
    ]
}

```


Parameters |Type| Required |
--------|-----|---|
notice | STRING | `OrderModified`
accountId | STRING | Account identifer   
marketCode | STRING |  Market Code i.e. `BTC-USD-SWAP-LIN`
orderId | STRING |     Order ID generated by the server
clientOrderId |  STRING | Client order ID submitted   
price | STRING | Price submitted
quantity | STRING| Quantity submitted
side|STRING|`BUY` / `SELL`
timeInForce|STRING|Confirming user setting
status|STRING|  OPEN = success
timestamp|STRING|UNIX timestamp
orderType| STRING|`LIMIT`

#### Order: OrderClosed

* Order partially cancelled by IOC

> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderClosed",
          "accountId":"1", 
          "marketCode":"BTC-USD-SWAP-LIN",
          "orderId" :"73",  
          "clientOrderId": "16", 
          "price":"9600", 
          "quantity":"2",
          "remainQuantity":"0",
          "orderType":"LIMIT"
          "side": "BUY", 
          "timeInForce":"IOC", 
          "status": "CANCELED_PARTIAL_BY_IOC",
          "timestamp":"1592496154"
        }
    ]
}


```

Parameters | Type | Required 
-------------------------- | -----|--------- | 
accountId | STRING |  Account identifer  
orderId | STRING |     Order ID generated by the server
orderType|STRING|`LIMIT`
notice | STRING | `OrderClosed`
orderId|STRING|Order ID generated by the server
clientOrderId|STRING|  Client order ID submitted
price|STRING|Price submitted
quantity|STRING|Quantity submitted
remainQuantity|STRING|Remaining quantity
side|STRING|`BUY` / `SELL`
timeInForce|STRING|Confirming user setting
status|STRING|CANCELED PARTIAL BY IOC
timestamp|STRING|UNIX timestamp

* Order fully cancelled by IOC:

> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderClosed"
          "accountId":"1", 
          "marketCode":"BTC-USD-SWAP-LIN",
          "orderId" : "133",  
          "clientOrderId":"16", 
          "price":"8000", 
          "quantity": "2",
          "remainQuantity":"0",
          "orderType":"LIMIT"
          "side": "BUY", 
          "timeInForce": "IOC", 
          "status": "CANCELED_ALL_BY_IOC",
          "timestamp":"1592497654"
        }
    ]
}


```

Parameters | Type | Required 
-------------------------- | -----|--------- | 
accountId | STRING |  Account identifer  
orderId | STRING |     Order ID generated by the server
orderType|STRING|`LIMIT`
notice | STRING | `OrderClosed`
clientOrderId|STRING|  Client order ID submitted
price|STRING|Price submitted
quantity|STRING|Quantity submitted
remainQuantity|STRING|Remaining quantity
side|STRING|`BUY` / `SELL`
timeInForce|STRING|Confirming user setting
status|STRING|cancelled all by IOC
timestamp|STRING|UNIX timestamp

* Order fully cancelled by FOK

> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderClosed",
          "accountId":"1", 
          "marketCode":"BTC-USD-SWAP-LIN",
          "orderId" :"73",  
          "clientOrderId":"16", 
          "price":"9609", 
          "quantity":"2",
          "remainQuantity":"0",
          "orderType":"LIMIT"
          "side": "BUY", 
          "timeInForce": "FOK", 
          "status": "CANCELED_BY_FOK",
          "timestamp":"1592499054"
        }
    ]
}

```

Parameters | Type | Required 
-------------------------- | -----|--------- | 
notice|STRING|`OrderClosed`
accountId | STRING |  Account identifer  
marketCode | STRING|  Market Code i.e. `BTC-USD-SWAP-LIN`
orderId | STRING |     Order ID generated by the server
clientOrderId|STRING |  Client order ID submitted
price|STRING |Price submitted
quantity|STRING |Quantity submitted
remainQuantity|STRING |Remaining quantity
side|STRING |`BUY` / `SELL`
timeInForce|STRING |Confirming user setting
status|STRING |cancelled all by FOK
timestamp|STRING|UNIX timestamp
orderType|STRING |`LIMIT`

* Order fully cancelled by MAKER_ONLY

> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderClosed",
          "accountId":"3", 
          "marketCode":"BTC-USD-SWAP-LIN",
          "orderId" : "7",  
          "clientOrderId":"16", 
          "price":"9600", 
          "quantity":"1",
          "remainQuantity":"0",
          "orderType":"LIMIT"
          "side": "BUY", 
          "timeInForce": "MAKER_ONLY", 
          "status": "CANCELED_BY_MAKER_ONLY",
          "timestamp":"1592492254"
        }
    ]
}

```

Parameters | Type | Required 
-------------------------- | -----|--------- | 
accountId | String |  Account identifer  
orderId | String |   Order ID generated by the server
orderType|string|`LIMIT`
notice | String | `OrderClosed`
orderId|String|Order ID generated by the server
clientOrderId|String|  Client order ID submitted
price|String|Price submitted
quantity|String|Quantity submitted
remainQuantity|String|Remaining quantity
side|String|`BUY` / `SELL`
timeInForce|String|Confirming user setting
status|String|CANCELED BY MAKER ONLY
timestamp|String|UNIX timestamp


*  Limit order cancelled by the user:

> **Notification:**


```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderClosed", 
          "accountId":"1", 
          "marketCode":"BTC-USD-SWAP-LIN",
          "orderId" :"13",  
          "clientOrderId":"16", 
          "price":"9060", 
          "quantity":"2",
          "remainQuantity":"0",
          "orderType":"LIMIT"
          "side": "BUY", 
          "timeInForce": "GTC", 
          "status": "CANCELED_BY_USER",
          "timestamp":"1592492714"
        }
    ]
}
```

Parameters | Type | Required 
-------------------------- | -----|--------- | 
notice | STRING | `OrderClosed`
accountId | STRING  |  Account identifer 
clientOrderId|STRING |  Client order ID submitted
marketCode|STRING |  Market Code i.e. BTC-USD-SWAP-LIN
orderId | STRING  |    Order ID generated by the server
orderType|STRING |`LIMIT`
price|STRING |Price submitted 
quantity|STRING |Quantity submitted
remainQuantity|STRING |Remaining quantity
side|STRING |`BUY` / `SELL`
status|STRING |`cancelled by user`
timeInForce|STRING |Confirming user setting
timestamp|STRING |UNIX timestamp

* stop-limit order cancelled by the user:

> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderClosed", 
          "accountId":"9", 
          "marketCode":"BTC-USD-SWAP-LIN",
          "orderId" :"13",  
          "clientOrderId":"16", 
          "stopPrice":"9100",
          "limitPrice":"9120", 
          "quantity":"2",
          "remainQuantity":"0",
          "orderType":"STOP"
          "side": "BUY", 
          "timeInForce": "GTC", 
          "status": "CANCELED_BY_USER",
          "timestamp":"1592494514"
        }
    ]
}

```

Parameters | Type | Required 
-------------------------- | -----|--------- | 
notice | STRING | `OrderClosed`
accountId | STRING  |  Account identifer 
clientOrderId|STRING |  Client order ID submitted
marketCode|STRING |  Market Code i.e. `BTC-USD-SWAP-LIN`
orderId | STRING  |    Order ID generated by the server
ordertype|STRING  | `STOP`
price|STRING |Price submitted 
quantity|STRING |Quantity submitted
remainQuantity|STRING |Remaining quantity
side|STRING |`BUY` / `SELL`
status|STRING |`cancelled by user`
stopPrice|STRING |stop price submitted
timeInForce|STRING |Confirming user setting
timestamp|STRING |UNIX timestamp



#### Order: OrderRejected

* Reject order with 0 quantity

> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderRejected",
          "quantity":"0",
          "clientOrderID":"13",
          "status":"REJECT_QUANTITY_ZERO",
          "timestamp":"1592496454"
        }
    ]
}

```

Parameters | Type | Required 
-------------------------- | -----|--------- | 
notice | STRING | `OrderRejected`
quantity|STRING|Quantity submitted
status|STRING|REJECT QUANTITY ZERO
timestamp|STRING|UNIX timestamp
clientOrderId|STRING|  Client order ID submitted



* Reject order with unknown order action

> **Notification:**


```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderRejected",
          "status": "REJECT_UNKNOW_ORDER_ACTION",
          "timestamp":"1592495554"
        }
    ]
}

```

* Reject limit order with the market price

> **Notification:**


```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderRejected",
          "status": "REJECT_LIMITE_ORDER_WITH_MARKET_PRICE",
          "timestamp":"1592493454"
        }
    ]
}

```

* Reject stop-limit order

> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderRejected",
          "clientOrderId":"12"
          "orderType":"STOP",
          "side":"BUY",
          "stopPrice":"9630",
          "limitPrice":"9650",
          "status": "STOP_PRICE_LARGER_THAN_LIMIT_PRICE",
          "timestamp":"1592496774"
        }
    ]
}

```


Parameters | Type | Required 
-------------------------- | -----|--------- | 
notice | String | `OrderRejected`
accountId | String |  Account identifer    
clientOrderId|String|  Client order ID submitted
orderType|string|`STOP`
side|String|`BUY` / `SELL`
status|String|STOP PRICE LARGER THAN LIMIT PRIC
stopPrice|String|Stop price submitted
limitPrice|String|Limit price submitted
timestamp|String|UNIX timestamp

> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderRejected",
          "clientOrderId":"12",
          "orderType":"STOP",
          "side":"SELL",
          "stopPrice":"9800",
          "limitPrice":"9900",
          "status": "STOP_PRICE_LESS_THAN_LIMIT_PRICE",
          "timestamp":"1592496574"
        }
    ]
}

```


Parameters | Type | Required 
-------------------------- | -----|--------- | 
notice | String | `OrderRejected`
accountId | String |  Account identifer    
clientOrderId|String|  Client order ID submitted
orderType|string|`STOP`
side|String|`BUY` / `SELL`
status|String|STOP PRICE LARGER THAN LIMIT PRIC
stopPrice|String|Stop price submitted
limitPrice|String|Limit price submitted
timestamp|String|UNIX timestamp


* Reject cancel an order:

> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderRejected",
          "orderId":"13"
          "status": "REJECT_CANCEL_ORDER_ID_NOT_FOUND",
          "timestamp":"1592494474"
        }
    ]
}

```

Parameters | Type | Required 
-------------------------- | -----|--------- | 
notice | STRING |`OrderReject`
orderId | STRING |    Order ID generated by the server
status|STRING|REJECT CANCEL ORDER ID NOT FOUND
timestamp|STRING|UNIX timestamp

* Reject modify the order

> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderRejected",
          "orderId":"12",
          "status": "REJECT_AMEND_ORDER_ID_NOT_FOUND",
          "timestamp":"1592498890"
        }
    ]
}

```

Parameters |Type| Required |
--------|-----|---|
notice | STRING | `OrderRejected`
orderId | STRING  |     Order ID generated by the server
status|STRING |  REJECT AMEND ORDER ID NOT FOUND
timestamp|STRING  |UNIX timestamp


#### Order: OrderMatched

* Limit order fully matched

> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderMatched",
          "accountId":"1", 
          "marketCode":"BTC-USD-SWAP-LIN",
          "orderId" :"39",  
          "clientOrderId":"16", 
          "matchId":"1", 
          "price":"9300", 
          "quantity":"2",
          "matchPrice":"9300",
          "matchQuantity":"2",
          "orderMatchType":"MAKER",
          "remainQuantity":"0",
          "orderType":"LIMIT"
          "side": "BUY", 
          "timeInForce": "GTC", 
          "fees":"3.7"
          "feeInstrumentId":"FLEX",
          "status": "FILLED",
          "timestamp":"1592490254"
        }
    ]
}


```


Parameters | Type | Required 
-------------------------- | -----|--------- | 
notice | STRING | `OrderMatched`
accountId | STRING | Account identifer  
marketCode|STRING|      Market Code i.e. `BTC-USD-SWAP-LIN` 
orderId | STRING|     Order ID generated by the server
orderId|STRING|Order ID generated by the server
clientOrderId|STRING|  Client order ID submitted
price|STRING|Price submitted
quantity|STRING|Quantity submitted
remainQuantity|STRING|Remaining quantity
side|STRING|`BUY` / `SELL`
timeInForce|STRING|Confirming user setting
status|STRING|`FILL`
timestamp|STRING|UNIX timestamp
orderType|STRING|`LIMIT`


* Limit order partially matched


> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderMatched",
          "accountId":"11", 
          "marketCode":"BTC-USD-SWAP-LIN",
          "orderId" :"13",  
          "clientOrderId":"16", 
          "matchId":"2", 
          "price":"9333", 
          "quantity":"2",
          "matchPrice":"9333",
          "matchQuantity":"1",
          "orderMatchType":"MAKER",
          "remainQuantity":"1",
          "orderType":"LIMIT"
          "side": "BUY", 
          "timeInForce": "GTC", 
          "fees":"1.8"
          "feeInstrumentId":"FLEX",
          "status": "PARTIAL_FILL",
          "timestamp":"1592497054"
        }
    ]
}


```



Parameters | Type | Required 
-------------------------- | -----|--------- | 
notice | STRING | `OrderOpened`
accountId | STRING | Account identifer    
orderId | STRING |     Order ID generated by the server
fees|STRING|Fee amount
feeInstrumentId|STRING|Fee instrument (e.g. USD)
marketCode|STRING|  Market Code i.e. BTC-USD-SWAP-LIN
matchId|STRING|Match/trade identifier
matchPrice|STRING|Traded price
matchQuantity|STRING|Traded quantity
orderId|STRING|Order ID generated by the server
orderMatchType|STRING|`MAKER`/`TAKER`
orderType|STRING|`LIMIT`
price|STRING|Price submitted
quantity|STRING|Quantity submitted
remainQuantity|STRING|Remaining quantity
side|STRING|`BUY` / `SELL`
status|STRING|`PARTIAL_FILL`
timeInForce|STRING|Confirming user setting
timestamp|STRING|UNIX timestamp

* Stop limit order partially matched:

> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderMatched",
          "accountId":"7", 
          "marketCode":"BTC-USD-SWAP-LIN",
          "orderId" :"12",  
          "clientOrderId":"16", 
          "matchId":"1", 
          "stopPrice":"9700",
          "limitPrice":"9650",
          "quantity":"3",
          "matchPrice":"9650",
          "matchQuantity":"1",
          "orderMatchType":"MAKER",
          "remainQuantity":"2",
          "orderType":"STOP"
          "side": "BUY", 
          "timeInForce": "GTC", 
          "fees":"1.9",
          "feeInstrumentId":"FLEX",
          "status": "PARTIAL_FILL",
          "timestamp":"15924979954"
        }
    ]
}

```

Parameters | Type | Required 
-------------------------- | -----|--------- | 
notice | STRING | `OrderOpened`
accountId | STRING | Account identifer    
clientOrderId|STRING|  Client order ID submitted
marketCode|STRING|  Fee instrument (e.g. USD)
orderId | STRING |    Order ID generated by the server
matchId|STRING|Match/trade identifier
stopPrice|STRING|Stop price submitted
limitPrice|STRING|Limit price submitted
quantity|STRING|Quantity submitted
matchPrice|STRING|Traded price
matchQuantity|STRING|Traded quantity
orderMatchType|STRING|`MAKER`/`TAKER`
remainQuantity|STRING|Remaining quantity
side|STRING|`BUY` / `SELL`
timeInForce|STRING|Confirming user setting
status|STRING|`PARTIAL_FILL`
timestamp|STRING|UNIX timestamp
fees|STRING|Fee amount
feeInstrumentId|STRING|Fee instrument (e.g. USD)
orderType|STRING|`STOP`

* Stop-limit order fully matched:

> **Notification:**

```json
 
{
    "table":"order",
    "data":[
        {
          "notice":"OrderMatched",
          "accountId":"1", 
          "marketCode":"BTC-USD-SWAP-LIN",
          "orderId" : "123",  
          "clientOrderId":"16", 
          "matchId":"1", 
          "stopPrice":"9700",
          "limitPrice":"9650",
          "quantity": "2",
          "matchPrice":"9650",
          "matchQuantity":"2",
          "orderMatchType":"MAKER",
          "remainQuantity":"0",
          "orderType":"STOP"
          "side": "BUY", 
          "timeInForce": "GTC", 
          "fees":"3.8",
          "feeInstrumentId":"FLEX",
          "status": "FILLED",
          "timestamp":"15924979954"
        }
    ]
}

```


Parameters | Type | Required 
-------------------------- | -----|--------- | 
notice | STRING | `OrderMatched`
accountId | STRING | Account identifer    
marketCode|STRING|  Market Code i.e. `BTC-USD-SWAP-LIN`
orderId | STRING |    Order ID generated by the server
clientOrderId|STRING|  Client order ID submitted
matchId|STRING|Match/trade identifier
stopPrice|STRING|Stop price submitted
limitPrice|STRING|Limit price submitted
quantity|STRING|Quantity submitted
matchPrice|STRING|Traded price
matchQuantity|STRING|Traded quantity
orderMatchType|STRING|`MAKER`/`TAKER`
remainQuantity|SSTRING|Remaining quantity
side|STRING|`BUY` / `SELL`
timeInForce|STRING|Confirming user setting
status|STRING|`FILLED`
timestamp|STRING|UNIX timestamp
fees|STRING|Fee amount
feeInstrumentId|STRING|Fee instrument (e.g. USD)
orderType|STRING|`STOP`


## Subscriptions - Public 
### Orderbook Depth

> **Request**

```json
{
  "op": "subscribe", 
  "args":["futures/depth:BTC-USD-190628"]
}
```

> **Response**

```json
{
    "table": "futures/depth",
    "data": [{
        "asks": [
            ["5556.82", "11", "0", "0"],
            ["5556.84", "98", "0", "0"],
            ["5556.92", "1", "0", "0"],
            ["5557.6", "4", "0", "0"],
            ["5557.85", "2", "0", "0"]
        ],
        "bids": [
            ["5556.81", "1", "0", "0"],
            ["5556.8", "2", "0", "0"],
            ["5556.79", "1", "0", "0"],
            ["5556.19", "100", "0", "0"],
            ["5556.08", "2", "0", "0"]
        ],
        "instrumentId": "BTC-USD-190628",
        "timestamp": "2019-05-06T07:19:39.348Z"
    }]
}
```

**Description**

futures/depth is channel name ，BTC-USD-190628 is instrumentId

bids and asks value example:

["411.8", "10", "0", "0"] 411.8 is the price; 10 is the quantity.

Parameters | Parameters Types | Description| 
-------------------------- | -----| -------------|
bids| List<String>|Buy side depth |
instrumentId| String|Contract ID，e.g .BTC-USD-170310 ,BTC-USDT-191227|
asks|List|Sell side depth
timestamp|String|UNIX timestamp

### Trade

> **Request format**

```json

{"op": "subscribe", "args":["trade:BTC-USD-SWAP-LIN"],"tag":1}

```

> **Success response format**

```json

{"event": "subscribe", "channel":["trade:BTC-USD-SWAP-LIN"],"success":true,"tag":"1","timestamp":"1594299886880"}

```

> **Failure response format**

```json

{"event": "subscribe", "message": "<errorMessage>","code": "<code>","success": false,"tag":"1","timestamp":"1594299886880“}

```

> **Channel update format**

```json

{
    "table": "trade",
    "data": [{
        "side": "buy",
        "tradeId": "2778148208082945",
        "price": "5556.91",
        "quantity": "5",
        "marketCode": "BTC-USD-190628",
        "timestamp": "1594299886890"
    }]
}

```
The trade channel pushes the matched order data.

**Channel Name** : trade:\<marketCode\>

**Update Speed** : real-time

Request Parameters |Type | Required| Description| 
-------------------------- | -----|--------- |-----------|
tag |INTEGER| NO | Iff given and non-zero, it will be echoed in the reply.

Update Parameters |Type | Description| 
-------------------------- | -----|--------- |
tradeId   | STRING    | Transaction Id|
price | STRING    | Matched price|
quantity|STRING   | Matched quantity|
side    |STRING   | Matched side|
timestamp| STRING | Matched timestamp|
marketCode| STRING | Market code i.e BTC-USD|

### Ticker

> **Request format**

```json

{"op": "subscribe", "args":["ticker:BTC-USD-200925-LIN"],"tag":1} 

```

> **Success response format**

```json

{"event": "subscribe", "channel":["ticker:BTC-USD-200925-LIN"],"success":true,"tag":"1","timestamp":"1594299886890"}

```

> **Failure response format**

```json

{"event": "subscribe", "message": "<errorMessage>","code": "<code>","success": false,"tag":"1","timestamp":"1594299886890“}

```

> **Channel update format**

```json

{
    "table":"ticker",
    "data":[
        {
            "marketCode":"BTC-USD-SWAP-LIN",
            "last":"43.259", 
            "markPrice": "11012.80409769",   
            "open24h":"49.375",
            "volume24h":"11295421",
            "currencyVolume24h":"1225512",                       
            "high24h":"49.488",
            "low24h":"41.649",
            "openInterest":"1726003",
            "lastQty":"1"
            "timestamp":"123443563454",
        }
    ]
}

```
The ticker channel pushes the general information about the contract.

**Channel Name** : ticker:\<marketCode\>

**Update Speed** : 100ms

Request Parameters |Type | Required| Description| 
-------------------------- | -----|--------- |-----------|
tag |INTEGER| NO | Iff given and non-zero, it will be echoed in the reply.

Update Parameters |Type | Description| 
-------------------------- | -----|--------- |
marketCode    | STRING   | Market code i.e BTC-USD|
last          | STRING   | Last traded price|
markPrice     | STRING   | Matched quantity|
open24h       | STRING   | 24 hour rolling opening price|
volume24h     | STRING   | 24 hour rolling trading volume in counter currency |
currencyVolume24h     | STRING   | 24 hour rolling trading volume in base currency|
high24h     | STRING   | 24 hour highest price|
low24h     | STRING   | 24 hour lowest price|
openInterest     | STRING   | Open Interest|
lastQty     | STRING   | Last traded price amount|
timestamp   | STRING   | Timestamp|


### Candles

> **Request format**

```json

{"op": "subscribe", "args":["candles60s:BTC-USD-SWAP-LIN"],"tag":1} 

```

> **Success response format**

```json

{"event": "subscribe", "channel":["candles60s:BTC-USD-SWAP-LIN"],"success":true,"tag":"1","timestamp":"1594313762698"}

```

> **Failure response format**

```json

{"event": "subscribe", "message": "<errorMessage>","code": "<code>","success": false,"tag":"1","timestamp":"1594313762698“}

```

> **Channel update format**

```json

{
    "table":"candle60s",
    "data":[
        {
            "candle":[
                "1594313762698", //timestamp
                "9633.1",        //open
                "9693.9",        //high
                "9238.1",        //low
                "9630.2",        //close
                "45247",         //volume
                "5.3"            //currencyVolume
            ],
            "marketCode":"BTC-USD-SWAP-LIN"
        }
    ]
}

```
The candles channel pushes the K-line.

**Channel Name** : candles\<granularity\>:\<marketCode\>

**Update Speed** : 500ms

**Granularity**  : 60s, 180s, 300s, 900s, 1800s, 3600s, 7200s, 14400s, 21600s, 43200s, 86400s

Request Parameters |Type | Required| Description| 
-------------------------- | -----|--------- |-----------|
tag |INTEGER| NO | Iff given and non-zero, it will be echoed in the reply.

Update Parameters |Type | Description| 
-------------------------- | -----|--------- |
marketCode    | STRING   | Market code i.e BTC-USD|
candle        | STRING   | Timestamp, open price, highest price, lowest price, close price, trading volume in counter currency, and trading volume in base currency|
