# REST API

**TEST** site

* `https://v2stgapi.coinflex.com`

**LIVE** site

* `https://v2api.coinflex.com/v2`

For clients who do not wish to take advantage of CoinFLEX's native WebSocket API, CoinFLEX offers a RESTful API that implements much of the same functionality.

##Authentication 

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
ts = datetime.datetime.utcnow().isoformat()
nonce = 123

if body:
    path = '/v2/positions?' + body
    msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, '/v2/positions', body)
else:
    path = '/v2/positions'
    msg_string = '{}\n{}\n{}\n{}\n{}\n'.format(ts, nonce, 'GET', rest_path, '/v2/positions')

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
Verb | Yes| 'GET' | 
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

###GET `/v2/balances`

> **Request**

```json
GET/v2/balances
```

> **Response**

```json
{
    "event": "balances",
    "accountId": "<Your Account ID>",
    "timestamp": "1593627419000",
    "tradeType": "LINEAR"|"INVERSE",
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
        "total": "1585.890"              
        "available": "325.890",         
        "reserved": "1260",
        "quantityLastUpdated": "1593627415123"
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


###GET `/v2/balances/<instrumentId>`

>**Request**

```json
GET/v2/balances/BTC
```


> **RESPONSE**

```json
{
    "event": "balancesById",
    "accountId": "<Your Account ID>",
    "timestamp": "1593627415293",
    "tradeType": "LINEAR"|"INVERSE",
    "data":[
    {   
        "instrumentId": "BTC",
        "total": "4468.823"              
        "available": "4468.823",        
        "reserved": "0",
        "quantityLastUpdated": "1593627415001"
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

###GET  `/v2/positions`

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
contractValCurrency |STRING|Contract valuation currency|
entryPrice|STRING|Average entry price|
positionPnl|STRING|Postion profit and lost|
estLiquidationPx|STRING||
estLiqPrice|STRING|Estimated liquidation price|


###GET  `/v2/positions/<instrumentId>`

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


###GET `/v2/trades/{marketCode}`

> **Request**

```json
GET/v2/trades/{marketCode}?limit={limit}&startTime={startTime}&endTime{endTime}
```

> **RESPONSE**


```json
{
  "event": "trades", 
  "timestamp": "1595635101845", 
  "accountId": "<Your Account ID>", 
  "data": [
    {
    "matchId": "160067484555913077", 
    "matchTimestamp": "1595514663626", 
    "marketCode": "BTC-USD-SWAP-LIN", 
    "matchQuantity": "0.010000000", 
    "matchPrice": "9680.000000000", 
    "total": "96.800000000000000000", 
    "orderMatchType": "TAKER", 
    "fees": "0.009600000", 
    "feeInstrumentId": "FLEX", 
    "orderId": "160067484555913076", 
    "side": "SELL", 
    "clientOrderId": "123"
   },
...
]
}
 
```

Requires authentication. Get most recent trades of current user.

Request Parameters | Type | Required |Description| 
-------------------------- | -----|--------- | -------------|
marketCode| STRING | YES |  | 
limit| LONG | NO | Default 500, max 1000 | 
startTime| LONG | NO |  | 
endTime| LONG | NO |  | 

Response Parameters |Type | Description| 
-------------------------- | -----|--------- |
accountId | STRING    | Account ID|
timestamp | STRING    | Timestamp of this response|
matchId   | STRING    | Match ID          |
matchTimestamp   | STRING    | Order Matched timestamp          |
marketCode   | STRING    | Market code          |
matchQuantity   | STRING    | Match quantity          |
matchPrice   | STRING    | Match price          |
total   | STRING    | Total price          |
side   | STRING    |  Side of the match         |
orderMatchType   | STRING    | Define taker or maker          |
fees   | STRING    |  fees         |
feeInstrumentId   | STRING    |   Instrument ID of the fees        |
orderId   | STRING    | Order ID generated by the server           |
clientOrderID   | STRING    | Client order ID generated by the user           |

###GET  `/v2/orders`

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
orderId   | STRING    | Order ID which generated by the server      |
marketCode| STRING    | Market code          |
clientOrderId| STRING | Client order ID which given by the user          |
side      | STRING    | Side of the order          |
orderType | STRING    | Type of the order          |
quantity  | STRING    | Quantity submitted          |
remainQuantity|STRING | Remainning quantity          |
price     | STRING    | Price submitted          |
stopPrice | STRING    | Stop price for the stop order           |
limitPrice| STRING    | Limit price for the stop limit order          |
orderCreated| STRING  |Timestamp when order was created|
lastModified| STRING  |Timestamp when order was last mordified|
lastTradeTimestamp| STRING| Timestamp when order was last traded|
timeInForce | STRING  | Time in force          |


###DELETE `/v2/cancel/orders`

> **Request**

```json
DELETE/v2/cancel/orders 
```

> **RESPONSE**


```json
{
    "event": "orders",
    "accountId":"<AccountID>",
    "timestamp":"1594412077100",
    "data":[
    "msg": "All open orders for the account have been queued for cancellation"
    ]
}
 
```

Requires authentication. Cancel all open orders.

###DELETE `/v2/cancel/orders/{marketCode}`

> **Request**

```json
DELETE/v2/cancel/orders/BTC-USD 
```

> **RESPONSE**


```json
{
    "event": "orders",
    "accountId":"<AccountID>",
    "marketCode":"BTC-USD",
    "timestamp":"1594412077100",
    "data":[
    "msg": "All open orders for the specified market have been queued for cancellation"
    ]
}
 
```

Requires authentication. Cancel all open orders for a specific market code.


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
