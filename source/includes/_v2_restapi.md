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
              "quantity": "0.94",
              "lastUpdated": "1592486212218",
              "contractValCurrency": "BTC",
              "entryPrice": "7800.00",       
              "positionPnl": "200.3"              
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
              "quantity": "0.94",
              "lastUpdated": "1592486212218",
              "contractValCurrency": "BTC",
              "entryPrice": "7800.00",       
              "positionPnl": "200.3"              
            },
            ...
          ]
}
```

Returns all the positions of the account connected to the API key initiating the request. 

<sub>**Response Parameters**</sub> 

Parameters |Type | Description| 
-------------------------- | -----|--------- |
event | STRING | `positions`
timestamp | INTEGER | Millisecond timestamp
accountId | STRING    | Account ID
data | LIST of dictionaries |
\>instrumentId | STRING | Contract symbol, e.g. 'BTC-USD-SWAP-LIN' |
\>quantity | STRING | Quantity of position, e.g. '0.94' |
\>lastUpdated| STRING| Timestamp when position was last updated|
\>contractValCurrency |STRING|Contract valuation currency|
\>entryPrice|STRING|Average entry price|
\>positionPnl|STRING|Postion profit and lost|


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
              "instrumentId": "FLEX-USD-SWAP-LIN",
              "quantity": "0.94",
              "lastUpdated": "1592486212218",
              "contractValCurrency": "FLEX",
              "entryPrice": "1.1",       
              "positionPnl": "200.3"              
          } ]
}
```

```python
{
  "event": "positionsById",
  "timestamp": 1593617005438,
  "accountId":"<Your Account ID>",
  "data": [ {
              "instrumentId": "FLEX-USD-SWAP-LIN",
              "quantity": "0.94",
              "lastUpdated": "1592486212218",
              "contractValCurrency": "FLEX",
              "entryPrice": "1.1",       
              "positionPnl": "200.3"              
          } ]
}
```

Returns the specified instrument ID position of the account connected to the API key initiating the request.

<sub>**Response Parameters**</sub> 

Parameters |Type | Description| 
-------------------------- | -----|--------- |
event | STRING | `positionsById`
timestamp | INTEGER | Millisecond timestamp
accountId | STRING    | Account ID
data | LIST of dictionaries |
\>instrumentId | STRING | Contract symbol, e.g. 'FLEX-USD-SWAP-LIN' |
\>quantity | STRING | Quantity of position, e.g. '0.94' |
\>lastUpdated| STRING| Timestamp when position was last updated|
\>contractValCurrency |STRING|Contract valuation currency|
\>entryPrice|STRING|Average entry price|
\>positionPnl|STRING|Postion profit and lost|

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
\>matchId   | STRING    | Match ID          |
\>matchTimestamp   | STRING    | Order Matched timestamp          |
\>marketCode   | STRING    | Market code          |
\>matchQuantity   | STRING    | Match quantity          |
\>matchPrice   | STRING    | Match price          |
\>total   | STRING    | Total price          |
\>side   | STRING    |  Side of the match         |
\>orderMatchType   | STRING    | `TAKER` or `MAKER` |
\>fees   | STRING    |  Fees    |
\>feeInstrumentId   | STRING    |   Instrument ID of the fees        |
\>orderId   | STRING    |	Unique order ID from the exchange          |
\>clientOrderID   | STRING    | Client assigned ID to help manage and identify orders  |

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
\>orderId | STRING | Unique order ID from the exchange |
\>marketCode| STRING | Market code |
\>clientOrderId| STRING | Client assigned ID to help manage and identify orders |
\>side | STRING | `BUY` or `SELL` |
\>orderType | STRING | `LIMIT` or `STOP` |
\>quantity  | STRING | Quantity submitted |
\>remainingQuantity|STRING | Remainning quantity |
\>price | STRING | Price submitted |
\>stopPrice | STRING | Stop price for the stop order |
\>limitPrice| STRING | Limit price for the stop limit order |
\>orderCreated| INTEGER | Timestamp when order was created |
\>lastModified| INTEGER | Timestamp when order was last mordified |
\>lastTradeTimestamp| INTEGER | Timestamp when order was last traded |
\>timeInForce | STRING | Time in force |


### GET `/v2.1/orders` PENDING

> **Request**

```json
GET /v2.1/orders?marketCode={marketCode}&orderId={orderId}&clientOrderId={clientOrderId}&limit={limit}&startTime={startTime}&endTime={endTime}

{
  "marketCode": "BTC-USD-SWAP-LIN",
  "orderId": "123456789",
  "clientOrderId": "987654321",
  "limit": "3",
  "startTime": "2020-12-08 20:00:00",
  "endTime": "2020-12-09 20:00:00"
}
```

> **Response**

```json
{
    "accountId": "4499257",
    "event": "orders",
    "timestamp": "1611645666834",
    "data": [
        {
            "status": "OrderOpened",
            "orderId": 304330302278421284,
            "clientOrderId": 1611727170448,
            "marketCode": "BTC-USD-SWAP-LIN",
            "side": "sell",
            "orderType": "LIMIT",
            "price": "33108",
            "lastTradedPrice": null,
            "averageFillPrice": null,
            "stopPrice": null,
            "limitPrice": null,
            "quantity": "1",
            "matchQuantity": null,
            "remainQuantity": "1",
            "orderMatchType": null,
            "matchId": null,
            "leg1Price": null,
            "leg2Price": null,
            "fees": null,
            "feeInstrumentId": null,
            "timeInForce": "MAKER_ONLY",
            "isTriggered": null,
            "timestamp": "1611727170650"
        },
        {
            "status": "OrderMatched",
            "orderId": 5204263117330811401,
            "clientOrderId": 1611644738148,
            "marketCode": "BTC-Rate-Jan21",
            "side": "sell",
            "orderType": "LIMIT",
            "price": "472",
            "lastTradedPrice": "472",
            "averageFillPrice": "472",
            "stopPrice": null,
            "limitPrice": null,
            "quantity": "3",
            "matchQuantity": "0.5",
            "remainQuantity": "2.5",
            "orderMatchType": "MAKER",
            "matchId": [
                5204263117330811403,
                5204263117330811405,
                5204263117330811407
            ],
            "leg1Price": null,
            "leg2Price": null,
            "fees": "-0.0028391",
            "feeInstrumentId": null,
            "timeInForce": "GTC",
            "isTriggered": false,
            "timestamp": "1611645088765"
        },
        ...
    ]
}
```

Returns all the open orders of the account connected to the API key initiating the request.

<sub>**Request Parameters**</sub> 

Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
marketCode | STRING | NO | |
orderId | Integer | NO | |
clientId | Integer | NO | |
limit | STRING | NO | should equal to or less than `1000`, default is `1000` |
startTime | STRING | NO | e.g. `2020-02-01 20:00:00`, default is 24 hours ago from time now |
endTime | STRING | NO | e.g. `2020-02-02 20:00:00`, default is time now |

<sub>**Response Parameters**</sub> 

Parameter | Type | Description |
------------------- | ---- | ----------- |
accountId | STRING | Account ID |
timestamp | STRING | Timestamp of this response |
status | STRING | Status of the order |
orderId | INTEGER | Order ID which generated by the server | clientOrderId | INTEGER | Client order ID which given by the user | marketCode | STRING | Market code |
side | STRING | Side of the order |
orderType | STRING | Type of the order |
price | STRING | Price submitted |
lastTradedPrice | STRING | Price when order was last traded |
averageFillPrice | STRING | Price when order was last traded |
stopPrice | STRING | Stop price for the stop order |
limitPrice | STRING | Limit price for the stop limit order |
quantity | STRING | Quantity submitted |
matchQuantity | STRING | Match quantity |
remainQuantity | STRING | Remainning quantity |
orderMatchType | STRING | `MAKER` or `TAKER` |
matchId | ARRAY | Exchange match ID |
leg1Price | STRING | |
leg2Price | STRING | |
fees | STRING | Amount of fees paid from this match ID |
feeInstrumentId | STRING | Instrument ID of fees paid from this match ID |
timeInForce | STRING | Time in force |
isTriggered | BOOLEAN | |
timestamp(in data) | STRING | |


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
\>timestamp | STRING | Millisecond timestamp of the delivery action
\>instrumentId | STRING | Perpetual swap market code
\>status | STRING | Request status
\>quantity | Null Type| null
\>deliverPrice | STRING|  Mark price at delivery
\>transferAsset | STRING | Asset being sent
\>transferQty | STRING | Quantity being sent
\>instrumentIdDeliver | STRING |Asset being received: long position = coin, short position = USD
\>deliverQty | STRING |  Quantity of the received asset
\>deliverOrderId | STRING | Order id
\>clientOrderId | Null Type|  null


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
    "timestamp": "1593617005438",
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
            "listingDate": "-2208988800000",
            "endDate": null,
            "marginCurrency": null,
            "contractValCurrency": "BTC",
            "upperPriceBound": "11000.00",
            "lowerPriceBound": "9000.00",
            "marketPrice": "10000.00"
        },
        ...
    ]
}
```
Get a list of all available markets on CoinFlex.

Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
timestamp | STRING    | Timestamp of this response|
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
GET/v2.1/deliver-auction
GET/v2.1/deliver-auction/<instrumentId>
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

### GET `/v2/candles`

Get candlestick data for the current candle.

> **Request**

```json
GET /v2/candles
```
> **Request body**

```json
{
    "marketCode": "BTC-USD-SWAP-LIN",
    "timeframe": "1m",
    "limit": 1000
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
marketCode | STRING | YES | |
timeframe | STRING | YES | e.g. 1m, 5m, 15m, 30m, 1h, 4h, 1d, 1w |
limit | INTEGER | NO | default value: 500 |

> **RESPONSE**

```json
{
    "marketCode": "BTC-USD-SWAP-LIN",
    "timeframe": "1m",
    "event": "candle1m",
    "timestamp": "1611718910850",
    "data": [
        [
            "1604301120000",
            "11744",
            "11744",
            "11744",
            "11744",
            "11814.464",
            "1.006"
        ],
        ...
    ]
}
```

Fields |Type | Description|
-------------------------- | -----|--------- |
candle | LIST of strings  | <ul><li>timestamp</li><li>open</li><li>high</li><li>low</li><li>close</li><li>volume in counter currency</li><li>volume in base currency</li></ul>


### GET `/v2/funding-rates/{marketCode}`

Get funding rates by marketCode.

> **Request**

```json
GET /v2.1/funding-rates/{marketCode}
```

> **Request body**

```json
{
    "limit": 3,
    "startTime": 1579450778000,
    "endTime": 1613978625000
}
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
            "marketCode": "BTC-USD-REPO-LIN",
            "marketPrice": "33937.2848",
            "index": "2.78222423",
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
marketCode | STRING | |
marketPrice | STRING | |
index | STRING | |
timestamp(in the data list) | STRING | |
