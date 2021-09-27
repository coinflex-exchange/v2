# REST API V3

**TEST** site

* `https://v3stgapi.coinflex.com`

**LIVE** site

* `https://v3api.coinflex.com/v3`

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

### Place order - POST /v3/orders/place

> **Request**

```json
POST /v3/orders/place

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


### Modify order - POST /v3/orders/modify
> **Request**

```json
POST /v3/orders/modify

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


### Cancel order by ID - DELETE /v3/orders/cancel

> **Request**

```json

DELETE /v3/orders/cancel

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


### Cancel ALL orders - DELETE /v3/orders/cancel-all 

> **Request**

```json

DELETE /v3/orders/cancel-all

{
“marketCode”: “FLEX-USD”        # STRING optional, if this is sent cancel ALL orders for the market code. If this is NULL or not sent then cancel ALL orders for the account. 
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

### Account - GET /v3/account


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

### Balances - GET /v3/balances    

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

### Positions - GET /v3/positions    

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

rest_url = 'https://v3stgapi.coinflex.com'
rest_path = 'v3stgapi.coinflex.com'

api_key = <API-KEY>
api_secret = <API-SECRET>

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method
method = '/v3/positions'

# Not required for /v3/positions
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

### Trade History -  GET /v3/trades

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

# Optional for /v2/trades/{marketCode}
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
“message”: “Invalid authentication”
}

```python
{
  "event": "trades", 
  "timestamp": 1595635101845, 
  "accountId": "<Your Account ID>", 
  "data": [ {
              "matchId": "160067484555913077", 
              "matchTimestamp": "1595514663626", 
              "marketCode": "FLEX-USD", 
              "matchQuantity": "0.1", 
              "matchPrice": "0.065", 
              "total": "0.0065", 
              "orderMatchType": "TAKER", 
              "fees": "0.0096", 
              "feeInstrumentId": "FLEX", 
              "orderId": "160067484555913076", 
              "side": "SELL", 
              "clientOrderId": "123"
            },
            ...
          ]
}
```

Returns the most recent trades of the account connected to the API key initiating the request.

<sub>**Request Parameters**</sub> 

Parameters | Type | Required |Description| 
-------------------------- | -----|--------- | -------------|
marketCode| STRING | YES |  | 
limit| LONG | NO | Default `500`, max `1000` | 
startTime| LONG | NO | Millisecond timestamp | 
endTime| LONG | NO | Millisecond timestamp | 

<sub>**Response Parameters**</sub> 

Parameters |Type | Description| 
-------------------------- | -----|--------- |
event | STRING | `trades`
timestamp | INTEGER | Millisecond timestamp
accountId | STRING    | Account ID
data | LIST of dictionaries |
matchId   | STRING    | Match ID          |
matchTimestamp   | STRING    | Order Matched timestamp          |
marketCode   | STRING    | Market code          |
matchQuantity   | STRING    | Match quantity          |
matchPrice   | STRING    | Match price          |
total   | STRING    | Total price          |
side   | STRING    |  Side of the match         |
orderMatchType   | STRING    | `TAKER` or `MAKER` |
fees   | STRING    |  Fees    |
feeInstrumentId   | STRING    |   Instrument ID of the fees        |
orderId   | STRING    |	Unique order ID from the exchange          |
clientOrderID   | STRING    | Client assigned ID to help manage and identify orders  |

### Order History  - GET /v3/orders/history

> **Request**

```json
GET /v3/orders/history?marketCode={marketCode}&orderId={orderId}&clientOrderId={clientOrderId}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Success response format**

```json
{
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
“isTriggered”: false,
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

> **Failure response format**
{
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
}


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

###Delivery - POST /v3/delivery

> **Request**

```json
POST /v3/delivery

{
  "marketCode": {marketCode},    # Required
  "qtyDeliver": {qtyDeliver}        # Required
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

> **Success response format**

```json
{
“success”: true,
"data":  {
               "deliveryId": "586985384617312258",
               "marketCode": "BTC-USD-SWAP-LIN",
               "status": "PENDING”,
               "deliveryPrice": "10000.000",
               "deliveryPosition": "1.000",
               "deliveryType": "NEXT_CYCLE",
               "deliveryAsset": "USD",
                          "deliveryQuantity": "32000.000", // deliveryQuantity = quantity
               "receiveAsset": "BTC",
               "receiveQuantity": "1.000",
     “deliveredQuantity”: “0.000” // deliveredQuantity = matchedQuantity
     “receivedQuantity”: “0.000”                // how much of the receiveAsset was received
               "remainingDeliveryPosition": "1.000",
               "auctionTime": "1599204122693",
               "createdAt": "1599204122693",
               "lastUpdatedAt": "1599204122693",
          }
}

```
> **Failure response format**
{
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
}

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
\>deliverOrderId | STRING | Order id
\>accountId | STRING    | Account ID|
\>clientOrderId | Null Type|  null
\>instrumentId | STRING | Perpetual swap market code
\>deliverPrice | STRING|  Mark price at delivery
\>deliverPosition | STRING | Delivered position size
\>deliverType | STRING| ‘NEXT_CYCLE’: Queueing for the upcoming auction
\>instrumentIdDeliver | STRING |Asset being received: long position = coin, short position = USD
\>deliverQty | STRING |  Quantity of the received asset
\>remainingQty | STRING | Remaining quantity
\>remainingPosition | STRING | Remaining position
\>transferAsset | STRING | Asset being sent
\>transferQty | STRING | Quantity being sent
\>auctionTime | STRING | Millisecond timestamp of the next auction
\>created | STRING | Millisecond timestamp
\>lastUpdated | STRING | Millisecond timestamp 
\>status | STRING | Delivery status

###Cancel delivery - DELETE /v3/delivery
> **Request**

```json
DELETE /v3/delivery
{
  "deliveryId": {deliveryId}        # LONG or STRING required
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

# REST API method
method = 'DELETE /v3/delivery'


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

> **Success response format**

```json
{
“success”: true,
“data”: [{
               "deliveryId": "586985384617312258",
               "marketCode": "BTC-USD-SWAP-LIN",
               "status": "CANCELED”,
               "deliveryPrice": "10000.000",
               "deliveryPosition": "1.000",
               "deliveryType": "NEXT_CYCLE",
               "deliveryAsset": "USD",
               "deliveryQuantity": "32000.000",
               "receiveAsset": "BTC",
               "receiveQuantity": "1.000",
     “deliveredQuantity”: “0.000”
     “receivedQuantity”: “0.000”                // how much of the receiveAsset was received
               "remainingDeliveryPosition": "1.000",
               "auctionTime": "1599204122693",
               "createdAt": "1599204122693",
     “closedAt”:  "1599204182693",
               "lastUpdatedAt": "1599204122693",
          }]
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
> **Filure response format**

```json
{
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
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

###Delivery orders - GET /v3/delivery
> **Request**

```json
GET /v3/delivery?deliveryId={deliveryId}&marketCode={marketCode}&limit={limit} &startTime={startTime}&endTime={endTime}&status={status}
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
method = '/v3/delivery?deliveryId={deliveryId}&marketCode={marketCode}&limit={limit} &startTime={startTime}&endTime={endTime}&status={status} 

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
“success": true,
"data": [ {
               "deliveryId": "586985384617312258",
               "marketCode": "BTC-USD-SWAP-LIN",
               "status": "CANCELED”,
               "deliveryPrice": "10000.000",
               "deliveryPosition": "1.000",
               "deliveryType": "NEXT_CYCLE",
               "deliveryAsset": "USD",
               "deliveryQuantity": "32000.000",
               "receiveAsset": "BTC",
               "receiveQuantity": "1.000",
     “deliveredQuantity”: “0.000”
     “receivedQuantity”: “0.000”                // how much of the receiveAsset was received
               "remainingDeliveryPosition": "1.000",
               "auctionTime": "1599204122693",
               "createdAt": "1599204122693",
     “closedAt”:  "1599204182693",
               "lastUpdatedAt": "1599204122693",
          }

            }, { ……… },
          ]
}

```

> **Failure response format**

{
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
}


```python
{
  "event": "deliverOrders",
  "timestamp": "1596685339910",
  "data": [ {
              "timestamp": "1595781719394",
              "instrumentId": "BTC-USD-SWAP-LIN",
              "status": "DELIVERED",
              "quantity": null,
              "deliverPrice": "9938.480000000",
              "transferAsset": "USD",
              "transferQty": "993.848000000",
              "instrumentIdDeliver": "BTC",
              "deliverQty": "0.100000000",
              "deliverOrderId": "575770851486007299",
              "clientOrderId": null
            },
            {
              "timestamp": "1595786511155",
              "instrumentId": "BTC-USD-SWAP-LIN",
              "status": "CANCELLED",
              "quantity": null,
              "deliverPrice": "9911.470000000",
              "transferAsset": "USD",
              "transferQty": "0.000000000",
              "instrumentIdDeliver": "BTC",
              "deliverQty": "0.000000000",
              "deliverOrderId": "575786553086246913",
              "clientOrderId": null
            },
          ]
}
```

Returns the entire delivery history for the account connected to the API key initiating the request.

<sub>**Response Parameters**</sub> 

Parameters |Type | Description| 
-------------------------- | -----|--------- |
event | STRING | `deliverOrders`
timestamp | STRING | Millisecond timestamp of the repsonse
data | LIST of dictionaries |
timestamp | STRING | Millisecond timestamp of the delivery action
instrumentId | STRING | Perpetual swap market code
status | STRING | Request status
quantity | Null Type| null
deliverPrice | STRING|  Mark price at delivery
transferAsset | STRING | Asset being sent
transferQty | STRING | Quantity being sent
instrumentIdDeliver | STRING |Asset being received: long position = coin, short position = USD
deliverQty | STRING |  Quantity of the received asset
deliverOrderId | STRING | Order id
clientOrderId | Null Type|  null

###Funding Payments - GET /v3/funding
> **Request**

```json
GET /v3/funding?marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Success response format**

```json
{
        "event": "fundingPayments,
“success”: true,
        "data": [
                    {
            "marketCode": "BTC-USD-SWAP-LIN",
            "payment": "12.12321"    // negative = user credit, positive = user debit
            "fundingRate": "0.000005",
            "position": "1.123"
            "markPrice": "32000.21"
            "createdAt/updatedAt": "162163213213"
                    },...
            ]
}

```

> **Failure response format**

{
“event”: “fundingPayments”,
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
}


```python
{
  "event": "deliverOrders",
  "timestamp": "1596685339910",
  "data": [ {
              "timestamp": "1595781719394",
              "instrumentId": "BTC-USD-SWAP-LIN",
              "status": "DELIVERED",
              "quantity": null,
              "deliverPrice": "9938.480000000",
              "transferAsset": "USD",
              "transferQty": "993.848000000",
              "instrumentIdDeliver": "BTC",
              "deliverQty": "0.100000000",
              "deliverOrderId": "575770851486007299",
              "clientOrderId": null
            },
            {
              "timestamp": "1595786511155",
              "instrumentId": "BTC-USD-SWAP-LIN",
              "status": "CANCELLED",
              "quantity": null,
              "deliverPrice": "9911.470000000",
              "transferAsset": "USD",
              "transferQty": "0.000000000",
              "instrumentIdDeliver": "BTC",
              "deliverQty": "0.000000000",
              "deliverOrderId": "575786553086246913",
              "clientOrderId": null
            },
          ]
}
```

Returns the entire delivery history for the account connected to the API key initiating the request.

<sub>**Request Parameters**</sub> 

Parameters |Type | Description| 
-------------------------- | -----|--------- |
event | STRING | `deliverOrders`
timestamp | STRING | Millisecond timestamp of the repsonse
data | LIST of dictionaries |
timestamp | STRING | Millisecond timestamp of the delivery action
instrumentId | STRING | Perpetual swap market code
status | STRING | Request status
quantity | Null Type| null
deliverPrice | STRING|  Mark price at delivery
transferAsset | STRING | Asset being sent
transferQty | STRING | Quantity being sent
instrumentIdDeliver | STRING |Asset being received: long position = coin, short position = USD
deliverQty | STRING |  Quantity of the received asset
deliverOrderId | STRING | Order id
clientOrderId | Null Type|  null

###Withdrawal request - POST /v3/withdrawal
> **Request**

```json
 POST /v3/withdrawal
```

> **Success response format**

```json
{
    “success”: true,
    "data":
        {
             “asset”: “flexUSD”,
    “network”:”SLP”,
    “address”:”simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7”,
    “memo”:”642694646”,
    “quantity”: “1000.0”,
    “externalFee”:true,
“fee”:”0”,
    “status”:”pending”,
“requestedAt”:“1617940800000”
        }
}
```

> **Failure response format**

{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}


<sub>**Request Parameters**</sub> 

Parameters |Type | Description| 
-------------------------- | -----|--------- |
asset | STRING | `flexUDS`
network | STRING | 'SLP'
Address | STRING |'simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7'
memo | STRing | Only for chains that have 2 part addresses
quantity | Null Type| null
externalFee | BOLEAN |Required, if externalFee is true, fee will be deducted from account balance, if it's false, fee will be deducted from the withdrawal amount
2faType | STRING |  Authy_SECRET or GOOGLE or YUBIKEY
code | STRING | 2FA if required by the account

### Withdrawal history - GET /v3/withdrawal
> **Request**

```json
 GET /v3/withdrawal?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Success response format**

```json
{
    “success”: true,
    "data":
        [{
             “asset”: “flexUSD”,
    “network”:”SLP”,
    “address”:”simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7”,
    “Memo”:”642694646”,            // only for chains that have 2 part addresses
    “quantity”: “1000.0”,
“fee”:”0”,
“id”:”651573911056351237”
“status”:”completed”,
“txId”:”38c09755bff75d33304a3cb6ee839fcb78bbb38b6e3e16586f20852cdec4886d”,
    “requestedAt”: “1617940800000”,
    “completedAt”: “16003243243242”
        }, …………………. ]
}

```

> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}





<sub>**Response Parameters**</sub> 

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
Asset | STRING | NO | | default all assets most recent first
Limit | LONG | NO | Max 200, Default 50 |
startTime | LONG | NO | e.g. 1579450778000, default 24 hours ago|
endTime | LONG | NO | | e.g. 1613978625000, default time now

### Deposit address - GET /v3/deposit-address

> **Request**

```json
 GET /v3/deposit-address?asset={asset}&network={network}
```

> **Success response format**

```json
{
    “success”: true,
    "data":
        {
    “address”:”bitcoincash:qpsvxmpv2eqzc2cm7hf826m0afzfru6wdg4atgdxtt”,
    “legacyAddress”:”19pdycXvr7GJChTwuq5WFqrrTrMBYLYUyg”,
    “memo”:”642694646”
        }
}

```

> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}


<sub>**Requests Parameters**</sub> 

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
Asset | STRING | YES | 
Network | STRING | YES 


### Deposit history - GET /v3/deposit-history
> **Request**

```json
 GET /v3/deposit-history?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Success response format**

```json
{
    “success”: true,
    "data":
        [    {
             “asset”: “flexUSD”,
    “network”:”SLP”,
    “address”:”simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7”,
    “Memo”:”642694646”,            // only for chains that have 2 part addresses
    “quantity”: “1000.0”,
“id”:”651573911056351237”
“status”:”completed”,
“txId”:”38c09755bff75d33304a3cb6ee839fcb78bbb38b6e3e16586f20852cdec4886d”,
    “creditedAt”: “1617940800000”
            }, ………. ]
}

```

> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}


<sub>**Request Parameters**</sub> 

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
Asset | STRING | NO | | default all assets most recent first
Limit | LONG | NO | Max 200, Default 50 |
startTime | LONG | NO | e.g. 1579450778000, default 24 hours ago|
endTime | LONG | NO | | e.g. 1613978625000, default time now


###  List withdrawal addresses - GET /v3/withdrawal-addresses
> **Request**

```json
 GET /v3/withdrawal-addresses
```

> **Success response format**

```json
{
    “success”: true,
    "data":
        [    {
             “asset”: “flexUSD”,
    “network”: ”SLP”,
    “address”: ”simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7”,
    “memo”: ”642694646”,
“label”: ”Mum”
“whitelisted”: true
        }, ………. ]
}

```

> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}

###  Withdrawal fee estimate - GET /v3/withdrawal-fee
> **Request**

```json
 GET /v3/withdrawal-fee?asset=flexUSD&network=SLP&address=simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7&memo=642694646&quantity=1000&externalFee=true
```

> **Success response format**

```json
{
    “success”: true,
    "data":
        {
             “asset”: “flexUSD”,
    “network”:”SLP”,
    “address”:”simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7”,
    “memo”:”642694646”,
    “quantity”: “1000.0”,
    “externalFee”:true,
“estimatedFee”:”0”
        }
}

```

> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}



<sub>**Request Parameters**</sub> 

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
Asset | STRING | YES | | default all assets, mos recent first|
Network | STRING | YES | Network to withdraw on|
Address | STRING | YES | Address to withdraw to|
Memo | STRING | NO | Required only for 2 part addresses|
Quantity | STRING | YES | Quantity to withdraw|
externalFee | BOOL | YES | If True then the fee will be in addition to the quantity provided|

###  Sub-account balance transfer - POST /v3/transfer
> **Request**

```json
POST /v3/transfer
```

> **Success response format**

```json
{
    “asset”: “flexUSD”, 
“quantity”: 1000,
“fromAccount”:”36”,
“toAccount”:”9”
}

```

> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}

##Flex Assets - Private

### flexAsset mint - POST /v3/flexasset/mint 

Mint.

> **Request**

```json
POST /v3/flexasset/mint

{
    “asset”: “flexUSD”,         # STRING Required
“quantity”: 1000        # FLOAT or STRING Required
}

```

> **SUCCESSFUL RESPONSE**

```json
{
    “success”: true,
    "data":
        {
             “asset”: “flexUSD”,
    “quantity”: “1000.0”
        }
}

```
> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}


Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING/DECIMAL | YES | Quantity of the asset |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
asset | STRING | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING | Quantity of the asset |

### flexAsset redeem - POST /v3/flexasset/redeem

Redeem.

> **Request**

```json
POST /v3/flexasset/redeem

{
 "asset": "flexUSD",        # STRING Required
 "quantity": 1000,        # FLOAT or STRING Required
 "type": "normal" | “immediate”        # STRING Required
}

```

> **SUCCESSFUL RESPONSE**

```json
{
    “success”: true,
    "data":
        {
             “asset”: “flexUSD”,
    “quantity”: “1000.0”,
            “type”: “normal”,
    “redemptionAt”: “1617940800000”
        }
}

```
> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}



Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING/DECIMAL | YES | Quantity of the asset |
type | STRING | YES | Redeem type, available types: `Normal`, `Instant` |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
asset | STRING | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING | Quantity of the asset |
redeemAt | STRING | Redeemed time |
type | STRING | Redeem type, available types: `Normal`, `Instant` |

### flexAsset mint history - GET /v3/flexasset/mint

Get mint history by asset and sorted by time in descending order.

> **Request**

```json
GET /v3/flexasset/mint?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}

```

> **Successful response format**

```json
{
“success”: true,
"data": [  {
             “asset”: “flexUSD”,
    “quantity”: “1000.0”,
    “mintedAt”: “16003243243242”
             }, ………… ]
}
```
> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}



Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | NO | Default most recent funding first|
limit | LONG | NO | max `100`, default `100`|
startTime | STRING | NO | Millisecond timestamp, e.g. `1620977300`, default 24hours ago |
endTime | STRING | NO | Millisecond timestamp, e.g `1620977300`, default time now |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
asset | STRING | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING | Quantity of the asset |
mintedAt | STRING | Minted time in millisecond timestamp |

### flexAsset redeem history - GET /v3/flexasset/redeem

Get redemption history by asset and sorted by time in descending order.

> **Request**

```json
GET /v3/flexasset/redeem?asset={asset}&type={type}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Succesful response Format**

```json
{
    “success”: true,
    "data":
        [{
             “asset”: “flexUSD”,
    “quantity”: “1000.0”,
            “type”: “normal”,           // normal or immediate
    “requestedAt”: “16003243243242”,
    “redeemedAt”: “1643542342222”
        },]
}
```
> **Failure response Format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}


Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
asset | STRING | NO | Default most recent funding first |
limit | STRING | NO | max `100`, default `100`|
startTime | STRING | NO | Millisecond timestamp, e.g. `1620977300`, default `0` |
endTime | STRING | NO | Millisecond timestamp, e.g `1620977300`, default time now |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
asset | STRING | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING | Quantity of the asset |
requestedAt | STRING | when the redeem request was made |
redeemedAt | STRING | when the redemption was actually processed |

### flexAsset earn history - GET /v3/flexasset/earned

> **Request**

```json
GET  /v3/flexasset/earned?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}

```

> **Succesful response Format**

```json
{
    “success”: true,
    "data":
        [{
             “asset”: “flexUSD”,
    “snapshotQuantity”: “100000.0”,
    “apr”: “0.0001”,
    “rate”: “0.0001”,  // 8 decimals
    “amount”: “31.324344”, // 8 decimals
    “paidAt”: “16003243243242”,
        },]
}

```
> **Failure response Format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}



Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
asset | STRING | NO | Default most recent funding first |
limit | STRING | NO | max `500`, default `100`|
startTime | STRING | NO | Millisecond timestamp, e.g. `1620977300`, default `0` |
endTime | STRING | NO | Millisecond timestamp, e.g `1620977300`, default time now |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
asset | STRING | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING | Quantity of the asset |
requestedAt | STRING | when the redeem request was made |
redeemedAt | STRING | when the redemption was actually processed |

### noteToken earn history - GET /v3/notetoken/earned

> **Request**

```json
GET  /v3/notetoken/earned?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Succesful response Format**

```json
{
    “success”: true,
    "data":
        [{
             “asset”: “noteNibbio”,
    “snapshotQuantity”: “100000.0”,
    “apr”: “0.0001”,
    “rate”: “0.0001”,
    “amount”: “31.324344” 
    “createdAt/lastUpdatedAt”: “16003243243242”, // created time or updated time
        },]
}

```
> **Failure response Format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}




Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
asset | STRING | NO | Default most recent funding first |
limit | STRING | NO | max `500`, default `100`|
startTime | STRING | NO | Millisecond timestamp, e.g. `1620977300`, default `0` |
endTime | STRING | NO | Millisecond timestamp, e.g `1620977300`, default time now |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
asset | STRING | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING | Quantity of the asset |
requestedAt | STRING | when the redeem request was made |
redeemedAt | STRING | when the redemption was actually processed |

##AMM - Private

### Create AMM - POST /v3/AMM/create

> **Request**

```json
POST /v3/AMM/create
{
Leveraged buy/sell or neutral
    “leverage”: string or int from 1 to 10        # required
    “direction”: “BUY” | “SELL”,            # required
    “marketCode”: “BCH-USD-SWAP-LIN”,    # required
    “collateralAsset”: “BCH”,            # required
    “collateralQuantity”: "50",            # required, minimum $200 notional
    “minPriceBound”: "200",            # required 
    “maxPriceBound”: "800"            # required
    
 Unleveraged buy/sell
 
 “direction”: “BUY” | “SELL”,            # required
    “marketCode”: “BCH-USD-SWAP-LIN”,        # required
    “collateralAsset”: “BCH” | "USD",        # required
    “collateralQuantity”: "250",            # required, minimum $200 notional
    “minPriceBound”: "200",            # required 
    “maxPriceBound”: "800"                # required

Unleveraged neutral
{
    “direction”: “NEUTRAL",            # required
    “marketCode”: “BCH-USD-SWAP-LIN”,        # required
    “baseQuantity”: "3"                # required
    “counterQuantity”: "500",            # required, minimum $200 notional
    “minPriceBound”: "200",            # required 
    “maxPriceBound”: "800"                # required
}

```

> **Succesful response Format**

```json

Leveraged buy/sell or neutral
{
    
    “success”: true,
    "data":
        {
        “hashToken”: “CF-BCH-AMM-ABCDE3iy“,
        “leverage”: "5",
        “direction”: “BUY” | “SELL”,
        “marketCode”: “BCH-USD-SWAP-LIN”,
        “collateralAsset”: “BCH”,    
        “collateralQuantity”: "50",                
        “minPriceBound”: "200",
        “maxPriceBound”: "800"

 Unleveraged buy/sell
 
 “success”: true,
    "data":
        {
        “hashToken”: “CF-BCH-AMM-ABCDE3iy“,
        “direction”: “BUY” | "SELL",
        “marketCode”: “BCH-USD-SWAP-LIN”,
        “collateralAsset”: “BCH”,    
        “collateralQuantity”: "50",                
        “minPriceBound”: "200",
        “maxPriceBound”: "800"

Unleveraged neutral
{
    “success”: true,
    "data":
        {
        “hashToken”: “CF-BCH-AMM-ABCDE3iy“,
        “direction”: “NEUTRAL",
        “marketCode”: “BCH-USD-SWAP-LIN”,
        "baseQuantity": "3",    
        “counterQuantity”: "500",                
        “minPriceBound”: "200",
        “maxPriceBound”: "800"
}
}
```
> **Failure response Format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
Leverage | STRING | YES | String or int from 1 to 10|
Direction | STRING | YES |BUY/SELL|
Marketcode | STRING | YES | E.G "BCH" |
CollateralAsset | STRING | YES | E.G "BCH" |
CollateralQuantity | STRING | YES | E.G "50" minimum notional $200 |
minPriceBound | STRING | YES | "200"|
maxPriceBound | STRING | YES | "800"|

### Redeem AMM - POST /v3/AMM/redeem

> **Request**

```json
POST /v3/AMM/redeem
{
“hashToken”: “CF-BCH-AMM-ABCDE3iy“,            # STRING, required 
“type”:  “deliver” or manual or twap1hr or twap24hr”        # STRING, required
}

#TWAP1HR and TWAP24H not yet available in prod, but it will be in the future.
```

> **Succesful response Format**

```json

{
    “success”: true,
    "data": [
     {
    “hashToken”: “CF-BCH-AMM-ABCDE3iy“,
    “leverage”: null, or string (0 to 10)    
    “direction”: “BUY” or “SELL” or “NEUTRAL”,
    “marketCode”: “BCH-USD-SWAP-LIN”,
    “initialCollateral”: {“BCH”: “123”, “USD”: 0}    
    “minPriceBound”: "200",
    “maxPriceBound”: "800",
    “status”: “ENDED” or “EXECUTING”,
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
    “usdReward”: "200",
    “flexReward”: "200",
    “interestPaid”: “123”, // sum of all funding payments in tx_account_transfer
    “apr”: "0.1",
   “collateral”: “1231231”,           // notional USD
                           “notionalPositionSize”: “5000.00”, //sum(position_qty * markPrice) position表里的quantity 乘以 markPrice，然后累加
                           "portfolioVarMargin": "500",
                           "riskRatio": "20.0000",                
                             ‘maintenanceMargin’: ‘1231’,            // maintenance margin, maintenanceMargin = portfolioVarMargin / 2
                            "marginRatio": "12.3179",             // like the GUI
                           “liquidating”: false,         // flag to check if the account is being liquidated
                            ‘feeTier’: ‘6’,                // account fee tier (VIP level)
    "createdAt": "1623042343252",
    "lastUpdatedAt": "1623142532134",
      }
…….
}



```
> **Failure response Format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}
...


### Get AMM information - GET /v3/AMM

> **Request**

```json
GET /v3/AMM?hashToken=[1,2,3,4 ……. ]
```

> **Succesful response Format**

```json
{
    “success”: true,
    "data": [
     {
    “hashToken”: “CF-BCH-AMM-ABCDE3iy“,
    “leverage”: null, or string (0 to 10)    
    “direction”: “BUY” or “SELL” or “NEUTRAL”,
    “marketCode”: “BCH-USD-SWAP-LIN”,
    “initialCollateral”: {“BCH”: “123”, “USD”: 0}    
    “minPriceBound”: "200",
    “maxPriceBound”: "800",
    “status”: “ENDED” or “EXECUTING”,
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
    “usdReward”: "200",
    “flexReward”: "200",
    “interestPaid”: “123”, // sum of all funding payments in tx_account_transfer
    “apr”: "0.1",
   “collateral”: “1231231”,           // notional USD
                           “notionalPositionSize”: “5000.00”, //sum(position_qty * markPrice) position表里的quantity 乘以 markPrice，然后累加
                           "portfolioVarMargin": "500",
                           "riskRatio": "20.0000",                
                             ‘maintenanceMargin’: ‘1231’,            // maintenance margin, maintenanceMargin = portfolioVarMargin / 2
                            "marginRatio": "12.3179",             // like the GUI
                           “liquidating”: false,         // flag to check if the account is being liquidated
                            ‘feeTier’: ‘6’,                // account fee tier (VIP level)
    "createdAt": "1623042343252",
    "lastUpdatedAt": "1623142532134",
      }
…….
}

```
> **Failure response Format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}
...


Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
hashToken | List of STRING | YES | list filter, limit 10 AMM|

### Get AMM orders - GET /v3/AMM

> **Request**

```json
GET /v3/AMM/orders?hashToken={hashToken}
```

> **Succesful response Format**

```json
{ 
“success”: true,
"data": [
{ 
'orderId': '304354590153349202', 
'clientOrderId': '1', 
'marketCode': 'BTC-USD-SWAP-LIN', 
'status': PARTIALLY_FILLED | OPEN
'side': 'BUY', 
'price': '1.0',
'stopPrice': ‘0.9’,
‘isTriggered’: true,
'quantity': '0.001',
'remainQuantity': '0.001',
‘matchedQuantity’: ‘0’,
“avgFillPrice”: ‘1’,
“fees”: {‘USD’: "0", ‘FLEX’: “0”}
'orderType': 'LIMIT', 
'timeInForce': 'GTC'
'createdAt': “1613089383656”, 
'lastModifiedAt': “1613089383656”,
‘lastMatchedAt’: “1613089383656”
}]
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
hashToken | List of STRING | YES | filter|

### Get AMM positions - GET /v3/AMM

> **Request**

```json
GET /v3/AMM/positions?hashToken=[1,2,3,4 ……. ]&marketCode={marketCode}
```

> **Succesful response Format**

```json
{
    ”success”: true,
    "data":  [
        {
“hashToken”: “CF-BCH-AMM-ABCDE3iy“,
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

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
hashToken | List of STRING | YES | filter|

### Get AMM balances - GET /v3/AMM

> **Request**

```json
GET /v3/AMM/balances?hashToken=[1,2,3,4 ……. ]&asset={asset}
```

> **Succesful response Format**

```json
{
    “success”: true,
    "data": [
        {
“hashToken”: “CF-BCH-AMM-ABCDE3iy“,
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

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
hashToken | List of STRING | YES | filter|

### Get AMM trades - GET /v3/AMM

> **Request**

```json
GET /v3/AMM/trades?hashToken=1 ……. &marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Succesful response Format**

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
            “leg1Price”: “0.001”,         // REPO & SPR
            “leg2Price”: “0.001”,        // REPO & SPR
              "orderMatchType": "TAKER",
    “feeAsset”: "FLEX”,
           "fee”: "0.0096",
            "lastMatchedAt": "1595514663626"
            },
            …
]
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
hashToken | STRING | YES | filter|
marketCode | STRING | NO |
limit |INT/STRING | NO | default 500 max 1000|
startTime | INT/STRING  | NO | default 24 hours ago|
endTime | INT/STRING  | NO | default current epoch time|

##Market Data - Public

### Markets - GET /v3/markets

> **Request**

```json
GET /v3/markets`
```

> **Succesful response Format**

```json
{
    "success": true,
    "data": [
        {
            "marketCode": "BTC-USD",
            "name": "BTC/USD Spot",
            "referencePair": "BTC/USD",
            "base": "BTC",
            "counter": "USD",
            "type": "SPOT",
            "tickSize": "0.1",
            "qtyIncrement": "0.001",
            "listedAt": "2208988800000",
    “settlementAt”: “16343242424223”, // null when no settlement
            "upperPriceBound": "11000.00",
            "lowerPriceBound": "9000.00",
            "markPrice": "10000.00",
    “lastUpdatedAt”: “1620810131131”
        },
        …...
    ]
}

```
> **Failure response Format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
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

### Assets - GET /v3/assets

> **Request**

```json
GET /v3/assets
```

> **Succesful response Format**

```json
{
    “success”: true,
    "data": [
        {
            "name": "BTC",
    “canDeposit”: true,
            "canWithdraw": true,
            "isCollateral": true,
            "loanToValue": "0.9",
    “minDeposit”: “0.0001”,
    “minWithdrawal”: “0.0001”,
            “network”: [“erc20”, “trc20”, “bep20”],
},
    ]
}


```
> **Failure response Format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
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

### Tickers - GET /v3/ticker

> **Request**

```json
GET /v3/ticker?marketCode={marketCode}
```

> **Succesful response Format**

```json
{
"success": true,
"data" :
[{
"marketCode": "BTC-USD-SWAP-LIN",
"markPrice": "11012.80409769",  
"open24h": "49.375",
"high24h": "49.488",
"low24h": "41.649",
"volume24h": "11295421",
"currencyVolume24h": "1025.7", 
"openInterest": "1726003",
"lastTradedPrice": "43.259",
"lastTradedQuantity": "1”,
“lastUpdatedAt”: “1621321313131”
},
]
}
```

> **Failure response Format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
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

### Upcoming delivery auction - GET /v3/auction?marketCode={marketCode}

> **Request**

```json
GET /v3/auction?marketCode={marketCode}
marketCode={marketCode}
```
> **Sucess response Format**

```json
{
    "success": true,
    "data": [
        {
            "marketCode": "BTC-USD-SWAP-LIN",
            "auctionTime": "1596620805000",
            "netDelivered": "-5.100000000",
            "estFundingRate": "0.0001"
        },
      ...
    ]
}

```
> **Failure response Format**
 {
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}
```

Requires authentication. Get entire delivery history.

Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
timestamp | STRING | UNIX Timestamp of the response|
instrumentId | STRING |  Market code
auctionTime | STRING | UNIX timestamp of the next auction
netDeliver | STRING | Delivery imbalance (negative = more shorts than longs and vice versa)
estFundingRate | STRING | Estimated funding rate a positive rate means longs pay shorts

### Historical funding rates - GET /v3/funding-rates

Get funding rates by marketCode and sorted by time in descending order.

> **Request**

```json
GET /v3/funding-rates?marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}

marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "success": true,
    "data": [
        {
         "marketCode": "BTC-USD-SWAP-LIN" //all marketCode should be x-USD-SWAP-LIN,
            "fundingRate": "0.00008",
            "markPrice": "33937.2848",
            "index": "2.78222423",
    "netDelivered": "-5.100000000",
            "createdAt/lastUpdatedAt": "1606461774458" // created time or updated time
},
...
    ]
}
```
> **FAILURE RESPONSE RESPONSE**
```json
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
marketCode | STRING | YES | e.g. BTC-USD-REPO-LIN , available values: BTC-USD-REPO-LIN, ETH-USD-REPO-LIN |
limit | LONG | NO | default is `50` |
startTime | LONG | NO | millisecond timestamp, e.g. `1579450778000`, default is 7 days ago from time now |
endTime | LONG | NO | millisecond timestamp, e.g. `1613978625000`, default is time now |

Response Fields | Type | Description |
------------------- | ---- | ----------- |
timestamp | STRING | Timestamp of this response |
fundingRate | STRING | |
index | STRING | |
markPrice | STRING | |
marketCode | STRING | |
timestamp(in the data list) | STRING | |

###Exchange trades - GET /v3/exchange-trades

> **Request**

```json
GET /v3/exchange-trades?marketCode={marketCode}&limit={limit}
&startTime={startTime}&endTime={endTime}

```

> **Success response format**


```json
{
"success": true, 
"data": [
{
"marketCode": "BTC-USD-SWAP-LIN",
"matchPrice": "9600.000000000", 
"matchQuantity": "0.100000000", 
"side": "BUY",
"lastMatchedAt": "1595585860254"
},
]
}

```
> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}

```

Get most recent trades.

Request Parameters | Type | Required |Description| 
-------------------------- | -----|--------- | -------------|
marketCode| STRING | YES |  | 
limit| LONG | NO | Default 100, max 300 | 
startTime| LONG | NO |  | 
endTime| LONG | NO |  | 

Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
timestamp | STRING    | Timestamp of this response|
marketCode | STRING    | |
matchID | STRING    | |
matchQuantity | STRING    | |
matchPrice | STRING    | |
side | STRING    | |
matchTimestamp | STRING    | |

### Candles - GET /v3/candles

Get historical candles of active and expired markets.

> **Request**

```json
GET /v3/candles?marketCode={marketCode}&timeframe={timeframe}&limit={limit}
&startTime={startTime}&endTime={endTime}

marketCode={marketCode}&timeframe={timeframe}&limit={limit}&startTime={startTime}&endTime={endTime}


> **Failure response format**

```json
{
    "success": true,
    "timeframe": "60s",
    "data": [
        {
            "open": "51706.50000000",
            "high": "51758.50000000",
            "low": "51705.50000000",
            "close": "51754.00000000",
            "volume24h": "0",
            "currencyVolume24h": "0",
"openedAt": "1616713140000”
        },
        {
            
            "open": "51755.50000000",
            "high": "51833.00000000",
            "low": "51748.00000000",
            "close": "51815.00000000",
            "volume24h": "0",
            "currencyVolume24h": "0",
"openedAt": "1616713200000”
        },
        ...
    ]
}
```
> **Failure response format**
 {
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
marketCode | STRING | YES | When marketCode is expired market like `BTC-USD-201225-LIN`, the startTime and the endTime should be explicitly set in `2020` |
timeframe | STRING | NO | e.g. `60s`, `300s`, `900s`, `1800s`, `3600s`, `7200s`, `14400s`, `86400s`, default `3600s ` |
limit | LONG | NO | max `5000 `, default `500`|
startTime | LONG | NO | Millisecond timestamp, e.g. `1579450778000`, default is `limit` times `timeframe` ago, if the limit is `300` and the timeframe is `3600s` then the default startTime is `time now - 300x3600s`, if the limit is not present and the timeframe is `3600s` then the default startTime is `time now - 500x3600s` |
endTime | LONG | NO | Millisecond timestamp, e.g `1579450778000`, default time now |
```
Response Fields | Type | Description |
----------------| ---- | ----------- |
timestamp(outer) | STRING | |
timeframe | STRING | Selected timeframe |
timestamp(inner) | STRING | Beginning of the candle |
open | STRING | |
high | STRING | |
low | STRING | |
close | STRING | |
volume24h | STRING | 24 hour rolling trading volume in counter currency |
currencyVolumn24h | STRING | 24 hour rolling trading volume in base currency |

### Orderbook depth - GET /v3/depth

Get order book by marketCode and level.

> **Request**

```json
GET /v3/depth?marketCode={marketCode}&level={level}
marketCode={marketCode}&level={level}
```

> **Sucess response format**

```json
{
    "success": true,
    “level”: “5”,
    "data": {
        "marketCode": "BTC-USD-SWAP-LIN",
            "lastUpdatedAt": "1615457834388”
            "asks": [     [54792, 0.001],
                    [54802.5, 0.366],
                    [54803, 0.75],
                    [54806, 1.5],
                    [54830.5, 0.687]        
],
            "bids": [    [54786.5, 0.1],
                    [54754.5, 0.1],
                    [54752, 0.1],
                    [54749.5, 0.1],
                    [54745.5, 0.1]        
]
}
}
```
> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}

```

Response Fields | Type | Description |
----------------| ---- | ----------- |
event | STRING | |
timestamp | STRING | |
data | LIST | |
asks | LIST of floats | Sell side depth: <ol><li>price</li><li>quantity</li><li>0</li><li>0</li></ol> |
bids | LIST of floats | Buy side depth: <ol><li>price</li><li>quantity</li><li>0</li><li>0</li></ol> |
marketCode | STRING | |
timestamp | STRING | |

## flexAssets - Public

### flexAsset balances - GET /v3/flexasset/balances/{asset}

Get flexAsset balances.

> **Request**

```json
GET /v3/flexasset/balances/{asset}
```

> **Sucess response format**

```json
{
'success’: true,
'asset': ‘flexUSD’,
'data':  [{
'instrumentId': 'BAND', 
'total': '0.00000000', 
'available': '0.00000000', 
'reserved': '0.00000000', 
'lastUpdatedAt': '1602918327182'
}, 
{
'instrumentId': 'BNB', 
'total': '0.00000000', 
'available': '0.00000000', 
'reserved': '0.00000000', 
'lastUpdatedAt': '1602647599866'
}]
}
```

> **Failure response format**
{
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
flexProtocol | STRING | YES | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |

Response Fields | Type | Description |
----------------| ---- | ----------- |
tradeType | STRING | Trade type |
flexProtocol | STRING | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
instrumentId | STRING |Coin symbol, e.g. 'BTC' |
total| STRING | Total balance |
available | STRING | Available balance |
reserved | STRING|Reserved balance (unavailable) due to working spot orders |
quantityLastUpdated | STRING|Millisecond timestamp of when balance was last updated |

### flexAsset positions - GET /v3/flexasset/positions

Get flexAsset positions.

> **Request**

```json
GET /v3/flexasset/positions?asset={asset}
asset={asset}
```
> **Sucessful response format**

```json
{
'success': true,
'data': [{
"marketCode": "BTC-USD-SWAP-LIN",
    “baseAsset”: “BTC”,
    “counterAsset”: “USD”,
"quantity": "-0.94",
              "entryPrice": "7800.00", 
“markPrice”: “33000.00”, 
              "positionPnl": "200.3",
“estLiquidationPrice”: “12000.05”,
“lastUpdatedAt": "1592486212218"      
},
{
"marketCode": "ETH-USD-SWAP-LIN",
    “baseAsset”: “ETH”,
    “counterAsset”: “USD”,
"quantity": "0.94",
              "entryPrice": "7800.00", 
“markPrice”: “33000.00”, 
              "positionPnl": "200.3",
“estLiquidationPrice”: “12000.05”,
“lastUpdatedAt": "1592486212218"      
},.....]

}
```
> **Failure response format**
{
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
}
```


Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
flexProtocol | STRING | YES | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |

Response Fields | Type | Description |
----------------| ---- | ----------- |
flexProtocol | STRING | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
instrumentId | STRING | Contract symbol, e.g. `FLEX-USD-SWAP-LIN` |
quantity | STRING | Quantity of position, e.g. `0.94` |
lastUpdated | STRING | Timestamp when position was last updated |
contractValCurrency | STRING | Contract valuation currency |
entryPrice | STRING | Average entry price |
positionPnl | STRING | Postion profit and lost |

### flexAsset open orders - GET /v3/flexasset/orders

Get flexAsset orders.

> **Request**

```json
GET /v3/flexasset/orders?asset={asset}
asset={asset}
```

> **Sucessful response format**

```json
{
‘success’: true,
'asset': ‘flexUSD’,
'data': [{
'orderId': '304354590153349202', 
'clientOrderId': '1', 
'marketCode': 'BTC-USD-SWAP-LIN', 
'side': 'BUY', 
'orderType': 'LIMIT', 
'quantity': '0.001', 
'remainQuantity': '0.001',
‘matchedQuantity’: ‘0’,
'price': '1.0',
'stopPrice': Null, 
'createdAt': “1613089383656”, 
'lastModifiedAt': “1613089383656”,
‘lastMatchedAt’: “1613089383656”,
'timeInForce': 'GTC'
}]
}
```
> **Failure response format**
{
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
flexProtocol | STRING | YES | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |

Response Fields | Type | Description |
----------------| ---- | ----------- |
flexProtocol | STRING | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
orderId | STRING | Unique order ID from the exchange |
marketCode| STRING | Market code |
clientOrderId| STRING | Client assigned ID to help manage and identify orders |
side | STRING | `BUY` or `SELL` |
orderType | STRING | `LIMIT` or `STOP` |
quantity  | STRING | Quantity submitted |
remainingQuantity|STRING | Remainning quantity |
price | STRING | Price submitted |
stopPrice | STRING | Stop price for the stop order |
limitPrice| STRING | Limit price for the stop limit order |
orderCreated| INTEGER | Timestamp when order was created |
lastModified| INTEGER | Timestamp when order was last mordified |
lastTradeTimestamp| INTEGER | Timestamp when order was last traded |
timeInForce | STRING | Time in force |

### flexAsset trades - GET v3/flexasset/trades

Get flexAsset trades.

> **Request**

```json
GET /v3/flexasset/trades?asset={asset}&marketCode={marketCode}&limit={limit} &startTime={startTime}&endTime={endTime}

asset={asset}&marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Sucess response format**

```json
{
‘success: true,
'asset': ‘flexUSD’,
'data':[{
        "orderId": "160067484555913076",
              "clientOrderId": "123",
              "matchId": "160067484555913077",
              "marketCode": "FLEX-USD",
        "side": "SELL",
              "matchQuantity": "0.1",
              "matchPrice": "0.065",
              "total": "0.0065",
              "orderMatchType": "TAKER",
    “feeAsset”: "FLEX”,
           "fee”: "0.0096",
    "lastMatchedAt": "1595514663626"
}]
}
```
> **Failure response format**
{
“success”: false,
“code”: “40002”,
“message”: “Invalid key”
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
flexProtocol | STRING | YES | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
marketCode | STRING | YES | |
limit | LONG | NO | max `100`, default `100`, max `500` |
startTime | LONG | NO | e.g. `1579450778000`, default `0` |
endTime | LONG | NO | e.g. `1613978625000`, default time now |


Response Fields | Type | Description |
----------------| ---- | ----------- |
flexProtocol | STRING | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
matchId | STRING | Match ID |
matchTimestamp | STRING | Order Matched timestamp |
marketCode | STRING | Market code |
matchQuantity | STRING | Match quantity |
matchPrice | STRING | Match price |
total | STRING | Total price |
side | STRING | Side of the match |
orderMatchType | STRING | `TAKER` or `MAKER` |
fees | STRING | Fees |
feeInstrumentId | STRING | Instrument ID of the fees |
orderId | STRING | Unique order ID from the exchange |
clientOrderID | STRING | Client assigned ID to help manage and identify orders |

### flexAsset delivery orders - GET /v3/flexasset/delivery

> **Request**

```json
GET /v3/flexasset/delivery?asset={asset}&marketCode={marketCode}&limit={limit} &startTime={startTime}&endTime={endTime}&status={status}
asset={asset}&marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}&status={status}
```

> **Sucess response format**

```json
{
{
    "success": true,
    “asset”: “flexUSD”,
    "data": [
        {
"marketCode": "BTC-USD-SWAP-LIN",
              "deliveryId": "575770851486007299",
              "status": "PENDING",
              "price": "9938.480000000",
              "deliverAsset": "USD",
              "deliverQty": "993.848000000",
              "receiveAsset": "BTC",
              "receiveQty": "0.100000000",
              "createdAt": "1595781719394",
    “lastMatchedAt”: "1595781719394", // null if no match yet (status == “PENDING”)
    “
            },
            { ………………...}
        },
      ...
    ]
}
```
> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}

```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
flexProtocol | STRING | YES | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
marketCode | STRING | YES | |
limit | LONG | NO | max `100`, default `100`, max `500` |
startTime | LONG | NO | e.g. `1579450778000`, default `0` |
endTime | LONG | NO | e.g. `1613978625000`, default time now |


Response Fields | Type | Description |
----------------| ---- | ----------- |
flexProtocol | STRING | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
matchId | STRING | Match ID |
matchTimestamp | STRING | Order Matched timestamp |
marketCode | STRING | Market code |
matchQuantity | STRING | Match quantity |
matchPrice | STRING | Match price |
total | STRING | Total price |
side | STRING | Side of the match |
orderMatchType | STRING | `TAKER` or `MAKER` |
fees | STRING | Fees |
feeInstrumentId | STRING | Instrument ID of the fees |
orderId | STRING | Unique order ID from the exchange |
clientOrderID | STRING | Client assigned ID to help manage and identify orders |


### flexAsset yields - GET /v3/flexasset/yields

> **Request**

```json
 GET /v3/flexasset/yields?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Sucess response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "flexUSD",
            "apr": "0.0001",
            "interestRate": "0.0001",
    “amount”: “1231”
            "paidAt": "16003243243242"
        },
        {
            "asset": "flexBTC",
            "apr": "0.0001",
            "interestRate": "0.0001",
    “amount”: “1231”
            "paidAt": "16003243243242"
        },
        ...
    ]
}


```
> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
flexProtocol | STRING | YES | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
marketCode | STRING | YES | |
limit | LONG | NO | max `100`, default `100`, max `500` |
startTime | LONG | NO | e.g. `1579450778000`, default `0` |
endTime | LONG | NO | e.g. `1613978625000`, default time now |


Response Fields | Type | Description |
----------------| ---- | ----------- |
flexProtocol | STRING | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
matchId | STRING | Match ID |
matchTimestamp | STRING | Order Matched timestamp |
marketCode | STRING | Market code |
matchQuantity | STRING | Match quantity |
matchPrice | STRING | Match price |
total | STRING | Total price |
side | STRING | Side of the match |
orderMatchType | STRING | `TAKER` or `MAKER` |
fees | STRING | Fees |
feeInstrumentId | STRING | Instrument ID of the fees |
orderId | STRING | Unique order ID from the exchange |
clientOrderID | STRING | Client assigned ID to help manage and identify orders |

### noteToken yields - GET /v3/notetoken/yields

> **Request**

```json
GET /v3/notetoken/yields?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Sucess response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "nibbioUSD",
            "apr": "0.0001",
            "interestRate": "0.0001",
    “amount”: “1231”
            "paidAt": "16003243243242"
        },
        {
            "asset": "grapfruitUSD",
            "apr": "0.0001",
            "interestRate": "0.0001",
    “amount”: “1231”
            "paidAt": "16003243243242"
        },
        ...
    ]
}

```
> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}

```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
flexProtocol | STRING | YES | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
marketCode | STRING | YES | |
limit | LONG | NO | max `100`, default `100`, max `500` |
startTime | LONG | NO | e.g. `1579450778000`, default `0` |
endTime | LONG | NO | e.g. `1613978625000`, default time now |


Response Fields | Type | Description |
----------------| ---- | ----------- |
flexProtocol | STRING | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
matchId | STRING | Match ID |
matchTimestamp | STRING | Order Matched timestamp |
marketCode | STRING | Market code |
matchQuantity | STRING | Match quantity |
matchPrice | STRING | Match price |
total | STRING | Total price |
side | STRING | Side of the match |
orderMatchType | STRING | `TAKER` or `MAKER` |
fees | STRING | Fees |
feeInstrumentId | STRING | Instrument ID of the fees |
orderId | STRING | Unique order ID from the exchange |
clientOrderID | STRING | Client assigned ID to help manage and identify orders |

### Leverage tiers - GET /v3/leverage-tiers?marketCode={marketCode}

> **Request**

```json
GET /v3/leverage-tiers?marketCode={marketCode}

marketCode={marketCode}
```

> **Sucess response format**

```json
{
    "success": true,
    "data": [
        {
           "asset": "BTC",
           "tiers": [[100, 5], [50, 20], [20, 100], [10, 350], … ] //[leverage, position] 
        },
        {
           "asset": "ETH",
           "tiers": [[100, 5], [50, 200], [20, 500], …]
        },
        ...
    ]
}

```
> **Failure response format**
{
“success”: false,
“code”: “41002”,
“message”: “Internal server error”
}


```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
flexProtocol | STRING | YES | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
marketCode | STRING | YES | |
limit | LONG | NO | max `100`, default `100`, max `500` |
startTime | LONG | NO | e.g. `1579450778000`, default `0` |
endTime | LONG | NO | e.g. `1613978625000`, default time now |


Response Fields | Type | Description |
----------------| ---- | ----------- |
flexProtocol | STRING | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
matchId | STRING | Match ID |
matchTimestamp | STRING | Order Matched timestamp |
marketCode | STRING | Market code |
matchQuantity | STRING | Match quantity |
matchPrice | STRING | Match price |
total | STRING | Total price |
side | STRING | Side of the match |
orderMatchType | STRING | `TAKER` or `MAKER` |
fees | STRING | Fees |
feeInstrumentId | STRING | Instrument ID of the fees |
orderId | STRING | Unique order ID from the exchange |
clientOrderID | STRING | Client assigned ID to help manage and identify orders |

