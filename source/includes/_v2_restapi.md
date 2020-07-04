# REST API

**TEST** site

* `https://v2stgapi.coinflex.com`

**LIVE** site

* 'COMING SOON'

For clients who do not wish to take advantage of CoinFLEX's native WebSocket API, CoinFLEX offers a RESTful API that implements much of the same functionality.

##Authentication 

Public market data methods do not require authentication, however private methods require a *Signature* to be sent in the header of the request.  These private REST methods  use HMAC SHA256 signatures. 

The HMAC SHA256 signature is a keyed HMAC SHA256 operation using a clients API Secret as the key and a message string as the value for the HMAC operation.  

The message string is constructed by the following formula:-

`msgString = Timestamp + "\n" + Nonce + "\n" + Verb + "\n" + URL + "\n" + Path +"\n" + Body`

Component | Required | Example | Description| 
-------------------------- |--------- |------- |------- | 
Timestamp | Yes | 2020-04-30T15:20:30 | YYYY-MM-DDThh:mm:ss
Nonce | Yes | 123 | User generated
Verb | Yes| 'GET' | 
Path | Yes | 'v2stgapi.coinflex.com' |
Method | Yes | '/v2/positions | Available REST methods: <li>`V2/positions`</li><li>`V2/orders`<li><li>`V2/balances`<li>
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

> **Request**

```python
import requests
import hmac
import base64
import hashlib
import datetime
from urllib.parse import urlencode

rest_url = 'https://v2stgapi.coinflex.com'
rest_path = 'v2stgapi.coinflex.com'

api_key = API-KEY
api_secret = API-SECRET

body = urlencode({'key1': 'value1', 'key2': 'value2'})

if body:
    path = '/v2/positions?' + body
else:
    path = '/v2/positions'

ts = datetime.datetime.utcnow().isoformat()
nonce = 123

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, '/v2/positions', body)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
print(resp.json())
```

##GET `/v2/balances`

> **Request**

```json
GET/v2/balances
```

> **Response**

```json
{
    "event": "balances",
    "accountId":"<Your Account ID>",
    "timestamp":"1593627419000",
    "tradeType":"LINEAR"|"INVERSE",
    "data":[
    {   
        "instrumentId": "BTC",
        "total":"4468.823"              
        "available": "4468.823",        
        "reserved": "0",
        "quantityLastUpdated":"1593627415234"
    },
    .....
    {
        "instrumentId": "EOS",
        "total":"1585.890"              
        "available": "325.890",         
        "reserved": "1260",
        "quantityLastUpdated":"1593627415123"
    }
    ]
}
```

Requires authentication. GET coin balanaces in your account. 

Reponse Parameter |Type | Description| 
-------------------------- | -----|--------- |
accountId | STRING    | Account ID|
timestamp | STRING    | Timestamp of this response|
tradeType | STRING    | Define this account is trading linear or inverse derivatives|
instrumentId | STRING |Token symbol, e.g. 'BTC' |
total| STRING| Total balance|
available |STRING|Available balance|
reserved|STRING|Reserved balance (unavailable)|
quantityLastUpdated|STRING|Timestamp when was balance last updated|


##GET `/v2/balances/<instrumentId>`


>**Request**

```json
GET/v2/balances/BTC
```


> **RESPONSE**

```json
{
    "event": "balancesById",
    "accountId":"<Your Account ID>",
    "timestamp":"1593627415293",
    "tradeType":"LINEAR"|"INVERSE",
    "data":[
    {   
        "instrumentId": "BTC",
        "total":"4468.823"              
        "available": "4468.823",        
        "reserved": "0",
        "quantityLastUpdated":"1593627415001"
    }
    ]
}
```

Requires authentication. GET balance for a specific coin.

Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
accountId | STRING    | Account ID|
timestamp | STRING    | Timestamp of this response|
tradeType | STRING    | Define this account is trading linear or inverse derivatives|
instrumentId | STRING |Token symbol, e.g. 'BTC' |
total| STRING| Total balance|
available |STRING|Available balance|
reserved|STRING|Reserved balance (unavailable)|
quantityLastUpdated|STRING|Timestamp when was balance last updated|

##GET  `/v2/positions`


> **Request**

```json
GET/v2/positions
```



> **RESPONSE**

```json
{
    "event": "positions",
    "accountId":"<Your Account ID>",
    "timestamp":"1593627415000",
    “data”: [
        {
            "instrumentId": "BTC-USD-200626-LIN",
            "quantity": "0.94",
            "lastUpdated": "1592486212218",
            "contractValCurrency": "BTC",
            "entryPrice": "7800.00",       
            "positionPnl": "200.3",
            "estLiquidationPx": "6301.1",
            "estLiqPrice": "2342.2",                  

        }
        ...
    ]
}
```

Requires authentication. GET all positions for current user.

Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
accountId | STRING    | Account ID|
timestamp | STRING    | Timestamp of this response|
instrumentId | STRING |Contract symbol, e.g. 'BTC-USD-200626-LIN' |
lastUpdated| STRING| Timestamp when position was last updated|
contractValCurrency |STRING||
entryPrice|STRING|Average entry price|
positionPnl|STRING|Postion profit and lost|
estLiquidationPx|STRING||
estLiqPrice|STRING|Estimated liquidation price|



##GET  `/v2/positions/<instrumentId>`


> **Request**

```json
GET/v2/positions/BTC-USD-200626-LIN
```


> **RESPONSE**

```json
{
    "event": "positionsById",
    "accountId":"<Your Account ID>",
    "timestamp":"1593617005438",
    "data": [
        {
            "instrumentId": "BTC-USD-200626-LIN",
            "quantity": "0.94",
            "lastUpdated": "1592486212218",
            "contractValCurrency": "BTC",
            "entryPrice": "7800.00",       
            "positionPnl": "200.3",
            "estLiquidationPx": "6301.1",
            "estLiqPrice": "2342.2",                  

        }
    ]
}
```
Requires authentication. GET specific positions for current user.

Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
accountId | STRING    | Account ID|
timestamp | STRING    | Timestamp of this response|
instrumentId | STRING |Contract symbol, e.g. 'BTC-USD-200626-LIN' |
lastUpdated| STRING| Timestamp when position was last updated|
contractValCurrency |STRING|Contract valuation currency|
entryPrice|STRING|Average entry price|
positionPnl|STRING|Postion profit and lost|
estLiquidationPx|STRING||
estLiqPrice|STRING|Estimated liquidation price|




##GET `/v2/all/markets`


> **Request**

```json
GET/v2/all/markets`
```

> **RESPONSE**


```json
{
    "event": "markets",
    "timestamp":"1593617005438",
    "data": [
        {
            "marketCode": "BTC-USD",
            "name": "BTC/USD Spot",
            "referencePair": "BTC/USD"
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
            "marketPrice": "10000.00",
        }
        ...
    ]
}
```
Get a list of all available markets on CoinFlex.

Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
timestamp | STRING    | Timestamp of this response|
marketCode| STRING    |                   |
name      | STRING    |                   |
referencePair| STRING |                   |
base      | STRING    |                   |
counter   | STRING    |                   |
type      | STRING    |                   |
tickSize  | STRING    |                   |
qtyIncrement| STRING  |Minimum increamet quantity|
listingDate| STRING   |                   |
endDate    |STRING    |                   |
marginCurrency|STRING |                   |
contractValCurrency| STRING| Contract valuation currency|
upperPriceBound| STRING|                  |
lowerPriceBound| STRING|                  |
marketPrice    | STRING|                  |





##GET `/v2/all/assets`

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
instrumentId| STRING    |                   |
name      | STRING    |                   |
base      | STRING    |                   |
counter   | STRING    |                   |
type      | STRING    |                   |
marginCurrency| STRING |                  |
contractValCurrency| STRING|              |
deliveryDate       | STRING|              |
deliveryInstrument | STRING|              |





##GET `/v2/trades`

> **Request**

```json
GET/v2/trades?limit={limit}&marketCode={marketCode}
```

> **RESPONSE**


```json
{
    "event": "trades",
    "accountId":"<Your Account ID>",
    "timestamp":"1593617005438",
    "data":[ 
    {
        "matchId": "23534545",
        "matchTimestamp": "1588218296439236",
        "marketCode": "BTC-USD"
        "matchQuantity": "1",
        "matchPrice": "9171",
        "total": "9171",
        "side" :"BUY"|”SELL“， 
        "orderMatchType": "TAKER",
        "fees": "2.7513",
        "feeInstrumentId": "USD",
        "orderId": "322805624"
    }
...
]
}
 
```

Requires authentication. Get most recent trades of current user.

Request Parameters |Type | Description| 
-------------------------- | -----|--------- |
marketCode| STRING |      |           |
limit     | STRING |      |Default 500, max 1000|

Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
accountId | STRING    | Account ID|
timestamp | STRING    | Timestamp of this response|
matchId   | STRING    |           |
matchTimestamp   | STRING    |           |
marketCode   | STRING    |           |
matchQuantity   | STRING    |           |
matchPrice   | STRING    |           |
total   | STRING    |           |
side   | STRING    |           |
orderMatchType   | STRING    |           |
fees   | STRING    |           |
feeInstrumentId   | STRING    |           |
orderId   | STRING    |           |


##GET  `/v2/orders`
> **Request**

```json
GET/v2/orders
```


> **RESPONSE**


```json
{
    "event": "orders",
    "accountId":"<Your Account ID>",
    "timestamp":"1593617005438",
    "data":[
       {
          "orderId": "160039151345856176",
          "marketCode": "BTC-USD-200626-LIN",
          "clientOrderId": null|"<clientOrderId>",              
          "side": "BUY",
          "orderType": "LIMIT"|"STOP",
          "quantity": "1.00",
          "remainQuantity": "1.00",
          "price": "1.00"|null,               #for limit order, null for stop order
          "stopPrice": "<stopPrice>"|null,    #for stop order, null for limit order 
          "limitPrice": "《limitPrice>"|null,    #for stop order, null for limit order 
          "orderCreated": "1593617008698",
          "lastModified": "1593617008698",      
          "lastTradeTimestamp": "1593617008698",
          "timeInForce": "GTC"
      }
      ...
  ]
}
 
```
Requires authentication. Get all open orders of current user.


Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
accountId | STRING    | Account ID|
timestamp | STRING    | Timestamp of this response|
orderId   | STRING    |           |
marketCode| STRING    |           |
clientOrderId| STRING |           |
side      | STRING    |           |
orderType | STRING    |           |
quantity  | STRING    |           |
remainQuantity|STRING |           |
price     | STRING    |           |
stopPrice | STRING    |           |
limitPrice| STRING    |           |
orderCreated| STRING  |Timestamp when order was created|
lastModified| STRING  |Timestamp when order was last mordified|
lastTradeTimestamp| STRING| Timestamp when order was last traded|
timeInForce | STRING  |           |





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
