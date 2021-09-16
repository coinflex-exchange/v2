# REST API V3

**TEST** site

* `https://v2stgapi.coinflex.com`

**LIVE** site

* `https://v2api.coinflex.com/v3`

For clients who do not wish to take advantage of CoinFLEX's native WebSocket API, CoinFLEX offers a RESTful API that implements much of the same functionality.

##Authentication 

> **Request**

```json
{
  "Content-Type": "application/json",
  "AccessKey": "<string>",
  "Timestamp": "<string>", 
  "Signature": "<string>", 
  "Nonce": "<string>"
}
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
from urllib.parse import urlencode

rest_url = 'https://v2stgapi.coinflex.com'
rest_path = 'v2stgapi.coinflex.com'

api_key = <API-KEY>
api_secret = <API-SECRET>

ts = datetime.datetime.utcnow().isoformat()
nonce = 123
method = <API-METHOD>

# Optional and can be omitted depending on the REST method being called 
body = urlencode({'key1': 'value1', 'key2': 'value2'})

if body:
    path = method + '?' + body
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, body)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
print(resp.json())
```

Public market data methods do not require authentication, however private methods require a *Signature* to be sent in the header of the request.  These private REST methods  use HMAC SHA256 signatures. 

The HMAC SHA256 signature is a keyed HMAC SHA256 operation using a clients API Secret as the key and a message string as the value for the HMAC operation.  

The message string is constructed by the following formula:-

`msgString = Timestamp + "\n" + Nonce + "\n" + Verb + "\n" + URL + "\n" + Path +"\n" + Body`

Component | Required | Example | Description| 
-------------------------- |--------- |------- |------- | 
Timestamp | Yes | 2020-04-30T15:20:30 | YYYY-MM-DDThh:mm:ss
Nonce | Yes | 123 | User generated
Verb | Yes| 'GET' | Uppercase
Path | Yes | 'v2stgapi.coinflex.com' |
Method | Yes | '/v2/positions | Available REST methods: <li>`V2/positions`</li><li>`V2/orders`</li><li>`V2/balances`</li>
Body | No | instrumentID=BTC-USD-SWAP-LIN | Optional and dependent on the REST method being called

The constructed message string should look like:-

  `2020-04-30T15:20:30\n
  123\n
  GET\n
  v2stgapi.coinflex.com\n
  /v2/positions\n
  instrumentID=BTC-USD-SWAP-LIN`

Note the newline characters after each component in the message string. 
If *Body* is ommitted it's treated as an empty string.

Finally you must use the HMAC SHA256 operation to get the hash value using the API Secret as the key and the constructed message string as the value for the HMAC operation. Then encode this hash value with BASE-64.  This output becomes the signature for the specified authenticated REST API method. 

The signature must then be included in the header of the REST API call like so:

`header = {'Content-Type': 'application/json', 'AccessKey': API-KEY, 'Timestamp': TIME-STAMP, 'Signature': SIGNATURE, 'Nonce': NONCE}`

##Order Commands - Private

### POST /v3/orders/place

> **Request**

```json
POST /v2/orders/place

{
“recvWindow”: 1234           
“timestamp”: 235536            
“responseType”: “FULL” or “ACK”    
“orders”: [
{        
“clientOrderId”: “345srete”       
“marketCode”:     “BTC-USD”       
“side”: “BUY” or “SELL”        
“quantity”: “0.002”           
“timeInForce”:    “IOC”           
“orderType”: LIMIT/MARKET/STOP  
“price”: “1.345”            
“stopPrice”: “2.567”            
“limitPrice”: “4.567”           
        },
        { ………. } 
]
}


```

> **Success response format**

```json
{
    "event": "placeOrders",
    "success": true,
    "data": [
        {
            "code": "710002",
            "message": "FAILED sanity bound check as price (50007.0) <  lower bound (54767.0)",
    "submitted": false,
            "clientOrderId": "1612249737724",
            "marketCode": "BTC-USD-SWAP-LIN",
“side': 'SELL',
    "price": "50007.0",
            "quantity": "0.001",
            "orderType": "LIMIT",
            "timeInForce": "GTC", 
            "createdAt": "1615430915596",
        "source": 0
        },
        {
        'notice': 'OrderOpened',           // Can also be OrderMatched →use OM structure
        'accountId': '1076', 
        'orderId': '1000028795158', 
        ‘submitted’: true, 
        'clientOrderId': '1', 
        'marketCode': 'BTC-USD-SWAP-LIN', 
        'status': 'OPEN', 
        'side': 'BUY', 
        'price': '9600.0', 
        ‘stopPrice’: null,
        'isTriggered': null,
        'quantity': '0.01', 
                ‘remainQuantity’: ‘0.01’,
    'matchId': null,
    'matchPrice': null, 
    'matchQuantity': null, 
                ‘feeInstrumentId’: null,
        ‘fees’: null,
        'orderType': 'LIMIT', 
        'timeInForce': 'MAKER_ONLY', 
        'createdAt': '1629192975532',        // change timestamp to createdAt
                ‘modifiedAt’: null,            // use this when modifying
        ‘matchedAt’: null,            // populate matchedAt when matching
        },
    ]
}
```

> **Failure response format**

```json
{
“event”: “placeOrders”,
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
}
```

Place orders.

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
recvWindow | LONG | NO | |
timestamp | STRING | YES | |
responseType | STRING | YES | `FULL` or `ACK`  if FULL return data array 
orders | LIST | YES | |
clientOrderId | STRING | NO| |
marketCode | STRING | YES | |
side | STRING | YES | |
quantity | STRING or FLOAT | YES | |
timeInForce | STRING | NO | Default `GTC` |
orderType | STRING | YES | Default 'limit' |
price | STRING or FLOAT | NO |Do not include for MARKET |
stopPrice | STRING | YES | |
limitPrice | STRING | YES | |

Response Parameters | Type | Description | 
--------------------| ---- | ----------- |
accountId | STRING | |
event | STRING | |
timestamp | STRING | |
data | LIST | |
success | STRING | Whether an order has been successfully placed |
timestamp | STRING | |
code | STRING | Error code |
message | STRING | Error message |
clientOrderId | STRING | |
orderId | STRING | |
price | STRING | |
quantity | STRING | |
side | STRING | `SELL` or `BUY` |
marketCode | STRING | |
timeInForce | STRING | |
orderType | STRING | |


### POST /v3/orders/modify
> **Request**

```json
POST /v2/orders/modify

{
“recvWindow”: 1234            
“timestamp”: 235536           
“responseType”: “FULL” or “ACK”   
    "orders": [
        {
            "clientOrderId": "1614330009059",       
            "orderId": "304369975621712260",       
            "marketCode": "BTC-USD-SWAP-LIN",
            "side": "BUY",                  
            "quantity": "0.007",                
            "price": "40001.0"               
    “stopPrice: “930.01”,             
    “limitPrice: “903.12”               
        },
        {
            "clientOrderId": "161224973777800",
"orderId": "203369975621712261",
            "marketCode": "BTC-USD-SWAP-LIN",
            "side": "SELL",
            "quantity": "0.002",
            "price": "40003.0”
        },
        {
            "clientOrderId": "161224973777900",
    "orderId": "588369975621712255",
            "marketCode": "BTC-USD-SWAP-LIN",
            "side": "SELL",
            "quantity": "0.003",
            "price": "40004.0"
        }
    ]
}

```

> **Success response format**

```json
{
    "event": "modifyOrders",
    “success”: true,
    "data": [
        {
            "code": "710002",
            "message": "FAILED sanity bound check as price (50007.0) <  lower bound (54767.0)",
            “orderId”: “123253532”,
            "clientOrderId": "1612249737724",
            "marketCode": "BTC-USD-SWAP-LIN",
 “side': 'SELL',
     "price": "50007.0",
            "quantity": "0.001",
            "orderType": "LIMIT",
            "timeInForce": "GTC", 
            "modifiedAt": "1615430915596",
        "source": 0
        },
        {
           'notice': 'OrderModified',          
        'accountId': '1076', 
        'orderId': '1000028795158', 
        ‘submitted’: true,
        'clientOrderId': '1', 
        'marketCode': 'BTC-USD-SWAP-LIN', 
        'status': 'OPEN', 
        'side': 'BUY', 
        'price': '9600.0', 
        ‘stopPrice’: null,
        'isTriggered': null,
        'quantity': '0.01', 
                ‘remainQuantity’: ‘0.01’,
    'matchId': null,
    'matchPrice': null, 
    'matchQuantity': null, 
                ‘feeInstrumentId’: null,
        ‘fees’: null,
        'orderType': 'LIMIT', 
        'timeInForce': 'MAKER_ONLY', 
        'createdAt': null,       
                ‘modifiedAt’: “162321312311”,            
        ‘matchedAt’: null,            
        }

    ]
}

```

> **Failure response format**

```json
{
“event”: “modifyOrders”,
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
}

```

Modify orders.

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
recvWindow | LONG | NO | |
timestamp | STRING | NO | |
responseType | STRING | YES | `FULL` or `ACK` |
orders | LIST | YES | |
clientOrderId | STRING | NO | |
orderId | STRING | YES | |
marketCode | STRING | YES | |
side | STRING | NO | |
quantity | STRING | NO | |
price | STRING | NO | |
stopPrice | STRING | NO | |
limitPrice | STRING | NO | |

Response Parameters | Type | Description | 
--------------------| ---- | ----------- |
accountId | STRING | |
event | STRING | |
timestamp | STRING | |
data | LIST | |
success | STRING | |
timestamp | STRING | |
code | STRING | Error code |
message | STRING | Error message |
clientOrderId | STRING | |
orderId | STRING | |
price | STRING | |
quantity | STRING | |
side | STRING | `SELL` or `BUY` |
status | STRING | Status of the order |
marketCode | STRING | |
timeInForce | STRING | |
notice | STRING | `OrderClosed` or `OrderMatched` or `OrderOpend` |
orderType | STRING | |
isTriggered | STRING | `true` or `false` |


### DELETE /v3/orders/cancel

> **Request**

```json

{
    "recvWindow": 500,        
“timestamp”: 235536,         
    "responseType": “FULL” or “ACK”   
"orders": [
        {
            "marketCode": "BTC-USD-SWAP-LIN",  
            "orderId": "304384250571714215",    
            "clientOrderId": "1615453494726”    
        },
        {
            "marketCode": "BTC-USD-SWAP-LIN",
"orderId": "204285250571714316",
            "clientOrderId": "1612249737724”
        }
    ]
}

```

> **Success response format**

```json
{
    "event": "cancelOrders",
    “success”: true,
    "data": [
{
'notice': 'OrderClosed', 
          'accountId': '12005486', 
'orderId': '1000018305713',
‘submitted’: true,
'clientOrderId': '1', 
'marketCode': 'BTC-USD-SWAP-LIN', 
'status': 'CANCELED_BY_USER', 
'side': 'BUY', 
'price': '4870.0', 
‘stopPrice’: null,
'isTriggered': null
'quantity': '0.001', 
'remainQuantity': '0.001',
    'orderType': 'LIMIT',  
'timeInForce': 'GTC', 
'closedAt': '1629712561919', 
        },
        {
            "code": "40035",
            "message": "Open order not found with id",
    "submitted": false,
"orderId": "204285250571714316",
            "clientOrderId": "1612249737724",
            "marketCode": "BTC-USD-SWAP-LIN",
    "closedAt": "1615454881433",
        }
    ]
}

```

> **Failure response format**

```json
{
“event”: “cancelOrders”,
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
}
```


Cancel orders.

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
recvWindow | LONG | NO | |
timestamp | LONG | YES | |
responseType | STRING | YES | `FULL` or `ACK` If FULL return data array|
orders | LIST | YES | |
marketCode | STRING | YES | |
orderId | STRING or LONG | NO | |
clientOrderId | STRING | NO | |

Response Parameters | Type | Description | 
--------------------| ---- | ----------- |
accountId | STRING | |
event | STRING | |
timestamp | STRING | |
data | LIST | |
success | STRING | |
timestamp | STRING | |
code | STRING | Error code |
message | STRING | Error message |
clientOrderId | STRING | |
orderId | STRING | |
price | STRING | |
quantity | STRING | |
side | STRING | `SELL` or `BUY` |
status | STRING | Status of the order |
marketCode | STRING | |
timeInForce | STRING | |
notice | STRING | `OrderClosed` or `OrderMatched` or `OrderOpend` |
orderType | STRING | |
isTriggered | STRING | `true` or `false` |


### DELETE /v3/orders/cancel-all    

> **Request**

```json

{
    "recvWindow": 500,        
“timestamp”: 235536,         
    "responseType": “FULL” or “ACK”   
"orders": [
        {
            "marketCode": "BTC-USD-SWAP-LIN",  
        },
        {
            "marketCode": "BTC-USD-SWAP-LIN",
"orderId": "204285250571714316",
            "clientOrderId": "1612249737724”
        }
    ]
}

```

> **Success response format**

```json
{
    "event": "cancelOrders",
    “success”: true,
    "data": [
{
'notice': 'OrderClosed', 
          'accountId': '12005486', 
'orderId': '1000018305713',
‘submitted’: true,
'clientOrderId': '1', 
'marketCode': 'BTC-USD-SWAP-LIN', 
'status': 'CANCELED_BY_USER', 
'side': 'BUY', 
'price': '4870.0', 
‘stopPrice’: null,
'isTriggered': null
'quantity': '0.001', 
'remainQuantity': '0.001',
    'orderType': 'LIMIT',  
'timeInForce': 'GTC', 
'closedAt': '1629712561919', 
        },
        {
            "code": "40035",
            "message": "Open order not found with id",
    "submitted": false,
"orderId": "204285250571714316",
            "clientOrderId": "1612249737724",
            "marketCode": "BTC-USD-SWAP-LIN",
    "closedAt": "1615454881433",
        }
    ]
}

```

> **Failure response format**

```json
{
“event”: “cancelOrders”,
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
}
```


Cancel orders.

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
recvWindow | LONG | NO | |
timestamp | LONG | YES | |
responseType | STRING | YES | `FULL` or `ACK` If FULL return data array|
orders | LIST | YES | |
marketCode | STRING | YES | |
orderId | STRING or LONG | NO | |
clientOrderId | STRING | NO | |

Response Parameters | Type | Description | 
--------------------| ---- | ----------- |
accountId | STRING | |
event | STRING | |
timestamp | STRING | |
data | LIST | |
success | STRING | |
timestamp | STRING | |
code | STRING | Error code |
message | STRING | Error message |
clientOrderId | STRING | |
orderId | STRING | |
price | STRING | |
quantity | STRING | |
side | STRING | `SELL` or `BUY` |
status | STRING | Status of the order |
marketCode | STRING | |
timeInForce | STRING | |
notice | STRING | `OrderClosed` or `OrderMatched` or `OrderOpend` |
orderType | STRING | |
isTriggered | STRING | `true` or `false` |


##Methods - Private

All private REST API methods require authentication using the approach explained above. 

### GET Request

#### GET `/v2/accountinfo`

> **Request**

```json
GET /v2/accountinfo
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
from urllib.parse import urlencode

rest_url = 'https://v2stgapi.coinflex.com'
rest_path = 'v2stgapi.coinflex.com'

api_key = <API-KEY>
api_secret = <API-SECRET>

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2/accountinfo'

# Not required for /v2/accountinfo
body = urlencode({})

if body:
    path = method + '?' + body
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, body)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
print(resp.json())
```

> **Response**

```json
{
  "event": "accountinfo",
  "timestamp": 1611665626191
  "accountId": "<Your Account ID>",
  "data": [ {
              "accountId": "<Your Account ID>",
              "tradeType": "LINEAR",
              "marginCurrency": "USD",
              "totalBalance": "10000",
              "availableBalance": "10000",
              "collateralBalance": "10000",
              "portfolioVarMargin": "500",
              "riskRatio": "20.0000",
              "timestamp": "1611665624601"
          } ]
}
```

```python
{
  "event": "accountinfo",
  "timestamp": 1611665626191
  "accountId": "<Your Account ID>",
  "data": [ {
              "accountId": "<Your Account ID>",
              "tradeType": "LINEAR",
              "marginCurrency": "USD",
              "totalBalance": "10000",
              "availableBalance": "10000",
              "collateralBalance": "10000",
              "portfolioVarMargin": "500",
              "riskRatio": "20.0000",
              "timestamp": "1611665624601"
          } ]
}
```

Returns the account level information connected to the API key initiating the request. 

<sub>**Response Parameters**</sub> 

Parameters |Type | Description 
-------------------------- | -----|--------- |
event | STRING | `accountinfo`
timestamp | INTEGER | Millisecond timestamp
accountId | STRING    | Account ID
data | LIST of dictionary |
\>accountId | STRING    | Account ID
\>tradeType | STRING    | Account type `LINEAR`
\>marginCurrency | STRING | Asset `USD`
\>totalBalance | STRING | Total balance denoted in marginCurrency
\>collateralBalance | STRING | Collateral balance with LTV applied
\>availableBalance | STRING | Available balance
\>portfolioVarMargin| STRING | Portfolio margin
\>riskRatio | STRING | collateralBalance / portfolioVarMargin, Orders are rejected/cancelled if the risk ratio drops below 1 and liquidation occurs if the risk ratio drops below 0.5
\>timestamp | STRING | Millisecond timestamp

#### GET `/v2/balances`

> **Request**

```json
GET /v2/balances
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
from urllib.parse import urlencode

rest_url = 'https://v2stgapi.coinflex.com'
rest_path = 'v2stgapi.coinflex.com'

api_key = <API-KEY>
api_secret = <API-SECRET>

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2/balances'

# Not required for /v2/balances
body = urlencode({})

if body:
    path = method + '?' + body
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, body)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
print(resp.json())
```

> **Response**

```json
{
  "event": "balances",
  "timestamp": 1611665626191,
  "accountId": "<Your Account ID>",
  "tradeType": "LINEAR",
  "data": [ {   
              "instrumentId": "BTC",
              "total": "4468.823",              
              "available": "4468.823",        
              "reserved": "0",
              "quantityLastUpdated": "1593627415234"
            },
            ...
            {
              "instrumentId": "FLEX",
              "total": "1585.890",              
              "available": "325.890",         
              "reserved": "1260",
              "quantityLastUpdated": "1593627415123"
            }
          ]
}
```

```python
{
  "event": "balances",
  "timestamp": 1611665626191,
  "accountId": "<Your Account ID>",
  "tradeType": "LINEAR",
  "data": [ {   
              "instrumentId": "BTC",
              "total": "4468.823",              
              "available": "4468.823",        
              "reserved": "0",
              "quantityLastUpdated": "1593627415234"
            },
            ...
            {
              "instrumentId": "FLEX",
              "total": "1585.890",              
              "available": "325.890",         
              "reserved": "1260",
              "quantityLastUpdated": "1593627415123"
            }
          ]
}
```

Returns all the coin balances of the account connected to the API key initiating the request. 

<sub>**Response Parameters**</sub> 

Parameters |Type | Description| 
-------------------------- | -----|--------- |
event | STRING | `balances`
timestamp | INTEGER | Millisecond timestamp
accountId | STRING    | Account ID
tradeType | STRING    | `LINEAR` |
data | LIST of dictionaries |
instrumentId | STRING |Coin symbol, e.g. 'BTC' |
total| STRING| Total balance|
available |STRING| Available balance|
reserved|STRING|Reserved balance (unavailable) due to working spot orders|
quantityLastUpdated|STRING|Millisecond timestamp of when balance was last updated|


### POST Request

#### POST `/v2.1/delivery/orders`
> **Request**

```json
POST /v2.1/delivery/orders?instrumentId={instrumentId}&qtyDeliver={qtyDeliver}

{
  "instrumentId": {instrumentId},
  "qtyDeliver": {qtyDeliver}
}
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
from urllib.parse import urlencode

rest_url = 'https://v2stgapi.coinflex.com'
rest_path = 'v2stgapi.coinflex.com'

api_key = <API-KEY>
api_secret = <API-SECRET>

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2.1/delivery/orders' 

# Required for POST /v2.1/delivery/orders
body = urlencode({"instrumentId": "BTC-USD-SWAP-LIN", "qtyDeliver": "1"})

if body:
    path = method + '?' + body
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'POST', rest_path, method, body)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.post(rest_url + path, headers=header)
print(resp.json())
```

> **Response**

```json
{
  "event": "delivery",
  "timestamp": "1599204123484",
  "accountId": "164",
  "data": [ {
              "deliverOrderId": "586985384617312258",
              "accountId": "164",
              "clientOrderId": null,
              "instrumentId": "BTC-USD-SWAP-LIN",
              "deliverPrice": "10000.000000000",
              "deliverPosition": "1.000",
              "deliverType": "NEXT_CYCLE",
              "instrumentIdDeliver": "BTC",
              "deliverQty": "1.000",
              "remainingQty": "1.000",
              "remainingPosition": "1.000",
              "transferAsset": "USD",
              "transferQty": "1",
              "auctionTime": "1599204122693",
              "created": "1599204122693",
              "lastUpdated": "1599204122693",
              "status": "PENDING"
          } ]
}
```

```python
{
  "event": "delivery",
  "timestamp": "1599204123484",
  "accountId": "164",
  "data": [ {
              "deliverOrderId": "586985384617312258",
              "accountId": "164",
              "clientOrderId": null,
              "instrumentId": "BTC-USD-SWAP-LIN",
              "deliverPrice": "10000.000000000",
              "deliverPosition": "1.000",
              "deliverType": "NEXT_CYCLE",
              "instrumentIdDeliver": "BTC",
              "deliverQty": "1.000",
              "remainingQty": "1.000",
              "remainingPosition": "1.000",
              "transferAsset": "USD",
              "transferQty": "1",
              "auctionTime": "1599204122693",
              "created": "1599204122693",
              "lastUpdated": "1599204122693",
              "status": "PENDING"
          } ]
}
```

Submits a request for physical delivery for a specified perpetual swap instrument and quantity.

<sub>**Request Parameters**</sub> 

Parameters | Type | Required |Description| 
-------------------------- | -----|--------- | -------------|
instrumentID| STRING | YES | Perpetual swap instrument intended for delivery | 
qtyDeliver| STRING | YES | Quantity intended for delivery | 

<sub>**Response Parameters**</sub> 

Parameters |Type | Description| 
-------------------------- | -----|--------- |
event | STRING | `delivery`
timestamp | STRING | Millisecond timestamp of the repsonse
accountId | STRING    | Account ID|
data | LIST of dictionary |
deliverOrderId | STRING | Order id
accountId | STRING    | Account ID|
clientOrderId | Null Type|  null
instrumentId | STRING | Perpetual swap market code
deliverPrice | STRING|  Mark price at delivery
deliverPosition | STRING | Delivered position size
deliverType | STRING| ‘NEXT_CYCLE’: Queueing for the upcoming auction
\>instrumentIdDeliver | STRING |Asset being received: long position = coin, short position = USD
deliverQty | STRING |  Quantity of the received asset
remainingQty | STRING | Remaining quantity
remainingPosition | STRING | Remaining position
transferAsset | STRING | Asset being sent
transferQty | STRING | Quantity being sent
auctionTime | STRING | Millisecond timestamp of the next auction
created | STRING | Millisecond timestamp
astUpdated | STRING | Millisecond timestamp 
status | STRING | Delivery status


### DELETE Request

#### DELETE `/v2.1/delivery/orders/{deliveryOrderId}`
> **Request**

```json
DELETE /v2.1/delivery/orders/{deliveryOrderId}
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
from urllib.parse import urlencode

rest_url = 'https://v2stgapi.coinflex.com'
rest_path = 'v2stgapi.coinflex.com'

api_key = <API-KEY>
api_secret = <API-SECRET>

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v2.1/delivery/orders/{deliveryOrderId}' 

# Not required for DELETE /v2.1/delivery/orders/{deliveryOrderId}
body = urlencode({})

if body:
    path = method + '?' + body
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'DELETE', rest_path, method, body)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.delete(rest_url + path, headers=header)
print(resp.json())
```

> **Success response**

```json
{
  "event": "delivery",
  "timestamp": "1599204310297",
  "accountId": "164",
  "data": [
            "Cancel of user order was successful"
          ]
}
```

```python
{
  "event": "delivery",
  "timestamp": "1599204310297",
  "accountId": "164",
  "data": [
            "Cancel of user order was successful"
          ]
}
```

> **Failure response**

```json
{
  "event": "delivery",
  "timestamp": "1599204310297",
  "accountId": "164",
  "data": [
            "Cancel exception please try again"
          ]
}
```

```python
{
  "event": "delivery",
  "timestamp": "1599204310297",
  "accountId": "164",
  "data": [
            "Cancel exception please try again"
          ]
}
```

## Methods - Public

### GET Request

#### GET `/v2/all/markets`

> **Request**

```json
GET/v2/all/markets`
```

> **RESPONSE**


```json
{
    "event": "markets",
    "timestamp": "1620810144786",
    "data": [
        {
            "marketId": "2001000000000",
            "marketCode": "BTC-USD",
            "name": "BTC/USD",
            "referencePair": "BTC/USD",
            "base": "BTC",
            "counter": "USD",
            "type": "SPOT",
            "tickSize": "0.1",
            "qtyIncrement": "0.001",
            "listingDate": 2208988800000,
            "endDate": 0,
            "marginCurrency": "USD",
            "contractValCurrency": "BTC",
            "upperPriceBound": "11000.00",
            "lowerPriceBound": "9000.00",
            "marketPrice": "10000.00"
            "marketPriceLastUpdated": "1620810131131"
        },
        ...
    ]
}
```
Get a list of all available markets on CoinFlex.

Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
timestamp | STRING    | Timestamp of this response|
marketId | STRING | |
marketCode| STRING    | Market Code                  |
name      | STRING    | Name of the contract                  |
referencePair| STRING | Reference pair                  |
base      | STRING    | Base asset                  |
counter   | STRING    | Counter asset                  |
type      | STRING    | Type of the contract                  |
tickSize  | STRING    | Tick size of the contract                  |
qtyIncrement| STRING  |Minimum increamet quantity|
listingDate| STRING   | Listing date of the contract                  |
endDate    |STRING    | Ending date of the contract                  |
marginCurrency|STRING | Margining currency                  |
contractValCurrency| STRING| Contract valuation currency|
upperPriceBound| STRING| Upper price bound                 |
lowerPriceBound| STRING| Lower price bound                 |
marketPrice    | STRING| Market price                 |
marketPriceLastUpdated | LONG | The time that market price last updated at |


#### GET `/v2/all/assets`

> **Request**

```json
GET/v2/all/assets
```

> **RESPONSE**

```json
{
    "event": "assets",
    "timestamp":"1593617008698",
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

Get a list of all assets available on CoinFLEX. These include coins and bookable contracts.

Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
timestamp | STRING    | Timestamp of this response|
instrumentId| STRING    | Instrument ID                   |
name      | STRING    | Name of the asset                  |
base      | STRING    | Base of the asset                  |
counter   | STRING    | Counter of the asset                  |
type      | STRING    | type of the asset                  |
marginCurrency| STRING | Margining currency                 |
contractValCurrency| STRING| Contract valuation currency              |
deliveryDate       | STRING| Delivery date             |
deliveryInstrument | STRING| Delivery instrument             |
