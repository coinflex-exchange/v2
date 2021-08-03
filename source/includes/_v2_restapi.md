# REST API

**TEST** site

* `https://v2stgapi.coinflex.com`

**LIVE** site

* `https://v2api.coinflex.com/v2`

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

##Methods - Private

All private REST API methods require authentication using the approach explained above. 

###GET `/v2/accountinfo`

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

###GET `/v2/balances`

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
\>instrumentId | STRING |Coin symbol, e.g. 'BTC' |
\>total| STRING| Total balance|
\>available |STRING| Available balance|
\>reserved|STRING|Reserved balance (unavailable) due to working spot orders|
\>quantityLastUpdated|STRING|Millisecond timestamp of when balance was last updated|


###GET `/v2/balances/{instrumentId}`

>**Request**

```json
GET /v2/balances/{instrumentId}
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

# REST API method, for example FLEX balances
method = '/v2/balances/FLEX'

# Not required for /v2/balances/{instrumentId}
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
  "event": "balancesById",
  "timestamp": 1593627415293,
  "accountId": "<Your Account ID>",
  "tradeType": "LINEAR",
  "data": [ {   
              "instrumentId": "FLEX",
              "total": "4468.823",              
              "available": "4468.823",        
              "reserved": "0",
              "quantityLastUpdated": "1593627415001"
            } ]
}
```

```python
{
  "event": "balancesById",
  "timestamp": 1593627415293,
  "accountId": "<Your Account ID>",
  "tradeType": "LINEAR",
  "data": [ {   
              "instrumentId": "FLEX",
              "total": "4468.823",              
              "available": "4468.823",        
              "reserved": "0",
              "quantityLastUpdated": "1593627415001"
            } ]
}
```

Returns the specified coin balance of the account connected to the API key initiating the request. 

<sub>**Response Parameters**</sub> 

Parameters |Type | Description| 
-------------------------- | -----|--------- |
event | STRING | `balancesById`
timestamp | INTEGER | Millisecond timestamp
accountId | STRING    | Account ID
tradeType | STRING    | `LINEAR` |
data | LIST of dictionary |
\>instrumentId | STRING |Coin symbol, e.g. 'FLEX' |
\>total| STRING| Total balance|
\>available |STRING|Available balance|
\>reserved|STRING|Reserved balance (unavailable) due to working spot orders|
\>quantityLastUpdated|STRING|Timestamp when was balance last updated|

###GET  `/v2/positions`

> **Request**

```json
GET /v2/positions
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


> **Response**

```json
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


###GET  `/v2/positions/{instrumentId}`

> **Request**

```json
GET /v2/positions/{instrumentId}
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

# REST API method, for example FLEX-USD-SWAP-LIN position
method = '/v2/positions/FLEX-USD-SWAP-LIN'

# Not required for /v2/positions/{instrumentId}
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
  "event": "positionsById",
  "timestamp": 1593617005438,
  "accountId":"<Your Account ID>",
  "data": [ {
              "instrumentId": "BTC-USD-SWAP-LIN",
              "quantity": "0.542000000",
              "lastUpdated": "1617099855966",
              "contractValCurrency": "BTC",
              "entryPrice": "56934.8258",
              "positionPnl": "1216.6135",
              "estLiquidationPrice": "53171.16"
          } ]
}
```

```python
{
  "event": "positionsById",
  "timestamp": 1593617005438,
  "accountId":"<Your Account ID>",
  "data": [ {
              "instrumentId": "BTC-USD-SWAP-LIN",
              "quantity": "0.542000000",
              "lastUpdated": "1617099855966",
              "contractValCurrency": "BTC",
              "entryPrice": "56934.8258",
              "positionPnl": "1216.6135",
              "estLiquidationPrice": "53171.16"
          } ]
}
```

Returns the specified instrument ID position of the account connected to the API key initiating the request.

Response Fields | Type | Description |
--------------- | ---- | ----------- |
event | STRING | `positionsById` |
timestamp | INTEGER | Millisecond timestamp |
accountId | STRING | Account ID |
data | LIST of dictionaries | |
instrumentId | STRING | Contract symbol, e.g. 'FLEX-USD-SWAP-LIN' |
quantity | STRING | Quantity of position, e.g. '0.94' |
lastUpdated | STRING | Timestamp when position was last updated |
contractValCurrency | STRING | Contract valuation currency |
entryPrice | STRING | Average entry price |
positionPnl | STRING | Postion profit and lost |
estLiquidationPrice | STRING | Estimated liquidation price, return 0 if it is negative(<0) |


###GET `/v2/trades/{marketCode}`

> **Request**

```json
GET /v2/trades/{marketCode}?limit={limit}&startTime={startTime}&endTime={endTime}

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

rest_url = 'https://v2stgapi.coinflex.com'
rest_path = 'v2stgapi.coinflex.com'

api_key = <API-KEY>
api_secret = <API-SECRET>

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

# REST API method, for example FLEX-USD spot trades
method = '/v2/trades/FLEX-USD'

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

> **Response**

```json
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

###GET  `/v2/orders`

> **Request**

```json
GET /v2/orders
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
method = '/v2/orders'

# Not required for /v2/orders
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
  "event": "orders",
  "timestamp": "1593617005438",
  "accountId": "<Your Account ID>",
  "data": [ {
              "orderId": "160039151345856176",
              "marketCode": "BTC-USD-SWAP-LIN",
              "clientOrderId": null|"<clientOrderId>",              
              "side": "BUY",
              "orderType": "LIMIT"|"STOP",
              "quantity": "1.00",
              "remainingQuantity": "1.00",
              "price": "1.00"|null,               #for limit order, null for stop order
              "stopPrice": "<stopPrice>"|null,    #for stop order, null for limit order 
              "limitPrice": "<limitPrice>"|null,  #for stop order, null for limit order 
              "orderCreated": 1593617008698,
              "lastModified": 1593617008698,
              "lastTradeTimestamp": 1593617008698,
              "timeInForce": "GTC"
            },
            ...
          ]
}
```

```python
{
  "event": "orders",
  "timestamp": "1593617005438",
  "accountId": "<Your Account ID>",
  "data": [ {
              "orderId": "160039151345856176",
              "marketCode": "BTC-USD-SWAP-LIN",
              "clientOrderId": null|"<clientOrderId>",              
              "side": "BUY",
              "orderType": "LIMIT"|"STOP",
              "quantity": "1.00",
              "remainingQuantity": "1.00",
              "price": "1.00"|null,               #for limit order, null for stop order
              "stopPrice": "<stopPrice>"|null,    #for stop order, null for limit order 
              "limitPrice": "<limitPrice>"|null,  #for stop order, null for limit order 
              "orderCreated": 1593617008698,
              "lastModified": 1593617008698,
              "lastTradeTimestamp": 1593617008698,
              "timeInForce": "GTC"
            },
            ...
          ]
}
```

Returns all the open orders of the account connected to the API key initiating the request.

<sub>**Response Parameters**</sub> 

Parameters |Type | Description| 
-------------------------- | -----|--------- |
event | STRING | `orders`
timestamp | STRING | Millisecond timestamp
accountId | STRING | Account ID
data | LIST of dictionaries |
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


### GET `/v2.1/orders`

> **Request**

```json
GET /v2.1/orders?marketCode={marketCode}&orderId={orderId}&clientOrderId={clientOrderId}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Response**

```json
{
    "event": "orders",
    "timestamp": "1619167719563",
    "accountId": "1076",
    "data": [
        {
            "status": "OrderClosed",
            "orderId": "304408197314577142",
            "clientOrderId": "1",
            "marketCode": "BTC-USD-SWAP-LIN",
            "side": "BUY",
            "orderType": "LIMIT",
            "price": "10006.0",
            "quantity": "0.001",
            "remainQuantity": "0.001",
            "timeInForce": "GTC",
            "orderClosedTimestamp": "1619131050779"
        },
        {
            "status": "OrderOpened",
            "orderId": "304408197314577143",
            "clientOrderId": "2",
            "marketCode": "BTC-USD-SWAP-LIN",
            "side": "SELL",
            "orderType": "LIMIT",
            "price": "60006.0",
            "quantity": "0.001",
            "remainQuantity": "0.001",
            "timeInForce": "GTC",
            "orderOpenedTimestamp": "1619131049574"
        },
        {
            "status": "OrderMatched",
            "orderId": "448528458527567629",
            "clientOrderId": "1618870087524",
            "marketCode": "FLEX-USD-SWAP-LIN",
            "side": "BUY",
            "orderType": "MARKET",
            "price": "0.194",
            "lastTradedPrice": "0.170",
            "avgFillPrice": "0.170",
            "quantity": "12.1",
            "filledQuantity": "12.1",
            "remainQuantity": "0",
            "matchIds": [
                {
                    "448528458527567630": {
                        "matchQuantity": "12.1",
                        "matchPrice": "0.170",
                        "timestamp": "1618870088471",
                        "orderMatchType": "TAKER"
                    }
                }
            ],
            "fees": {
                "FLEX": "-0.00440786"
            },
            "timeInForce": "IOC",
            "isTriggered": "false"
        },
        ...
    ]
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


###DELETE `/v2/cancel/orders`

> **Request**

```json
DELETE /v2/cancel/orders 
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
method = '/v2/cancel/orders'

# Not required for /v2/orders
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

> **Response**

```json
{
  "event": "orders",
  "timestamp": 1594412077100,
  "accountId": "<AccountID>",
  "data": {
    "msg": "All open orders for the account have been queued for cancellation"
  }
}
```

```python
{
  "event": "orders",
  "timestamp": 1594412077100,
  "accountId": "<AccountID>",
  "data": {
            "msg": "All open orders for the account have been queued for cancellation"
          } 
}
```

Cancels **all** open orders of the account connected to the API key initiating the request.

If this REST method was sucessful it will also trigger a reponse message in an authenticated websocket of the account.  This is documented here [Cancel Open Orders](#websocket-api-other-responses-cancel-open-orders).

<sub>**Response Parameters**</sub> 

Parameters |Type | Description| 
-------------------------- | -----|--------- |
event | STRING | `orders`
timestamp | INTEGER | Millisecond timestamp
accountId | STRING    | Account ID
data | Dictionary |
\>msg   | STRING    | Confirmation of action |


###DELETE `/v2/cancel/orders/{marketCode}`

> **Request**

```json
DELETE /v2/cancel/orders/{marketCode}
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

# REST API method, for example FLEX-USD spot market
method = '/v2/cancel/orders/FLEX-USD' 

# Not required for /v2/orders/{marketCode}
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

> **Response**

```json
{
  "event": "orders",
  "timestamp": 1594412077100,
  "accountId": "<AccountID>",
  "data": {
            "marketCode": "FLEX-USD",
            "msg": "All open orders for the specified market have been queued for cancellation"
          }
}
```

```python
{
  "event": "orders",
  "timestamp": 1594412077100,
  "accountId": "<AccountID>",
  "data": {
            "marketCode": "FLEX-USD",
            "msg": "All open orders for the specified market have been queued for cancellation"
          }
}
```

Cancels all open orders for the **specified market** for the account connected to the API key initiating the request.

If this REST method was sucessful it will also trigger a reponse message in an authenticated websocket of the account.  This is documented here [Cancel Open Orders](#websocket-api-other-responses-cancel-open-orders).

<sub>**Response Parameters**</sub> 

Parameters |Type | Description| 
-------------------------- | -----|--------- |
event | STRING | `orders`
timestamp | INTEGER | Millisecond timestamp
accountId | STRING    | Account ID
data | Dictionary |
\>marketCode   | STRING | Market code |
\>msg   | STRING | Confirmation of action |


###GET `/v2.1/delivery/orders`
> **Request**

```json
GET /v2.1/delivery/orders
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

# Not required for GET /v2.1/delivery/orders
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


###POST `/v2.1/delivery/orders`
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

###DELETE `/v2.1/delivery/orders/{deliveryOrderId}`
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

Cancels a pending delivery.

###POST `/v2/orders/place`
> **Request**

```json
POST /v2/orders/place

{
    "recvWindow": 20000, 
    "responseType": "FULL", 
    "timestamp": 1615430912440, 
    "orders": [
        {
            "clientOrderId": "1612249737724", 
            "marketCode": "BTC-USD-SWAP-LIN", 
            "side": "SELL", 
            "quantity": "0.001", 
            "timeInForce": "GTC", 
            "orderType": "LIMIT", 
            "price": "50007"
        }, 
        {
            "clientOrderId": "1612249737724", 
            "marketCode": "BTC-USD-SWAP-LIN", 
            "side": "BUY", 
            "quantity": "0.002", 
            "timeInForce": "GTC", 
            "orderType": "LIMIT", 
            "price": "54900"
        }, 
        {
            "clientOrderId": "1612249737723", 
            "marketCode": "BTC-USD-SWAP-LIN", 
            "side": "SELL", 
            "quantity": "0.003", 
            "timeInForce": "GTC", 
            "orderType": "LIMIT", 
            "price": "54901"
        }
    ]
}
```

> **RESPONSE**

```json
{
    "accountId": "13670827", 
    "event": "placeOrder", 
    "timestamp": "1615430915625", 
    "data": [
        {
            "success": "false", 
            "timestamp": "1615430915596", 
            "code": "710002", 
            "message": "FAILED sanity bound check as price (50007.0) <  lower bound (54767.0)", 
            "clientOrderId": "1612249737724", 
            "orderId": "0", 
            "price": "50007.0", 
            "quantity": "0.001", 
            "side": "SELL", 
            "marketCode": "BTC-USD-SWAP-LIN", 
            "timeInForce": "GTC", 
            "orderType": "LIMIT"
        }, 
        {
            "success": "true", 
            "timestamp": "1615430915602", 
            "clientOrderId": "1612249737724", 
            "orderId": "0", 
            "price": "54900.0", 
            "quantity": "0.002", 
            "side": "BUY", 
            "marketCode": "BTC-USD-SWAP-LIN", 
            "timeInForce": "GTC", 
            "orderType": "LIMIT"
        }, 
        {
            "success": "true", 
            "timestamp": "1615430915625", 
            "clientOrderId": "1612249737723", 
            "orderId": "0", 
            "price": "54901.0", 
            "quantity": "0.003", 
            "side": "SELL", 
            "marketCode": "BTC-USD-SWAP-LIN", 
            "timeInForce": "GTC", 
            "orderType": "LIMIT"
        }
    ]
}
```

Place orders.

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
recvWindow | LONG | NO | |
timestamp | STRING | NO | |
responseType | STRING | YES | `FULL` or `ACK` |
orders | LIST | YES | |
clientOrderId | STRING | YES | |
marketCode | STRING | YES | |
side | STRING | YES | |
quantity | STRING | YES | |
timeInForce | STRING | NO | Default `GTC` |
orderType | STRING | YES | |
price | STRING | NO | |
stopPrice | STRING | NO | |
limitPrice | STRING | NO | |

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


### POST `/v2/orders/modify`
> **Request**

```json
POST /v2/orders/modify

{
    "recvWindow": 13000, 
    "responseType": "FULL", 
    "timestamp": 1614331650143, 
    "orders": [
        {
            "clientOrderId": "1614330009059", 
            "orderId": "304369975621712260", 
            "marketCode": "BTC-USD-SWAP-LIN", 
            "side": "BUY", 
            "quantity": "0.007", 
            "price": "40001.0"
        }, 
        {
            "clientOrderId": "161224973777800", 
            "marketCode": "BTC-USD-SWAP-LIN", 
            "side": "SELL", 
            "quantity": "0.002", 
            "price": "40003.0"
        }, 
        {
            "clientOrderId": "161224973777900", 
            "marketCode": "BTC-USD-SWAP-LIN", 
            "side": "SELL", 
            "quantity": "0.003", 
            "price": "40004.0"
        }
    ]
}
```

> **RESPONSE**

```json
{
    "accountId": "495", 
    "event": "modifyOrder", 
    "timestamp": "1614331651243", 
    "data": [
        {
            "success": "false", 
            "timestamp": "1614331651174", 
            "code": "40032", 
            "message": "Invalid request data", 
            "clientOrderId": "161224973777800", 
            "price": "40003.0", 
            "quantity": "0.002", 
            "side": "SELL", 
            "marketCode": "BTC-USD-SWAP-LIN"
        }, 
        {
            "success": "false", 
            "timestamp": "1614331651174", 
            "code": "40032", 
            "message": "Invalid request data", 
            "clientOrderId": "161224973777900", 
            "price": "40004.0", 
            "quantity": "0.003", 
            "side": "SELL", 
            "marketCode": "BTC-USD-SWAP-LIN"
        }, 
        {
            "success": "true", 
            "timestamp": "1614331651196", 
            "clientOrderId": "1614330009059", 
            "orderId": "304369975621712263", 
            "price": "40001.0", 
            "quantity": "0.007", 
            "side": "BUY", 
            "status": "OPEN", 
            "marketCode": "BTC-USD-SWAP-LIN", 
            "timeInForce": "GTC", 
            "notice": "OrderOpened", 
            "orderType": "LIMIT", 
            "isTriggered": "false"
        }
    ]
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


### DELETE `/v2/orders/cancel`

> **Request**

```json
DELETE /v2/orders/cancel

{
    "recvWindow": 200000, 
    "responseType": "FULL", 
    "timestamp": 1615454880374, 
    "orders": [
        {
            "marketCode": "BTC-USD-SWAP-LIN", 
            "orderId": "304384250571714215", 
            "clientOrderId": "1615453494726"
        }, 
        {
            "marketCode": "BTC-USD-SWAP-LIN", 
            "clientOrderId": "1612249737724"
        }
    ]
}
```

> **RESPONSE**

```json
{
    "accountId": "495", 
    "event": "cancelOrder", 
    "timestamp": "1615454881391", 
    "data": [
        {
            "success": "true", 
            "timestamp": "1615454881383", 
            "clientOrderId": "1615453494726", 
            "orderId": "304384250571714215", 
            "price": "55006.0", 
            "quantity": "0.006", 
            "side": "SELL", 
            "status": "CANCELED_BY_USER", 
            "marketCode": "BTC-USD-SWAP-LIN", 
            "timeInForce": "GTC", 
            "remainQuantity": "0.006", 
            "notice": "OrderClosed", 
            "orderType": "LIMIT", 
            "isTriggered": "false"
        }, 
        {
            "success": "false", 
            "timestamp": "1615454881433", 
            "code": "40035", 
            "message": "Open order not found with id", 
            "clientOrderId": "1612249737724", 
            "marketCode": "BTC-USD-SWAP-LIN"
        }
    ]
}
```

Cancel orders.

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
recvWindow | LONG | NO | |
timestamp | LONG | NO | |
responseType | STRING | YES | `FULL` or `ACK` |
orders | LIST | YES | |
marketCode | STRING | YES | |
orderId | STRING | Either one of orderId or clientOrderId is required | |
clientOrderId | STRING | Either one of orderId or clientOrderId is required | |

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


### POST `/v2/mint`

Mint.

> **Request**

```json
POST /v2/mint

{

    "asset": "flexUSD",
    "quantity": 1000

}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event":"mint",
    "timestamp":"1620964317199",
    "accountId":"1532",
    "data":{
        "asset":"flexUSD",
        "quantity":"10"
    }
}
```

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING/DECIMAL | YES | Quantity of the asset |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
asset | STRING | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING | Quantity of the asset |


### GET /v2/mint/{asset}

Get mint history by asset and sorted by time in descending order.

> **Request**

```json
GET /v2/mint/{asset}?limit={limit}&startTime={startTime}&endTime={endTime}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event":"mintHistory",
    "timestamp":"1620964764692",
    "accountId":"1570",
    "data":[
        {
            "asset":"flexETH",
            "quantity":"0.100000000",
            "mintedAt":"1619779905495"
        },
        {
            "asset":"flexETH",
            "quantity":"97.800000000",
            "mintedAt":"1619779812468"
        },
        {
            "asset":"flexETH",
            "quantity":"0.100000000",
            "mintedAt":"1619779696705"
        },
        ...
    ]
}
```

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
limit | STRING | NO | max `100`, default `100`|
startTime | STRING | NO | Millisecond timestamp, e.g. `1620977300`, default `0` |
endTime | STRING | NO | Millisecond timestamp, e.g `1620977300`, default time now |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
asset | STRING | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING | Quantity of the asset |
mintedAt | STRING | Minted time in millisecond timestamp |


### POST `/v2/redeem`

Redeem.

> **Request**

```json
POST /v2/redeem

{

    "asset": "flexUSD",
    "quantity": 1000,
    "type": "Normal"

}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event":"redeem",
    "timestamp":"1620964351508",
    "accountId":"1532",
    "data":{
        "asset":"flexUSD",
        "quantity":"10",
        "redeemAt":"1620964800000",
        "type":"NORMAL"
    }
}
```

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


### GET /v2/redeem/{asset}

Get redemption history by asset and sorted by time in descending order.

> **Request**

```json
GET /v2/redeem/{asset}?limit={limit}&startTime={startTime}&endTime={endTime}
```

> **SUCCESSFUL RESPONSE**

```json
{
  "event":"redemptionHistory",
  "timestamp":"1620964856842",
  "accountId":"1570",
  "data":[
    {
      "asset":"ETH",
      "quantity":"0.001000000",
      "requestedAt":"1619788358578",
      "redeemedAt":"1619788860219"
    },
    {
      "asset":"ETH",
      "quantity":"0.001000000",
      "requestedAt":"1619788328760",
      "redeemedAt":"1619788328963"
    },
    ...
  ]
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
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

### POST `/v2/borrow`

Borrow.

> **Request**

```json
POST /v2/borrow

{
    "borrowAsset": "USD",
    "collateralAsset": "BTC",
    "collateralAmount": "0.01"
}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "borrow",
    "timestamp": "1626433827430",
    "accountId": "3123",
    "data": {
        "borrowAsset": "USD",
        "borrowedAmount": "296.21588287",
        "collateralAsset": "BTC",
        "collateralizedAmount": "0.01",
        "nonCollateralizedAmount": "0",
        "rateType": "FLOATING_RATE",
        "status": "COMPLETED",
        "borrowedAt": "1626433827616"
    }
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
borrowAsset | STRING | YES | Borrow asset, can only be `USD` for now |
collateralAsset | STRING | YES | Collateral asset |
collateralAmount | STRING | YES | Collateral amount of the collateral asset |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
borrowAsset | STRING | Borrow asset, can only be `USD` for now |
borrowedAmount | STRING | Borrowed amount of the borrow asset |
collateralAsset | STRING | Collateral asset |
collateralizedAmount | STRING | Collateralized amount of the collateral asset |
nonCollateralizedAmount | STRING | `nonCollateralizedAmount` = `collateralAmount` - `collateralizedAmount` |
rateType | STRING | `FLOATING_RATE` or `FIXED_RATE` |
status | STRING | `COMPLETED` or `PARTIAL` or `CANCELLED` |
borrowedAt | STRING | The time of borrowed at |

### POST `/v2/repay`

Repay.

> **Request**

```json
POST /v2/repay

{
    "repayAsset": "USD",
    "regainAsset": "BTC",
    "regainAmount": "0.001"
}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "repay",
    "timestamp": "1626433876060",
    "accountId": "3123",
    "data": {
        "repayAsset": "USD",
        "repaidAmount": "31.33109616",
        "regainAsset": "BTC",
        "regainedAmount": "0.001",
        "nonRegainedAmount": "0",
        "status": "COMPLETED",
        "repaidAt": "1626433876398"
    }
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
repayAsset | STRING | YES | Repay asset, can only be `USD` for now |
regainAsset | STRING | YES | Regain asset |
regainAmount | STRING | YES | Regain amount of the regain asset |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
repayAsset | STRING | Repay asset, can only be `USD` for now |
repaidAmount | STRING | Repaid amount of the repay asset |
regainAsset | STRING | Regain asset |
regainedAmount | STRING | Already regained amount of the regain asset |
nonRegainedAmount | STRING | `nonRegainedAmount` = `regainAmount` - `regainedAmount` |
status | STRING | `COMPLETED` or `PARTIAL` or `CANCELLED` |
repaidAt | STRING | The time of repaid at |


### GET `v2/borrow/{asset}`

Get borrow history by asset and sorted by time in descending order.

> **Request**

```json
GET v2/borrow/{asset}?limit={limit}&startTime={startTime}&endTime={endTime}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "borrowHistory",
    "timestamp": "1626433904976",
    "accountId": "3123",
    "data": [
        {
            "borrowAsset": "USD",
            "borrowedAmount": "296.2158828",
            "collateralAsset": "BTC",
            "collateralizedAmount": "0.01000000",
            "nonCollateralizedAmount": "0",
            "rateType": "FLOATING_RATE",
            "status": "COMPLETED",
            "borrowedAt": "1626433827613"
        },
        {
            "borrowAsset": "USD",
            "borrowedAmount": "0.0000000",
            "collateralAsset": "BTC",
            "collateralizedAmount": "0.00000000",
            "nonCollateralizedAmount": "1",
            "rateType": "FLOATING_RATE",
            "status": "CANCELED",
            "borrowedAt": "1626432797124"
        },
        ...
    ]
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | Collateral asset name |
limit | STRING | NO | max `100`, default `100`|
startTime | STRING | NO | Millisecond timestamp, e.g. `1620977300`, default `0` |
endTime | STRING | NO | Millisecond timestamp, e.g `1620977300`, default time now |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
borrowAsset | STRING | Borrow asset, can only be `USD` for now |
borrowedAmount | STRING | Borrowed amount of the borrow asset |
collateralAsset | STRING | Collateral asset |
collateralizedAmount | STRING | Collateralized amount of the collateral asset |
nonCollateralizedAmount | STRING | `nonCollateralizedAmount` = `collateralAmount` - `collateralizedAmount` |
rateType | STRING | `FLOATING_RATE` or `FIXED_RATE` |
status | STRING | `COMPLETED` or `PARTIAL` or `CANCELLED` |
borrowedAt | STRING | The time of borrowed at |

### GET `v2/repay/{asset}`

Get repay history by asset and sorted by time in descending order.

> **Request**

```json
GET v2/repay/{asset}?limit={limit}&startTime={startTime}&endTime={endTime}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "repayHistory",
    "timestamp": "1626433953585",
    "accountId": "3123",
    "data": [
        {
            "repayAsset": "USD",
            "repaidAmount": "0",
            "regainAsset": "BTC",
            "regainedAmount": "0",
            "nonRegainedAmount": "0.001",
            "status": "CANCELLED",
            "repaidAt": "1626747403329"
        },
        {
            "repayAsset": "USD",
            "repaidAmount": "31863.01677",
            "regainAsset": "BTC",
            "regainedAmount": "1",
            "nonRegainedAmount": "0",
            "status": "COMPLETED",
            "repaidAt": "1626676776330"
        },
        ...
    ]
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | Regain asset name |
limit | STRING | NO | max `100`, default `100`|
startTime | STRING | NO | Millisecond timestamp, e.g. `1620977300`, default `0` |
endTime | STRING | NO | Millisecond timestamp, e.g `1620977300`, default time now |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
repayAsset | STRING | Repay asset, can only be `USD` for now |
repaidAmount | STRING | Repaid amount of the repay asset |
regainAsset | STRING | Regain asset |
regainedAmount | STRING | Already regained amount of the regain asset |
nonRegainedAmount | STRING | `nonRegainedAmount` = `regainAmount` - `regainedAmount` |
status | STRING | `COMPLETED` or `PARTIAL` or `CANCELLED` |
borrowedAt | STRING | The time of borrowed at |
repaidAt | STRING | The time of repaid at |

### GET `v2/borrowingSummary`

Get borrowing summary.

> **Request**

```json
GET v2/borrowingSummary
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "borrowingSummary",
    "timestamp": "1626433981644",
    "accountId": "3123",
    "data": [
        {
            "borrowAsset": "USD",
            "borrowedAmount": "109031.8252978",
            "collateralAsset": "BTC",
            "collateralizedAmount": "3.65800000",
            "rateType": "FLOATING_RATE",
            "status": "To be repaid"
        },
        ...
    ]
}
```

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
borrowAsset | STRING | Borrow asset, can only be `USD` for now |
borrowedAmount | STRING | Total borrowed amount of the borrow asset |
collateralAsset | STRING | Collateral asset |
collateralizedAmount | STRING | Total collateralized amount of the collateral asset |
rateType | STRING | `FLOATING_RATE` or `FIXED_RATE` |
status | STRING | `To be repaid` |

### POST `/v2/borrow/close`

Close borrow.

> **Request**

```json
POST /v2/borrow/close

{
    "marketCode": "BTC-USD-SWAP-LIN",
    "price": "29633.5",
    "quantity": "1"
}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "closeBorrow",
    "timestamp": "1626434069478",
    "accountId": "3123",
    "data": {
        "success": "true",
        "timestamp": "1626434069461",
        "clientOrderId": "1626434069478",
        "orderId": "1000018496331",
        "price": "29633.5",
        "quantity": "1.0",
        "side": "SELL",
        "status": "CANCELED_BY_FOK",
        "marketCode": "BTC-USD-SWAP-LIN",
        "timeInForce": "FOK",
        "notice": "OrderClosed",
        "orderType": "MARKET",
        "isTriggered": "false"
    }
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
marketCode | STRING | YES | Borrow asset, can only be `USD` for now |
price | STRING | NO | The price to close the borrow, default is the market price |
collateralAmount | STRING | NO | The quantity to close the borrow, default is full position |

Response Fields | Type | Description |
----------------| ---- | ----------- |
accountId | STRING | Account ID |
clientOrderId | STRING | Client order ID which given by the user |
orderId | STRING | Order ID which generated by the server |
price | STRING | Price submitted |
quantity | STRING | Quantity submitted |
side | STRING | Side of the order, `BUY` or `SELL` |
status | STRING | Status of the order |
marketCode | STRING | Market code |
timeInForce | STRING | Time in force |
notice | STRING | `OrderClosed` or `OrderMatched` or `OrderOpend` |
orderType | STRING | Type of the order, `LIMIT` or `STOP` |
isTriggered | STRING | `true`(for stop order) or `false` |


### GET `/v2/funding-payments`

Get funding payments by marketCode and sorted by time in descending order.

> **Request**

```json
GET v2/funding-payments?marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
marketCode | STRING | YES | e.g. `BTC-USD-REPO-LIN` |
limit | LONG | NO | default is `50`, max is `50` |
startTime | LONG | NO | millisecond timestamp, e.g. `1579450778000`, default is `0` |
endTime | LONG | NO | millisecond timestamp, e.g. `1613978625000`, default is time now |

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "fundingPayments",
    "timestamp": "1627626010074",
    "accountId": "276007",
    "data": [
        {
            "marketCode": "BTC-USD-SWAP-LIN",
            "payment": "-122.17530872",
            "rate": "-0.00005",
            "position": "-61.093",
            "markPrice": "39996.5",
            "timestamp": "1627617632190"
        },
        {
            "marketCode": "BTC-USD-SWAP-LIN",
            "payment": "98.71895684",
            "rate": "0.00005",
            "position": "-61.093",
            "markPrice": "32317.6",
            "timestamp": "1627041622046"
        },
        ...
    ]
}
```

Response Fields | Type | Description |
------------------- | ---- | ----------- |
timestamp | STRING | Timestamp of this response |
marketCode | STRING | Market code |
payment | STRING | Funding payment |
rate | STRING | Funding rate |
position | STRING | Position |
markPrice | STRING | Mark price |
timestamp(in the data list) | STRING | Updated time |


##Methods - Public

###GET `/v2/all/markets`


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


###GET `/v2/all/assets`

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

###GET `/v2/publictrades/{marketCode}`

> **Request**

```json
GET/v2/publictrades/{marketCode}?limit={limit}&startTime={startTime}&endTime{endTime}
```

> **RESPONSE**


```json
{
  "event": "publicTrades", 
  "timestamp": "1595636619410", 
  "marketCode": "BTC-USD-SWAP-LIN", 
  "data": [
    {
      "matchId": "160070803925856675", 
      "matchQuantity": "0.100000000", 
      "matchPrice": "9600.000000000", 
      "side": "BUY", 
      "matchTimestamp": "1595585860254"
      },
      ...
  ]
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


###GET `/v2/ticker`

> **Request**

```json
GET/v2/ticker
```

> **RESPONSE**

```json
{
  "event":"ticker",
  "timestamp":"123443563454",
  "data" :
  [
      {
            "marketCode": "BTC-USD-SWAP-LIN",
            "last": "43.259", 
            "markPrice": "11012.80409769",  
            "open24h": "49.375",
            "volume24h": "11295421",
            "currencyVolume24h": "1025.7",                       
            "high24h": "49.488",
            "low24h": "41.649",
            "openInterest": "1726003",
            "lastQty": "1"
      },
      ...
  ]
}
```

Get a list of all of the tickers.

Response Parameters | Type | Description| 
-------------------------- | -----|--------- |
timestamp | STRING | Timestamp of this response|
marketCode | STRING | "BTC-USD-SWAP-LIN",
last| STRING | Last traded price
markPrice| STRING | Mark price
open24h| STRING | Daily opening price
volume24h| STRING | 24 hour volume (USD)
currencyVolume24h| STRING | 24 hour volume (coin)
high24h| STRING | 24 hour high
low24h| STRING | 24 hour low
openInterest| STRING | Current open interest
lastQty| STRING | Last traded quantity


###GET `/v2/delivery/public/funding`

> **Request**

```json
GET /v2/delivery/public/funding
```

> **RESPONSE**

```json
[
    {
        "timestamp": "2020-12-08 04:00:00",
        "instrumentId": "COMP-USD-SWAP-LIN",
        "fundingRate": "0.000000000"
    },
    {
        "timestamp": "2020-12-08 04:00:00",
        "instrumentId": "CRV-USD-SWAP-LIN",
        "fundingRate": "0.000000000"
    },
    ...
]
```
Get funding rate

Request Parameters | Type | Description| 
-------------------------- | -----|--------- |
column | STRING | limit size of results, e.g. 3 |
instrumentId | STRING | e.g. UNI-USD-SWAP-LIN |
startTime | STRING | e.g. 2020-12-08 20:00:00 |

Response Parameters | Type | Description| 
-------------------------- | -----|--------- |
timestamp | STRING | Timestamp of this response |
instrumentId | STRING | "BTC-USD-SWAP-LIN" |
fundingRate| STRING | e.g. 0.000060000 |


###GET `/v2.1/deliver-auction/{instrumentId}`
> **Request**

```json
GET /v2/deliver-auction

GET /v2.1/deliver-auction/<instrumentId>
```

> **RESPONSE**

```json
{
    "event": "deliverAuction",
    "timestamp": "1596620815090",
    "data": [
        {
            "instrumentId": "BTC-USD-SWAP-LIN",
            "auctionTime": "1596620805000",
            "netDeliver": "-5.100000000",
            "estFundingRate": "0.0001"
        },
      ...
    ]
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

### GET `/v2/candles/{marketCode}`

Get historical candles of active and expired markets.

> **Request**

```json
GET /v2/candles/{marketCode}?timeframe={timeframe}&limit={limit}&startTime={startTime}&endTime={endTime}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
marketCode | STRING | YES | When marketCode is expired market like `BTC-USD-201225-LIN`, the startTime and the endTime should be explicitly set in `2020` |
timeframe | STRING | NO | e.g. `60s`, `300s`, `900s`, `1800s`, `3600s`, `7200s`, `14400s`, `86400s`, default `3600s ` |
limit | LONG | NO | max `5000 `, default `500`|
startTime | LONG | NO | Millisecond timestamp, e.g. `1579450778000`, default is `limit` times `timeframe` ago, if the limit is `300` and the timeframe is `3600s` then the default startTime is `time now - 300x3600s`, if the limit is not present and the timeframe is `3600s` then the default startTime is `time now - 500x3600s` |
endTime | LONG | NO | Millisecond timestamp, e.g `1579450778000`, default time now |

> **RESPONSE**

```json
{
    "event": "candles",
    "timestamp": "1616743098781",
    "timeframe": "60s",
    "data": [
        {
            "timestamp": "1616713140000",
            "open": "51706.50000000",
            "high": "51758.50000000",
            "low": "51705.50000000",
            "close": "51754.00000000",
            "volume24h": "0",
            "currencyVolume24h": "0"
        },
        {
            "timestamp": "1616713200000",
            "open": "51755.50000000",
            "high": "51833.00000000",
            "low": "51748.00000000",
            "close": "51815.00000000",
            "volume24h": "0",
            "currencyVolume24h": "0"
        },
        ...
    ]
}
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


### GET `/v2/funding-rates/{marketCode}`

Get funding rates by marketCode and sorted by time in descending order.

> **Request**

```json
GET /v2/funding-rates/{marketCode}?startTime={startTime}&endTime={endTime}&limit={limit}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
marketCode | STRING | YES | e.g. BTC-USD-REPO-LIN , available values: BTC-USD-REPO-LIN, ETH-USD-REPO-LIN |
limit | LONG | NO | default is `50` |
startTime | LONG | NO | millisecond timestamp, e.g. `1579450778000`, default is 7 days ago from time now |
endTime | LONG | NO | millisecond timestamp, e.g. `1613978625000`, default is time now |

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "fundingRates",
    "timestamp": "1611645666834",
    "data": [
        {
            "fundingRate": "0.00008",
            "index": "2.78222423",
            "markPrice": "33937.2848",
            "marketCode": "BTC-USD-REPO-LIN",
            "timestamp": "1606461774458"
        },
        ...
    ]
}
```

Response Fields | Type | Description |
------------------- | ---- | ----------- |
timestamp | STRING | Timestamp of this response |
fundingRate | STRING | |
index | STRING | |
markPrice | STRING | |
marketCode | STRING | |
timestamp(in the data list) | STRING | |


### GET `/v2/depth/{marketCode}/{level}`

Get order book by marketCode and level.

> **Request**

```json
GET /v2/depth/BTC-USD-SWAP-LIN/5 
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "depthL5", 
    "timestamp": "1615457834446", 
    "data": [
        {
            "asks": [
                [
                    54792, 
                    0.001, 
                    0, 
                    0
                ], 
                [
                    54802.5, 
                    0.366, 
                    0, 
                    0
                ], 
                [
                    54803, 
                    0.75, 
                    0, 
                    0
                ], 
                [
                    54806, 
                    1.5, 
                    0, 
                    0
                ], 
                [
                    54830.5, 
                    0.687, 
                    0, 
                    0
                ]
            ], 
            "bids": [
                [
                    54786.5, 
                    0.1, 
                    0, 
                    0
                ], 
                [
                    54754.5, 
                    0.375, 
                    0, 
                    0
                ], 
                [
                    54752, 
                    0.394, 
                    0, 
                    0
                ], 
                [
                    54749.5, 
                    0.001, 
                    0, 
                    0
                ], 
                [
                    54745.5, 
                    0.339, 
                    0, 
                    0
                ]
            ], 
            "marketCode": "BTC-USD-SWAP-LIN", 
            "timestamp": "1615457834388"
        }
    ]
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


### GET `/v2/ping`

Get API service status.

> **Request**

```json
GET /v2/ping
```

> **SUCCESSFUL RESPONSE**

```json
{
    "success": "true"
}
```

Response Fields | Type | Description |
----------------| ---- | ----------- |
sucess | STRING | `"true"` indicates that the API service is OK otherwise it will be failed |


### GET `/v2/flex-protocol/balances/{flexProtocol}`

Get flexAsset balances.

> **Request**

```json
GET /v2/flex-protocol/balances/{flexProtocol}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "flexBalances",
    "timestamp": "1621582973071",
    "tradeType": "LINEAR",
    "flexProtocol": "flexUSD",
    "data": [
        {
            "instrumentId": "BTC",
            "total": "168.515000000",
            "available": "0.000000000",
            "reserved": "168.515",
            "quantityLastUpdated": "1621582933075"
        },
        {
            "instrumentId": "DOT",
            "total": "1923.80",
            "available": "0.00",
            "reserved": "1923.8",
            "quantityLastUpdated": "1621569724731"
        },
        {
            "instrumentId": "REVV",
            "total": "336.0",
            "available": "0.0",
            "reserved": "336.0",
            "quantityLastUpdated": "1621569725061"
        },
        ...
    ]
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


### GET `/v2/flex-protocol/positions/{flexProtocol}`

Get flexAsset positions.

> **Request**

```json
GET /v2/flex-protocol/positions/{flexProtocol}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "flexPositions",
    "timestamp": "1621590427436",
    "flexProtocol": "flexUSD",
    "data": [
        {
            "instrumentId": "BTC-USD-SWAP-LIN",
            "quantity": "-169.048",
            "lastUpdated": "1621590364988",
            "contractValCurrency": "BTC",
            "entryPrice": "40766.3490",
            "positionPnl": "-23506.2934480"
        },
        {
            "instrumentId": "ETH-USD-SWAP-LIN",
            "quantity": "-1279.83",
            "lastUpdated": "1621587180441",
            "contractValCurrency": "ETH",
            "entryPrice": "2798.5100",
            "positionPnl": "53420.104200"
        },
        {
            "instrumentId": "LTC-USD-SWAP-LIN",
            "quantity": "-299.49",
            "lastUpdated": "1621585374591",
            "contractValCurrency": "LTC",
            "entryPrice": "207.9500",
            "positionPnl": "503.143200"
        },
        ...
    ]
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


### GET `/v2/flex-protocol/orders/{flexProtocol}`

Get flexAsset orders.

> **Request**

```json
GET /v2/flex-protocol/orders/{flexProtocol}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "flexOrders",
    "timestamp": "1621590953053",
    "data": [
        {
            "orderId": "1000085820424",
            "marketCode": "COMP-USD-REPO-LIN",
            "clientOrderId": "20",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "0.26",
            "remainingQuantity": "0.26",
            "price": "-0.0002",
            "stopPrice": null,
            "limitPrice": "-0.0002",
            "orderCreated": "1621590951780",
            "lastModified": "1621590951903",
            "lastTradeTimestamp": "1621590951828",
            "timeInForce": "GTC"
        },
        {
            "orderId": "1000085820409",
            "marketCode": "COMP-USD-REPO-LIN",
            "clientOrderId": "5",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "0.21",
            "remainingQuantity": "0.21",
            "price": "-0.00005",
            "stopPrice": null,
            "limitPrice": "-0.00005",
            "orderCreated": "1621590951510",
            "lastModified": "1621590951608",
            "lastTradeTimestamp": "1621590951543",
            "timeInForce": "GTC"
        },
        {
            "orderId": "1000085820408",
            "marketCode": "COMP-USD-REPO-LIN",
            "clientOrderId": "4",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "5.2",
            "remainingQuantity": "5.2",
            "price": "-0.00004",
            "stopPrice": null,
            "limitPrice": "-0.00004",
            "orderCreated": "1621590951381",
            "lastModified": "1621590951394",
            "lastTradeTimestamp": "1621590951392",
            "timeInForce": "GTC"
        },
        ...
    ]
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


### GET `/v2/flex-protocol/trades/{flexProtocol}/{marketCode}`

Get flexAsset trades.

> **Request**

```json
GET /v2/flex-protocol/trades/{flexProtocol}/{marketCode}?limit={limit}&startTime={startTime}&endTime={endTime}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "flexTrades",
    "timestamp": "1621591479201",
    "flexProtocol": "flexBTC",
    "data": [
        {
            "matchQuantity": "0.4",
            "total": "4.000000000",
            "fees": "0.788",
            "side": "BUY",
            "orderMatchType": "MAKER",
            "matchTimestamp": "1621483974496",
            "feeInstrumentId": "USD",
            "orderId": "1000009522024",
            "clientOrderId": "1621483945643",
            "marketCode": "BTC-USD-SWAP-LIN",
            "matchPrice": "39400",
            "matchId": "2001011000000"
        },
        {
            "matchQuantity": "0.4",
            "total": "4.400000000",
            "fees": "0.788",
            "side": "BUY",
            "orderMatchType": "MAKER",
            "matchTimestamp": "1621483973636",
            "feeInstrumentId": "USD",
            "orderId": "1000009522024",
            "clientOrderId": "1621483945643",
            "marketCode": "BTC-USD-SWAP-LIN",
            "matchPrice": "39400",
            "matchId": "2001011000000"
        },
        {
            "matchQuantity": "0.4",
            "total": "4.800000000",
            "fees": "0.788",
            "side": "BUY",
            "orderMatchType": "MAKER",
            "matchTimestamp": "1621483973476",
            "feeInstrumentId": "USD",
            "orderId": "1000009522024",
            "clientOrderId": "1621483945643",
            "marketCode": "BTC-USD-SWAP-LIN",
            "matchPrice": "39400",
            "matchId": "2001011000000"
        },
        ...
    ]
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


### GET `/v2/flex-protocol/delivery/orders/{flexProtocol}`

Get flexAsset delivery orders.

> **Request**

```json
GET /v2/flex-protocol/delivery/orders/{flexProtocol}?limit={limit}&startTime={startTime}&endTime={endTime}
```

> **SUCCESSFUL RESPONSE**

```json
{
    "event": "flexDeliveryOrders",
    "timestamp": "1621592173494",
    "flexProtocol": "flexBTC",
    "data": [
        {
            "timestamp": "1621411380000",
            "instrumentId": "BTC-USD-SWAP-LIN",
            "status": "DELIVERED",
            "quantity": null,
            "deliverPrice": "40225.4",
            "transferAsset": "BTC",
            "transferQty": "0.001",
            "instrumentIdDeliver": "USD",
            "deliverQty": "40.2254",
            "deliverOrderId": "659754125496582148",
            "clientOrderId": null
        },
        {
            "timestamp": "1621411378000",
            "instrumentId": "BTC-USD-SWAP-LIN",
            "status": "DELIVERED",
            "quantity": null,
            "deliverPrice": "40217.8",
            "transferAsset": "BTC",
            "transferQty": "0.001",
            "instrumentIdDeliver": "USD",
            "deliverQty": "40.2178",
            "deliverOrderId": "659754119608827908",
            "clientOrderId": null
        },
        {
            "timestamp": "1621411376000",
            "instrumentId": "BTC-USD-SWAP-LIN",
            "status": "DELIVERED",
            "quantity": null,
            "deliverPrice": "40226.5",
            "transferAsset": "BTC",
            "transferQty": "0.001",
            "instrumentIdDeliver": "USD",
            "deliverQty": "40.2265",
            "deliverOrderId": "659754113236107267",
            "clientOrderId": null
        },
        ...
    ]
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
flexProtocol | STRING | YES | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
limit | LONG | NO | max `100`, default `100`, max `500` |
startTime | LONG | NO | e.g. `1579450778000`, default `0` |
endTime | LONG | NO | e.g. `1613978625000`, default time now |


Response Fields | Type | Description |
----------------| ---- | ----------- |
flexProtocol | STRING | Available values `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
instrumentId | STRING | Perpetual swap market code |
status | STRING | Request status |
quantity | Null Type | null |
deliverPrice | STRING |  Mark price at delivery |
transferAsset | STRING | Asset being sent |
transferQty | STRING | Quantity being sent |
instrumentIdDeliver | STRING | Asset being received: long position is `coin`, short position is `USD` |
deliverQty | STRING | Quantity of the received asset |
deliverOrderId | STRING | Order id |
clientOrderId | Null Type| null |
