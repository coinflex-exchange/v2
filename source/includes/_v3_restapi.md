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


##Account Wallet - Private

#### GET /v3/accountinfo


> **Request**

```json
GET /v3/account?subAcc=1,2,3,4

```

```python
import requests
import hmac
import base64
import hashlib
import datetime
from urllib.parse import urlencode

rest_url = 'https://v3stgapi.coinflex.com'
rest_path = 'v3stgapi.coinflex.com'

api_key = <API-KEY>
api_secret = <API-SECRET>

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v3/accountinfo'

# Not required for /v3/accountinfo
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

> **Success response format**

```json
{
    “success”: true,
    "data”:  [
        {
            "accountId": "21213",
            “accountName”: “main”,
            "accountType": "LINEAR",
            "marginCurrency": "USD",
            “balances”:  [ 
                {
                    "asset": "BTC",
                    "total": "4468.823", 
                    "available": "4468.823",        
                    "reserved": "0",
                    "lastUpdatedAt": "1593627415234"
                },
                {
                    "asset": "FLEX",
                    "total": "1585.890",              
                    "available": "325.890",         
                    "reserved": "1260",
                    "lastUpdatedAt": "1593627415123"
                },
                ...
            ],
            “positions”:  [ 
                {
                    "marketCode": "BTC-USD-SWAP-LIN",
                    “baseAsset”: “BTC”,
                    “counterAsset”: “USD”,
                    "position": "0.94",
                    "entryPrice": "7800.00", 
                    “markPrice”: “33000.00”, 
                    "positionPnl": "200.3",
                    “estLiquidationPrice”: “12000.05”,
                    “lastUpdatedAt": "1592486212218"              
                },
                ...
            ],
            “collateral”: “1231231”,
            “notionalPositionSize”: “5000.00”, //sum(position_qty * markPrice) position表里的quantity 乘以 markPrice，然后累加
            "portfolioVarMargin": "500",
            "riskRatio": "20.0000",                
            ‘maintenanceMargin’: ‘1231’,            // maintenance margin, maintenanceMargin = portfolioVarMargin / 2
            "marginRatio": "12.3179",             // like the GUI
            “liquidating”: false,         // flag to check if the account is being liquidated
            ‘feeTier’: ‘6’,                // account fee tier (VIP level)
            "createdAt": "1611665624601"
        },
        ...
    ]
}
```

> **Failure response format**

{
“success”: false,
“code”: “40001”,
“message”: “Account not found”
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

#### GET /v3/balances

> **Request**

```json
GET /v3/balances
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
from urllib.parse import urlencode

rest_url = 'https://v3stgapi.coinflex.com'
rest_path = 'v3stgapi.coinflex.com'

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

> **Success response format**

```json
{
    “success”: true,
    "data": [
        {
            "accountId": "21213",
            "balances": [
                {
                    "asset": "BTC",
                    "total": "4468.823",              
                    "available": "4468.823",        
                    "reserved": "0",
                    "lastUpdatedAt": "1593627415234”
                },
                {
                    "asset": "FLEX",
                    "total": "1585.890",              
                    "available": "325.890",         
                    "reserved": "1260",
                    "lastUpdatedAt": "1593627415123"
                },
                ...
            ]
        },
        ...
    ]
}
```
> **Failure response format**

{
“success”: false,
“code”: “40001”,
“message”: “Account not found”
}

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

##Methods - Private

All private REST API methods require authentication using the approach explained above. 


#### GET /v3/positions

> **Request**

```json
GET v3/positions?subAcc=1,2,3,4 …….&marketCode={marketCode}
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
method = '/v2/positions'

# Not required for /v2/positions
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


> **Success response format**

```json
{
    ”success”: true,
    "data":  [
        {
            "accountId": "21213",
            "positions": [
                {
                    "marketCode": "BTC-USD-SWAP-LIN",
                    “baseAsset”: “BTC”,
                    “counterAsset”: “USD”,
                    "position": "0.94",
                    "entryPrice": "7800.00", 
                    “markPrice”: “33000.00”, 
                    "positionPnl": "200.3",
                    “estLiquidationPrice”: “12000.05”,
                    “lastUpdatedAt": "1592486212218"
                },
                ...
            ]
        },
        ...
    ]
}
```

> **Failure response format**

{
“success”: false,
“code”: “40002”,
“message”: “Invalid authentication”
}
```

```python
{
  "event": "positions",
  "timestamp": 1593627415000,
  "accountId":"<Your Account ID>",
  "data": [ {
              "instrumentId": "BTC-USD-SWAP-LIN",
              "quantity": "0.542000000",
              "lastUpdated": "1617099855966",
              "contractValCurrency": "BTC",
              "entryPrice": "56934.8258",
              "positionPnl": "1212.0065",
              "estLiquidationPrice": "52454.3592"
            },
            ...
          ]
}
```

Returns all the positions of the account connected to the API key initiating the request. 

Response Fields | Type | Description |
--------------- | ---- | ----------- |
event | STRING | `positions` |
timestamp | INTEGER | Millisecond timestamp |
accountId | STRING | Account ID |
data | LIST of dictionaries | |
instrumentId | STRING | Contract symbol, e.g. 'BTC-USD-SWAP-LIN' |
quantity | STRING | Quantity of position, e.g. '0.94' |
lastUpdated | STRING| Timestamp when position was last updated |
contractValCurrency | STRING | Contract valuation currency |
entryPrice | STRING | Average entry price |
positionPnl | STRING | Postion profit and lost |
estLiquidationPrice | STRING | Estimated liquidation price, return 0 if it is negative(<0) |

#### GET /v3/trades

> **Request**

```json
GET /v3/trades?marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}

{
  "limit": {limit},
  "startTime": {startTime},
  "endTime": {endTime}
}
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
from urllib.parse import urlencode

rest_url = 'https://v3stgapi.coinflex.com'
rest_path = 'v3stgapi.coinflex.com'

api_key = <API-KEY>
api_secret = <API-SECRET>

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method, for example FLEX-USD spot trades
method = '/v3/trades/FLEX-USD'

# Optional for /v3/trades/{marketCode}
body = urlencode({'limit': 10, 'startTime': 1611850042397, 'endTime': 1611850059021})

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

> **Success response format**

```json
{
"success": true,
“data": [ {        
            "orderId": "160067484555913076",
              "clientOrderId": "123",
              "matchId": "160067484555913077",
              "marketCode": "FLEX-USD",
        "side": "SELL",
              "matchedQuantity": "0.1",
              "matchPrice": "0.065",
              "total": "0.0065",    
            “leg1Price”: null,         // REPO & SPR
            “leg2Price”: null,        // REPO & SPR
              "orderMatchType": "TAKER",
    “feeAsset”: "FLEX”,
           "fee”: "0.0096",
            "lastMatchedAt": "1595514663626"
            },
            …
]
}

```

> **Failure response format**

{
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
}
```

```python
{
  "event": "positions",
  "timestamp": 1593627415000,
  "accountId":"<Your Account ID>",
  "data": [ {
              "instrumentId": "BTC-USD-SWAP-LIN",
              "quantity": "0.542000000",
              "lastUpdated": "1617099855966",
              "contractValCurrency": "BTC",
              "entryPrice": "56934.8258",
              "positionPnl": "1212.0065",
              "estLiquidationPrice": "52454.3592"
            },
            ...
          ]
}
```

Returns all the positions of the account connected to the API key initiating the request. 

Response Fields | Type | Description |
--------------- | ---- | ----------- |
event | STRING | `positions` |
timestamp | INTEGER | Millisecond timestamp |
accountId | STRING | Account ID |
data | LIST of dictionaries | |
instrumentId | STRING | Contract symbol, e.g. 'BTC-USD-SWAP-LIN' |
quantity | STRING | Quantity of position, e.g. '0.94' |
lastUpdated | STRING| Timestamp when position was last updated |
contractValCurrency | STRING | Contract valuation currency |
entryPrice | STRING | Average entry price |
positionPnl | STRING | Postion profit and lost |
estLiquidationPrice | STRING | Estimated liquidation price, return 0 if it is negative(<0) |

###  GET /v3/orders/history

> **Request**

```json
GET /v3/orders/history?marketCode={marketCode}&orderId={orderId}&clientOrderId={clientOrderId}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Sucessful Response**

```json
{
    "event": "orderHistory",
    "success": true,
    "data": [
        {
          "orderId": "304408197314577142",          // each orderId has only 1 response
"clientOrderId": "1",                // clientOrderId can have many resp
"marketCode": "BTC-USD-SWAP-LIN",
“status": CLOSED | OPEN | PARTIALLY_FILLED | FILLED
"side": "BUY",
"price": "10006.0",
“stopPrice”: null,
“isTriggered”: null,
"quantity": "0.001",
"remainQuantity": "0.001",
"matchedQuantity": "0.000",
“avgFillPrice”: null,
“fees”: {‘USD’: "0", ‘FLEX’: “0”}
"orderType": "LIMIT",
"timeInForce": "GTC",
"createdAt": "1619121040779"
            'lastModifiedAt': null,
“lastMatchedAt”: null,
"closedAt": "1619131050779",
        },
        ...
    ]
}
```

> **Sucessful Response**
{
“event”: “orders”,
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
}
```

Returns all orders of the account connected to the API key initiating the request.

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
marketCode | STRING | NO | |
orderId | LONG | NO | |
clientOrderId | LONG | NO | |
limit | LONG | NO | max `100`, default `100` |
startTime | LONG | NO | e.g. `1579450778000`, default `0` |
endTime | LONG | NO | e.g. `1613978625000`, default time now |


Response Fields | Type | Description |
--------------- | ---- | ----------- |
accountId | STRING | Account ID |
timestamp | STRING | Timestamp of this response |
status | STRING | Status of the order |
orderId | STRING | Order ID which generated by the server |
clientOrderId | STRING | Client order ID which given by the user |
marketCode | STRING | Market code |
side | STRING | Side of the order, `BUY` or `SELL` |
orderType | STRING | Type of the order, `LIMIT` or `STOP` |
price | STRING | Price submitted |
lastTradedPrice | STRING | Price when order was last traded |
avgFillPrice | STRING | Average of filled price |
stopPrice | STRING | Stop price for the stop order |
limitPrice | STRING | Limit price for the stop limit order |
quantity | STRING | Quantity submitted |
remainQuantity | STRING | Remainning quantity |
filledQuantity | STRING | Filled quantity |
matchIds | LIST of dictionaries | Exchange matched IDs and information about matching orders |
matchQuantity | STRING | Matched quantity |
matchPrice | STRING | Matched price |
orderMatchType | STRING | `MAKER` or `TAKER` |
timestamp in matchIds | STRING | Time matched at|
leg1Price | STRING | |
leg2Price | STRING | |
fees | LIST of dictionaries | Fees with instrument ID |
timeInForce | STRING | Time in force |
isTriggered | STRING | `true`(for stop order) or `false` |
orderOpenedTimestamp | STRING | Order opened at |
orderModifiedTimestamp | STRING | Order modified at |
orderClosedTimestamp | STRING | Order closed at |
