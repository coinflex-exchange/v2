
# Websocket API

> **Request format**

```json
{
  "op": "<value>",
  "args": ["<value1>", "<value2>",.....]
}

OR

{
  "op": "<value>",
  "data": {"<key1>": "<value1>",.....}
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

* `wss://v2stgapi.coinflex.com/v2/websocket`

**LIVE** site

* `wss://v2api.coinflex.com/v2/websocket`

CoinFLEX's application programming interface (API) provides our clients programmatic access to control aspects of their accounts and to place orders on the CoinFLEX trading platform. The API is accessible via WebSocket connection to the URIs listed above. Commands, replies, and notifications all traverse the WebSocket in text frames with JSON-formatted payloads.

Websocket commands can be sent in either of the following two formats:

**For subscription based commands**

`{"op": "<value>", "args": ["<value1>", "<value2>",.....]}`

`op`: can either be:

* subscribe
* unsubscribe

`args`: the value(s) will be the instrument ID(s) or asset ID(s), for example:

* order:BTC-USD-SWAP-LIN
* futures/depth:ETH-USD-REPO-LIN
* position:all

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
  "tag": "<integer>",
  "data": {
            "apiKey": "<string>",
            "timestamp": "<string>",
            "signature": "<string>"
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
    async with websockets.connect('wss://v2stgapi.coinflex.com/v2/websocket') as ws:
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

var ws = new WebSocket('wss://v2stgapi.coinflex.com/v2/websocket');

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
  "timestamp": "1592491803978"
}
```
```python
{
  "event": "login",
  "success": true,
  "tag": "1",
  "timestamp": "1592491808328"
}
```
```javascript
{
  "event": "login",
  "success": true,
  "tag": "1",
  "timestamp": "1592491808329"
}
```

> **Failure response format**

```json
{
  "event": "login",
  "success": false,
  "code": "<code>",
  "message": "<errorMessage>",
  "tag": "1",
  "timestamp": "1592492069732"
}
```
```python
{
  "event": "login",
  "success": false,
  "code": "<code>",
  "message": "<errorMessage>",
  "tag": "1",
  "timestamp": "1592492031972"
}
```
```javascript
{
  "event": "login",
  "success": false,
  "code": "<code>",
  "message": "<errorMessage>",
  "tag": "1",
  "timestamp": "1592492031982"
}
```

The Websocket API consists of public and private methods. The public methods do not require authentication.  The private methods requires an authenticated websocket connection.

To autenticate a websocket connection a "login" message must be sent containing the clients signature.

To construct the signature a clients API-Secret key is required.  API keys (public and corresponding secret key) can be generated via the GUI within the clients account.

The signature is calculated as:

* `Base64(HmacSHA256(API-Secret, timestamp + 'GET/auth/self/verify'))`

Parameter | Type | Required | Description |
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | **'login'** |
tag| INTEGER| No | If given, it will be echoed in the reply |
data | ARRAY object | Yes |
\>apiKey | STRING | Yes | Clients public API key, visible in the GUI when created |
\>timestamp | STRING | Yes | Current millisecond timestamp |
\>signature | STRING | Yes | `Base64(HmacSHA256(API-Secret, timestamp + 'GET/auth/self/verify'))` |

## Session Keep Alive

To maintain an active WebSocket connection it is imperative to either be subscribed to a channel that pushes data at least once per minute (Depth or Balance) or send a ping to the server once per minute.

## Order Commands

### Limit Order

> **Request format**

```json
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "clientOrderId": 1,
            "marketCode": "BTC-USD-SWAP-LIN",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": 1.5,
            "timeInForce": "GTC",
            "price": 9431.48
          }
}
```

> **Success response format**

```json
{
  "event": "placeorder",
  "submitted": true,
  "tag": "123",
  "timestamp": "1592491945248",
  "data": {
            "clientOrderId": "1",
            "marketCode": "BTC-USD-SWAP-LIN",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "1.5",
            "timeInForce": "GTC",
            "price": "9431.48"
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": false,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<code>",
  "submitted": false,
  "timestamp": "1592491945248",
  "data": {
            "clientOrderId": "1",
            "marketCode": "BTC-USD-SWAP-LIN",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "1.5",
            "timeInForce": "GTC",
            "price": "9431.48"
          }
}
```

Requires an authenticated websocket connection.  
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

Parameter | Type | Required | Description |
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `placeorder`
clientOrderId | INTEGER | No | Client assigned ID to help manage and identify orders |
marketCode | STRING | Yes | Market code e.g. `BTC-USD-SWAP-LIN` |
orderType | STRING | Yes |  `LIMIT` |
price | DECIMAL |  No | Price |
quantity |  DECIMAL | Yes | Quantity (denominated by contractValCurrency) |
side | STRING | Yes | `BUY` or `SELL` |
timeInForce | ENUM | No | <ul><li>`GTC` (Good-till-Cancel) - Default</li><li> `IOC` (Immediate or Cancel, i.e. Taker-only)</li><li> `FOK` (Fill or Kill, for full size)</li><li>`MAKER_ONLY` (i.e. Post-only)</li><li> `MAKER_ONLY_REPRICE` (Reprices order to the best maker only price if the specified price were to lead to a taker trade)</li></ul>
tag| INTEGER| No|If given and non-zero, it will be echoed in the reply

### Market Order

> **Request format**

```json
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "clientOrderId": 1,
            "marketCode": "ETH-USD-SWAP-LIN",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": 5
          }
}
```

> **Success response format**

```json
{
  "event": "placeorder",
  "submitted": true,
  "tag": "123",
  "timestamp": "1592491945248",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-USD-SWAP-LIN",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": "5"
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": false,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<code>",
  "submitted": false,
  "timestamp": "1592491503359"
}
```

Requires an authenticated websocket connection.  
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

Parameter | Type | Required | Description |
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `placeorder`
clientOrderId | INTEGER | No | Client assigned ID to help manage and identify orders |
marketCode | STRING | Yes | Market code e.g. `BTC-USD-SWAP-LIN` |
orderType | STRING | Yes |  `MARKET` |
quantity |  DECIMAL | Yes | Quantity (denominated by contractValCurrency) |
side | STRING | Yes | `BUY` or `SELL` |
tag| INTEGER| No|If given and non-zero, it will be echoed in the reply


### Stop Limit Order

> **Request format**

```json
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "clientOrderId": 1,
            "marketCode": "ETH-USD-SWAP-LIN",
            "side": "BUY",
            "orderType": "STOP",
            "quantity": 10,
            "timeInForce": "MAKER_ONLY_REPRICE",
            "stopPrice": 100,
            "limitPrice": 120
         }
}
```

> **Success response format**

```json
{
  "event": "placeorder",
  "submitted": true,
  "tag": "123",
  "timestamp": "1607639739098",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-USD-SWAP-LIN",
            "side": "BUY",
            "orderType": "STOP",
            "quantity": "10",
            "timeInForce": "MAKER_ONLY_REPRICE",
            "stopPrice": "100",
            "limitPrice": "120"
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": false,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<code>",
  "timestamp": "1592491503359",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-USD-SWAP-LIN",
            "side": "BUY",
            "orderType": "STOP",
            "quantity": "10",
            "timeInForce": "MAKER_ONLY_REPRICE",
            "stopPrice": "100",
            "limitPrice": "120"
          }
}
```

Requires an authenticated websocket connection.  
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

Parameters | Type | Required |Description|
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `placeorder`
clientOrderId | INTEGER | No | Client assigned ID to help manage and identify orders |
marketCode| STRING| Yes| Market code e.g. `ETH-USD-SWAP-LIN`|
orderType|STRING| Yes|  `STOP` for stop-limit orders (stop-market orders not supported)|
quantity|DECIMAL|Yes|Quantity (denominated by contractValCurrency)|
side|STRING| Yes| `BUY ` or `SELL`|
limitPrice| DECIMAL|Yes | Limit price for the stop-limit order. <p><p>For **BUY** the limit price must be greater or equal to the stop price.<p><p>For **SELL** the limit price must be less or equal to the stop price.|
stopPrice|DECIMAL|Yes|Stop price for the stop-limit order.<p><p>Triggered by the best bid price for the **SELL** stop-limit order.<p><p>Triggered by the best ask price for the **BUY** stop-limit order. |
timeInForce | ENUM | No | <ul><li>`GTC` (Good-till-Cancel) - Default</li><li> `IOC` (Immediate or Cancel, i.e. Taker-only)</li><li> `FOK` (Fill or Kill, for full size)</li><li>`MAKER_ONLY` (i.e. Post-only)</li><li> `MAKER_ONLY_REPRICE` (Reprices order to the best maker only price if the specified price were to lead to a taker trade)</li></ul>
tag| INTEGER| No|If given and non-zero, it will be echoed in the reply


### Place Batch Orders

> **Request format**

```json
{
  "op": "placeorders",
  "tag": 123,
  "dataArray": [{
                  "clientOrderId": 1,
                  "marketCode": "ETH-USD-SWAP-LIN",
                  "side": "BUY",
                  "orderType": "LIMIT",
                  "quantity": 10,
                  "timeInForce": "MAKER_ONLY",
                  "price": 100
                }, 
                {
                  "clientOrderId": 2,
                  "marketCode": "BTC-USD",
                  "side": "SELL",
                  "orderType": "MARKET",
                  "quantity": 0.2
                }]
}
```

> **Success response format**

```json
{
  "event": "placeorder",
  "submitted": true,
  "tag": "123",
  "timestamp": "1607639739098",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-USD-SWAP-LIN",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "10",
            "timeInForce": "MAKER_ONLY",
            "price": "100"
          }
}

AND

{
  "event": "placeorder",
  "submitted": true,
  "tag": "123",
  "timestamp": "1607639739136",
  "data": {
            "clientOrderId": "2",
            "marketCode": "BTC-USD",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": "0.2"
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": false,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<code>",
  "timestamp": "1592491503359",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-USD-SWAP-LIN",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "10",
            "timeInForce": "MAKER_ONLY",
            "price": "100"
          }
}

AND

{
  "event": "placeorder",
  "submitted": false,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<code>",
  "timestamp": "1592491503457",
  "data": {
            "clientOrderId": "2",
            "marketCode": "BTC-USD",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": "0.2"
          }
}
```

Requires an authenticated websocket connection.  
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

All existing single order placement methods are supported:-

* LIMIT
* MARKET
* STOP

The websocket reply from the exchange will repond to each order in the batch seperately, one order at a time, and has the same message format as the reponse for the single order placement method.

Parameters | Type | Required |Description|
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `placeorders`
tag| INTEGER| No|If given and non-zero, it will be echoed in the reply
dataArray | ARRAY Object | Yes | An array of orders with each order in JSON format, the same format as the method for placing single orders.  The max number of orders is still limited by the message length validation so by default up to 20 orders can be placed in a batch, assuming that each order JSON has 200 characters.


### Cancel Order

> **Request format**

```json
{
  "op": "cancelorder",
  "tag": 456,
  "data": {
            "marketCode": "BTC-USD-SWAP-LIN",
            "orderId": 12
          }
}
```

> **Success response format**

```json
{
  "event": "cancelorder",
  "submitted": true,
  "tag": "456",
  "timestamp": "1592491173964",
  "data": {
            "marketCode": "BTC-USD-SWAP-LIN",
            "orderId": "12"
          }
}
```

> **Failure response format**

```json
{
  "event": "cancelorder",
  "submitted": false,
  "tag": "456",
  "message": "<errorMessage>",
  "code": "<code>",
  "timestamp": "1592491173964",
  "data": {
            "marketCode": "BTC-USD-SWAP-LIN",
            "orderId": "12"
          }
}
```

Requires an authenticated websocket connection.  
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderClosed etc......).

Parameters | Type | Required | Description
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `cancelorder`
marketCode|STRING|Yes|Market code e.g. `BTC-USD-SWAP-LIN`|
orderId|INTEGER|Yes|Unique order ID from the exchange|


### Cancel Batch Order

> **Request format**

```json
{
  "op": "cancelorders",
  "tag": 456,
  "dataArray": [{
                  "marketCode": "BTC-USD-SWAP-LIN",
                  "orderId": 12
                },
                {
                  "marketCode": "BCH-USD",
                  "orderId": 34
                }]
}
```

> **Success response format**

```json
{
  "event": "cancelorder",
  "submitted": true,
  "tag": "456",
  "timestamp": "1592491173964",
  "data": {
            "marketCode": "BTC-USD-SWAP-LIN",
            "orderId": "12"
          }
}

AND

{
  "event": "cancelorder",
  "submitted": true,
  "tag": "456",
  "timestamp": "1592491173978",
  "data": {
            "marketCode": "BCH-USD",
            "orderId": "34"
          }
}

```

> **Failure response format**

```json
{
  "event": "cancelorder",
  "submitted": false,
  "tag": "456",
  "message": "<errorMessage>",
  "code": "<code>",
  "timestamp": "1592491173964",
  "data": {
            "marketCode": "BTC-USD-SWAP-LIN",
            "orderId": "12"
          }
}

AND

{
  "event": "cancelorder",
  "submitted": false,
  "tag": "456",
  "message": "<errorMessage>",
  "code": "<code>",
  "timestamp": "1592491173989",
  "data": {
            "marketCode": "BCH-USD",
            "orderId": "12"
          }
}
```

Requires an authenticated websocket connection.  
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderClosed etc......).

Parameters | Type | Required | Description
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `cancelorders`
marketCode|STRING|Yes|Market code e.g. `BTC-USD-SWAP-LIN`|
orderId|INTEGER|Yes|Unique order ID from the exchange|


### Modify Order

> **Request format**

```json
{
  "op": "modifyorder",
  "data":{
          "marketCode": "BTC-USD-SWAP-LIN",
          "orderId": 888,
          "side": "BUY",
          "price": 9800,
          "quantity": 2
         },
  "tag": 1
}
```

> **Success response format**

```json
{
  "event": "modifyorder",
  "submitted": true,
  "tag": "1",
  "timestamp": "1592491032427",
  "data":{
          "orderId": 888,
          "side": "BUY",
          "quantity": 2
          "price": 9800,
          "orderType": "LIMIT",
          "marketCode": "BTC-USD-SWAP-LIN"
         }
}
```

> **Failure response format**

```json
{
  "event": "modifyorder",
  "submitted": false,
  "tag": "1",
  "message": "<errorMessage>",
  "code": "<code>",
  "timestamp": "1592491032427",
  "data": {
            "orderId": 888,
            "side": "BUY",
            "quantity": 2,
            "price": 9800,
            "marketCode": "BTC-USD-SWAP-LIN"
          }
}
```

Requires an authenticated websocket connection.  
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderModified etc......).

Currently only LIMIT orders are supported by the modify order command.

* The price and/or quantity and/or side of an order can be modified.
* Reducing the quantity will leave the modified orders position in the order queue **unchanged**.
* Increasing the quantity will **always** move the modified order to the back of the order queue.
* Modifying the price will **always** move the modified order to the back of the order queue.
* Modifying the side will **always** move the modified order to the back of the order queue.

Please be aware that modifying the side of an existing GTC LIMIT order from BUY to SELL or vice versa **without** modifying the price could result in the order matching immediately since its quite likely the new order will become an agressing taker order.  

Parameters | Type | Required | Description|
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `modifyorder`
marketCode|STRING|Yes| Market code e.g. `BTC-USD-SWAP-LIN`|
orderId|INTEGER|Yes|Unique order ID from the exchange|
side| STRING|No| `BUY` or `SELL`|
price|DECIMAL|No|Price for limit orders|
quantity|DECIMAL|No|  Quantity (denominated by `contractValCurrency`)|


### Modify Batch Orders

> **Request format**

```json
{
  "op": "modifyorders",
  "tag": 123,
  "dataArray": [{
                  "marketCode": "ETH-USD-SWAP-LIN",
                  "side": "BUY",
                  "orderID": 304304315061932310,
                  "price": 101,
                }, 
                {
                  "marketCode": "BTC-USD",
                  "orderID": 304304315061864646,
                  "price": 10001,
                  "quantity": 0.21
                }]
}
```

> **Success response format**

```json
{
  "event": "modifyorder",
  "submitted": true,
  "tag": "123",
  "timestamp": "1607639739098",
  "data": {
            "orderId": "304304315061932310",
            "side": "BUY",
            "quantity": "5"
            "price": "101",
            "orderType": "LIMIT",
            "marketCode": "ETH-USD-SWAP-LIN"
          }
}

AND

{
  "event": "modifyorder",
  "submitted": true,
  "tag": "123",
  "timestamp": "1607639739136",
  "data": {
            "orderId": "304304315061864646",
            "side": "SELL",
            "quantity": 0.21,
            "price": "10001",
            "orderType": "LIMIT",
            "marketCode": "BTC-USD"
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": false,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<code>",
  "timestamp": "1592491503359",
  "data": {
            "orderID": 304304315061932310,
            "side": "BUY",
            "price": 101,
            "marketCode": "ETH-USD-SWAP-LIN"                 
          }
}

AND

{
  "event": "placeorder",
  "submitted": false,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<code>",
  "timestamp": "1592491503457",
  "data": {
            "orderID": 304304315061864646,
            "quantity": 0.21,
            "price": 10001,
            "marketCode": "BTC-USD"  
          }
}
```

Requires an authenticated websocket connection.  
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

The websocket reply from the exchange will repond to each order in the batch seperately, one order at a time, and has the same message format as the reponse for the single order modify method.

Parameters | Type | Required |Description|
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `modifyorders`
tag| INTEGER| No|If given and non-zero, it will be echoed in the reply
dataArray | ARRAY Object | Yes | An array of orders with each order in JSON format, the same format as the method for modifying single orders.  The max number of orders is still limited by the message length validation so by default up to 20 orders can be modified in a batch, assuming that each order JSON has 200 characters.


## Subscriptions - Private

All subscriptions to private account channels requires an authenticated websocket connection.  

Multiple subscriptions to different channels can be made within a single subscription command: 

`{"op": "subscribe", "args": ["<value1>", "<value2>",.....]}`


### Balance Channel

> **Request format**

```json
{
  "op": "subscribe",
  "args": ["balance:all"],
  "tag": 101
}

OR

{
  "op": "subscribe", 
  "args": ["balance:USD", "balance:FLEX", ........], 
  "tag": 101
}
```

> **Success response format**

```json
{
  "success": True, 
  "tag": "101", 
  "event": "subscribe", 
  "channel": "<args value>", 
  "timestamp": "1607985371401"
}
```

> **Balance channel format**

```json
{
  "table": "balance",
  "accountId": "<Your account ID>",
  "timestamp": "1599693365059",
  "tradeType": "Linear",
  "data": [ {
              "total": "10000",
              "reserved": "1000",
              "instrumentId": "USD",
              "available": "9000",
              "quantityLastUpdated": "1599694369431"
            },
            {
              "total": "100000",
              "reserved": "0",
              "instrumentId": "FLEX",
              "available": "100000",
              "quantityLastUpdated": "1599694343242"
            }, 
            ........
          ]
}
```

**Channel Update Frequency** : 250ms

The websocket will reply with the shown success response format for EACH balance asset channel which has been successfully subscribed to.

If a subscription has been made to balance:all, the data array in the message from this balance channel will contain a JSON **list**. Each JSON will contain balance details for each spot asset.  Otherwise the data array will contain a **single** JSON corresponding to one spot asset per asset channel subscription.

**Request Parameters**

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes |  `subscribe`
args | ARRAY | Yes | `balance:all` or a list of individual assets `balance:<assetId>`
tag | INTEGER | No | If given and non-zero, it will be echoed in the reply


**Channel Update Parameters**

Parameters |Type| Description |
--------|-----|---|
table | STRING| `balance`
accountId | STRING|  Account identifier
timestamp|STRING | Current millisecond timestamp
tradeType|STRING | `LINEAR`
total | STRING | Total spot asset balance
reserved | STRING | Reserved asset balance for working spot and repo orders
instrumentId | STRING |  Base asset ID e.g. `BTC`
available | STRING| Remaining available asset balance (total - reserved)
quantityLastUpdated|STRING | Millisecond timestamp

### Position Channel

> **Request format**

```json
{
  "op": "subscribe", 
  "args": ["position:all"], 
  "tag": 102
}

OR

{
  "op": "subscribe",
  "args": ["position:BTC-USD-SWAP-LIN", "position:BCH-USD-SWAP-LIN", ........], 
  "tag": 102
}
```

> **Success response format**

```json
{
  "success": True, 
  "tag": "102", 
  "event": "subscribe", 
  "channel": "<args value>", 
  "timestamp": "1607985371401"
}
```

> **Position channel format**

```json
{
  "table": "position",
  "accountId": "<Your account ID>",
  "timestamp": "1607985371481",
  "data": [ {
              "entryPrice": "10000",
              "lastUpdated": "1599693362699",
              "contractValCurrency": "BTC",
              "quantity" : "1.5",
              "instrumentId": "BTC-USD-SWAP-LIN"
            },
            {
              "entryPrice": "205.1",
              "lastUpdated": "1599693362699",
              "contractValCurrency": "ETH",
              "quantity" : "-0.5",
              "instrumentId": "ETH-USD-SWAP-LIN"
            } ]
}
```

**Channel Update Frequency** : on position update

The websocket will reply with the shown success response format for EACH position channel which has been successfully subscribed to.

If a subscription has been made to position:all, the data array in the message from this position channel will contain a JSON list. Each JSON will contain position details for instrument. Otherwise the data array will contain a single JSON corresponding to one instrument per position channel subscription.

**Request Parameters**

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes |  `subscribe`
args | ARRAY | Yes | `position:all` or a list of individual instruments `position:<instrumentId>`
tag | INTEGER | No | If given and non-zero, it will be echoed in the reply

**Channel Update Parameters**

Parameters |Type| Description |
--------|-----|---|
table | STRING| `position`
accountId | STRING|  Account identifier
timestamp|STRING | Current millisecond timestamp
entryPrice | STRING | Average entry price of total position (Cost / Size)
lastUpdated|STRING | Millisecond timestamp
contractValCurrency | STRING | Base asset ID e.g. `ETH`
quantity | STRING| Position size (+/-)
instrumentId | STRING | e.g. `ETH-USD-SWAP-LIN`


### Order Channel

> **Request format**

```json
{
  "op": "subscribe", 
  "args": ["order:all"], 
  "tag": 102
}

OR

{
  "op": "subscribe", 
  "args": ["order:FLEX-USD", "order:ETH-USD-SWAP-LIN", .....], 
  "tag": 102
}
```

> **Success response format**

```json
{
  "success": True, 
  "tag": "102", 
  "event": "subscribe", 
  "channel": "<args value>", 
  "timestamp": "1607985371401"
}
```

**Channel Update Frequency** : real-time

The websocket will reply with the shown success response format for EACH order channel which has been successfully subscribed to.

**Request Parameters**

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes |  `subscribe`
args | ARRAY | Yes | `order:all` or a list of individual markets `order:<marketCode>`
tag | INTEGER | No | If given and non-zero, it will be echoed in the reply


#### OrderOpened

> **OrderOpened message format - LIMIT order**

```json
{
  "table": "order",
  "data": [ {
              "notice": "OrderOpened",
              "accountId": "<Your account ID>",
              "clientOrderId": "16",
              "orderId" : "123",
              "price": "9600",
              "quantity": "2" ,
              "side": "BUY",
              "status": "OPEN",
              "marketCode": "BTC-USD-SWAP-LIN",
              "timeInForce": "MAKER_ONLY",
              "timestamp": "1594943491077"
              "orderType": "LIMIT",
              "isTriggered": "false"
            } ]
}
```

> **OrderOpened message format - STOP order**

```json
{
  "table": "order",
  "data": [ {
              "notice": "OrderOpened",
              "accountId": "<Your account ID>",
              "clientOrderId": "17",
              "orderId": "2245",
              "quantity": "2",
              "side": "BUY",
              "status": "OPEN",
              "marketCode": "USDT-USD-SWAP-LIN",
              "timeInForce": "IOC",
              "timestamp": "1594943491077",
              "stopPrice": "9280",
              "limitPrice": "9300",
              "orderType": "STOP",
              "isTriggered": "true"
            } ]
}
```

Parameters |Type| Description |
--------|-----|---|
notice | STRING| `OrderOpened`
accountId | STRING| Account identifier
clientOrderId |  STRING | Client assigned ID to help manage and identify orders
orderId | STRING | Unique order ID from the exchange
price |STRING | Limit price submitted (only applicable for LIMIT order types)
quantity | STRING| Quantity submitted
side|STRING|`BUY` or `SELL`
status|STRING|  Order status
marketCode | STRING |  Market code e.g. `FLEX-USD`
timeInForce|STRING| Client submitted time in force, `GTC` by default
timestamp|STRING |Current millisecond timestamp
orderType| STRING | `LIMIT` or `STOP`
stopPrice|STRING|Stop price submitted (only applicable for STOP order types)
limitPrice|STRING|Limit price submitted (only applicable for STOP order types)


#### OrderModified

> **OrderModified message format**

```json
{
  "table": "order",
  "data": [ {
              "notice": "OrderModified",
              "accountId": "<Your Account ID>",
              "clientOrderId": "16",
              "orderId": "94335",
              "price": "9600",
              "quantity": "1",
              "side": "BUY",
              "status": "OPEN",
              "marketCode": "BTC-USD-SWAP-LIN",    
              "timeInForce": "GTC",
              "timestamp": "1594943491077"
              "orderType": "LIMIT",
              "isTriggered": "false" 
            } ]
}
```

As described in a previous section [Order Commands - Modify Order](#websocket-api-order-commands-modify-order), the Modify Order command can potentially affect the queue position of the order depending on which parameter of the original order has been modified.

If the orders queue position is **unchanged** because the orders quantity has been **reduced** ano no other order parameter has been changed then this **OrderModified** message will be sent via the Order Channel giving the full details of the modified order.

If however the order queue position has changed becuase the orders: (1) side was modified, (2) quantity was increased, (3) price was changed or any combination of these then the exchange will cancel the orginal order with an **OrderClosed** message and open a new order with an **OrderModified** message with a new orderID to reflect the orders new position in the order queue.

Parameters |Type| Description |
--------|-----|---|
notice | STRING | `OrderModified`
accountId | STRING | Account identifier
clientOrderId |  STRING | Client assigned ID to help manage and identify orders
orderId | STRING | Unique order ID from the exchange
price |STRING | Limit price of modified order (only applicable for LIMIT order types)
quantity | STRING| Quantity of modified order
side|STRING|`BUY` or `SELL`
status|STRING|  Order status
marketCode | STRING |  Market code e.g. `BTC-USD-SWAP-LIN`
timeInForce|STRING| Client submitted time in force, `GTC` by default
timestamp|STRING |Current millisecond timestamp
orderType| STRING | `LIMIT` or `STOP`
stopPrice|STRING|Stop price of modified order (only applicable for STOP order types)
limitPrice|STRING|Limit price of modified order (only applicable for STOP order types)

#### OrderClosed

> **OrderClosed message format - LIMIT orders**

```json
{
  "table": "order",
  "data": [ {
              "notice": "OrderClosed",
              "accountId": "<Your account ID>",
              "clientOrderId": "16",
              "orderId": "73",
              "price": "9600",
              "quantity": "2",
              "side": "BUY",
              "status": "<Canceled status>",
              "marketCode": "BTC-USD-SWAP-LIN",
              "timeInForce": "<Time in force>",
              "timestamp": "1594943491077",
              "remainQuantity": "1.5",
              "orderType": "LIMIT",
              "isTriggered": "false" 
            } ]
}
```

> **OrderClosed message format - STOP orders**

```json
{
  "table": "order",
  "data": [ {
              "notice": "OrderClosed",
              "accountId": "<Your account ID>",
              "clientOrderId": "16",
              "orderId": "13",
              "quantity": "2",
              "side": "BUY",
              "status": "CANCELED_BY_USER",
              "marketCode": "BTC-USD-SWAP-LIN",
              "timeInForce": "<Time in force>",
              "timestamp": "1594943491077",
              "remainQuantity": "1.5",
              "stopPrice": "9100",
              "limitPrice": "9120",
              "orderType": "STOP",
              "isTriggered": "true" 
            } ]
}
```

There are multiple scenarios in which an order is closed as described by the **status** field in the OrderClosed message.  In summary orders can be closed by:-

* `CANCELED_BY_USER` - the client themselves initiating this action or the liquidation engine on the clients behalf if the clients account is below the maintenance margin threshold
* `CANCELED_BY_MAKER_ONLY` - if a time in force MAKER_ONLY order is priced such that it would actually be an agressing taker trade, the order is automatically canceled by to prevent this order from matching as a taker
* `CANCELED_BY_FOK` - since time in force fill-or-kill orders require **all** the of the order quantity to immediately take and match at the specified limit price or better, if no such match is possible then the whole order quantity is canceled
* `CANCELED_ALL_BY_IOC` - 

Parameters | Type | Description
-------------------------- | -----|--------- |
notice | STRING | `OrderClosed`
accountId | STRING  |  Account identifier
clientOrderId|STRING |  Client assigned ID to help manage and identify orders
orderId | STRING  |  Unique order ID from the exchange
price|STRING |Limit price of closed order (only applicable for LIMIT order types)
quantity|STRING |Original order quantity of closed order
side|STRING |`BUY` or `SELL`
status|STRING | <ul><li>`CANCELED_BY_USER`</li><li>`CANCELED_BY_MAKER_ONLY`</li><li>`CANCELED_BY_FOK`</li><li>`CANCELED_ALL_BY_IOC`</li><li>`CANCELED_PARTIAL_BY_IOC`</li></ul>
marketCode|STRING |  Market code e.g. `BTC-USD-SWAP-LIN`
timeInForce|STRING |Time in force of closed order
timestamp|STRING |Current millisecond timestamp
remainQuantity|STRING |Remaining order quantity of closed order
stopPrice|STRING|Stop price of closed stop order (only applicable for STOP order types)
limitPrice|STRING|Limit price of closed stop order (only applicable for STOP order types)
ordertype|STRING  | `LIMIT` or `STOP`

#### OrderRejected

* Reject order with 0 quantity

> **Notification:**

```json

{
    "table": "order",
    "data": [
        {
          "notice": "OrderRejected",
          "quantity": "0",
          "clientOrderID": "13",
          "status": "REJECT_QUANTITY_ZERO",
          "timestamp": "1592496454"
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
    "table": "order",
    "data": [
        {
          "notice": "OrderRejected",
          "status": "REJECT_UNKNOW_ORDER_ACTION",
          "timestamp": "1592495554"
        }
    ]
}

```

* Reject limit order with the market price

> **Notification:**

```json

{
    "table": "order",
    "data": [
        {
          "notice": "OrderRejected",
          "status": "REJECT_LIMITE_ORDER_WITH_MARKET_PRICE",
          "timestamp": "1592493454"
        }
    ]
}

```

* Reject stop-limit order

> **Notification:**

```json

{
    "table": "order",
    "data": [
        {
          "notice": "OrderRejected",
          "clientOrderId": "12",
          "orderType": "STOP",
          "side": "BUY",
          "stopPrice": "9630",
          "limitPrice": "9650",
          "status": "STOP_PRICE_LARGER_THAN_LIMIT_PRICE",
          "timestamp": "1592496774"
        }
    ]
}

```


Parameters | Type | Required
-------------------------- | -----|--------- |
notice | String | `OrderRejected`
accountId | String |  Account identifier
clientOrderId|String|  Client order ID submitted
orderType|string|`STOP`
side|String|`BUY` / `SELL`
status|String|STOP PRICE LARGER THAN LIMIT PRICE
stopPrice|String|Stop price submitted
limitPrice|String|Limit price submitted
timestamp|String|UNIX timestamp

> **Notification:**

```json

{
    "table": "order",
    "data": [
        {
          "notice": "OrderRejected",
          "clientOrderId": "12",
          "orderType": "STOP",
          "side": "SELL",
          "stopPrice": "9800",
          "limitPrice": "9900",
          "status": "STOP_PRICE_LESS_THAN_LIMIT_PRICE",
          "timestamp": "1592496574"
        }
    ]
}

```


Parameters | Type | Required
-------------------------- | -----|--------- |
notice | String | `OrderRejected`
accountId | String |  Account identifier
clientOrderId|String|  Client order ID submitted
orderType|string|`STOP`
side|String|`BUY` / `SELL`
status|String|STOP PRICE LARGER THAN LIMIT PRICE
stopPrice|String|Stop price submitted
limitPrice|String|Limit price submitted
timestamp|String|UNIX timestamp


* Reject cancel an order:

> **Notification:**

```json

{
    "table": "order",
    "data": [
        {
          "notice": "OrderRejected",
          "orderId": "13",
          "status": "REJECT_CANCEL_ORDER_ID_NOT_FOUND",
          "timestamp": "1592494474"
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
    "table": "order",
    "data": [
        {
          "notice": "OrderRejected",
          "orderId": "12",
          "status": "REJECT_AMEND_ORDER_ID_NOT_FOUND",
          "timestamp": "1592498890"
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
    "table": "order",
    "data": [
        {
          "notice": "OrderMatched",
          "accountId": "1",
          "marketCode": "BTC-USD-SWAP-LIN",
          "orderId": "39",
          "clientOrderId": "16",
          "matchId": "1",
          "price": "9300",
          "quantity": "2",
          "matchPrice": "9300",
          "matchQuantity": "2",
          "orderMatchType": "MAKER",
          "remainQuantity": "0",
          "orderType": "LIMIT",
          "side": "BUY",
          "timeInForce": "GTC",
          "fees": "3.7",
          "feeInstrumentId": "FLEX",
          "status": "FILLED",
          "timestamp": "1592490254"
        }
    ]
}


```


Parameters | Type | Required
-------------------------- | -----|--------- |
notice | STRING | `OrderMatched`
accountId | STRING | Account identifier
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
    "table": "order",
    "data": [
        {
          "notice": "OrderMatched",
          "accountId": "11",
          "marketCode": "BTC-USD-SWAP-LIN",
          "orderId": "13",
          "clientOrderId": "16",
          "matchId": "2",
          "price": "9333",
          "quantity": "2",
          "matchPrice": "9333",
          "matchQuantity": "1",
          "orderMatchType": "MAKER",
          "remainQuantity": "1",
          "orderType": "LIMIT",
          "side": "BUY",
          "timeInForce": "GTC",
          "fees": "1.8",
          "feeInstrumentId": "FLEX",
          "status": "PARTIAL_FILL",
          "timestamp": "1592497054"
        }
    ]
}


```



Parameters | Type | Required
-------------------------- | -----|--------- |
notice | STRING | `OrderOpened`
accountId | STRING | Account identifier
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
    "table": "order",
    "data": [
        {
          "notice": "OrderMatched",
          "accountId": "7",
          "marketCode": "BTC-USD-SWAP-LIN",
          "orderId": "12",
          "clientOrderId": "16",
          "matchId": "1",
          "stopPrice": "9700",
          "limitPrice": "9650",
          "quantity": "3",
          "matchPrice": "9650",
          "matchQuantity": "1",
          "orderMatchType": "MAKER",
          "remainQuantity": "2",
          "orderType": "STOP",
          "side": "BUY",
          "timeInForce": "GTC",
          "fees": "1.9",
          "feeInstrumentId": "FLEX",
          "status": "PARTIAL_FILL",
          "timestamp": "15924979954"
        }
    ]
}

```

Parameters | Type | Required
-------------------------- | -----|--------- |
notice | STRING | `OrderOpened`
accountId | STRING | Account identifier
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
    "table": "order",
    "data": [
        {
          "notice": "OrderMatched",
          "accountId": "1",
          "marketCode": "BTC-USD-SWAP-LIN",
          "orderId": "123",
          "clientOrderId": "16",
          "matchId": "1",
          "stopPrice": "9700",
          "limitPrice": "9650",
          "quantity": "2",
          "matchPrice": "9650",
          "matchQuantity": "2",
          "orderMatchType": "MAKER",
          "remainQuantity": "0",
          "orderType": "STOP",
          "side": "BUY",
          "timeInForce": "GTC",
          "fees": "3.8",
          "feeInstrumentId": "FLEX",
          "status": "FILLED",
          "timestamp": "15924979954"
        }
    ]
}

```


Parameters | Type | Required
-------------------------- | -----|--------- |
notice | STRING | `OrderMatched`
accountId | STRING | Account identifier
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
  "args": ["futures/depth:BTC-USD-190628"]
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

{"op": "subscribe", "args": ["trade:BTC-USD-SWAP-LIN"], "tag": 1}

```

> **Success response format**

```json

{"event": "subscribe", "channel": ["trade:BTC-USD-SWAP-LIN"], "success": true, "tag": "1", "timestamp": "1594299886880"}

```

> **Failure response format**

```json

{"event": "subscribe", "message": "<errorMessage>", "code": "<code>", "success": false, "tag": "1", "timestamp": "1594299886880"}

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

{"op": "subscribe", "args": ["ticker:BTC-USD-200925-LIN"], "tag": 1}

```

> **Success response format**

```json

{"event": "subscribe", "channel": ["ticker:BTC-USD-200925-LIN"], "success": true, "tag": "1", "timestamp": "1594299886890"}

```

> **Failure response format**

```json

{"event": "subscribe", "message": "<errorMessage>", "code": "<code>", "success": false, "tag": "1", "timestamp": "1594299886890"}

```

> **Channel update format**

```json

{
    "table": "ticker",
    "data": [
        {
            "marketCode": "BTC-USD-SWAP-LIN",
            "last": "43.259",
            "markPrice": "11012.80409769",
            "open24h": "49.375",
            "volume24h": "11295421",
            "currencyVolume24h": "1225512",
            "high24h": "49.488",
            "low24h": "41.649",
            "openInterest": "1726003",
            "lastQty": "1",
            "timestamp": "123443563454"
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

{"op": "subscribe", "args": ["candles60s:BTC-USD-SWAP-LIN"], "tag": 1}

```

> **Success response format**

```json

{"event": "subscribe", "channel": ["candles60s:BTC-USD-SWAP-LIN"], "success": true, "tag": "1", "timestamp": "1594313762698"}

```

> **Failure response format**

```json

{"event": "subscribe", "message": "<errorMessage>", "code": "<code>", "success": false, "tag": "1", "timestamp": "1594313762698"}

```

> **Channel update format**

```json

{
    "table": "candle60s",
    "data": [
        {
            "candle": [
                "1594313762698", //timestamp
                "9633.1",        //open
                "9693.9",        //high
                "9238.1",        //low
                "9630.2",        //close
                "45247",         //volume
                "5.3"            //currencyVolume
            ],
            "marketCode": "BTC-USD-SWAP-LIN"
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
