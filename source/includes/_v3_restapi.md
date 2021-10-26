# REST API V3

**TEST** site

* `https://v2stgapi.coinflex.com`

**LIVE** site

* `https://v2api.coinflex.com`

For clients who do not wish to take advantage of CoinFLEX's native WebSocket API, CoinFLEX offers a RESTful API that implements much of the same functionality.

## Authentication

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


# rest_url = 'https://v2api.coinflex.com'
# rest_path = 'v2api.coinflex.com'

rest_url = 'https://v2stgapi.coinflex.com'
rest_path = 'v2stgapi.coinflex.com'

api_key = "API-KEY"
api_secret = "API-SECRET"

ts = datetime.datetime.utcnow().isoformat()
nonce = 123
method = "API-METHOD"

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

The HMAC SHA256 signature is a keyed HMAC SHA256 operation using a client's API Secret as the key and a message string as the value for the HMAC operation.

The message string is constructed as follows:-

`msgString = f'{Timestamp}\n{Nonce}\n{Verb}\n{URL}\n{Path}\n{Body}'`

Component | Required | Example | Description| 
-------------------------- |--------- |------- |------- | 
Timestamp | Yes | 2020-04-30T15:20:30 | YYYY-MM-DDThh:mm:ss
Nonce | Yes | 123 | User generated
Verb | Yes| GET | Uppercase
Path | Yes | v2stgapi.coinflex.com |
Method | Yes | /v3/positions | Available REST methods
Body | No | marketCode=BTC-USD-SWAP-LIN | Optional and dependent on the REST method being called

The constructed message string should look like:-

  `2020-04-30T15:20:30\n
  123\n
  GET\n
  v2stgapi.coinflex.com\n
  /v3/positions\n
  marketCode=BTC-USD-SWAP-LIN`

Note the newline characters after each component in the message string. 
If *Body* is omitted it's treated as an empty string.

Finally, you must use the HMAC SHA256 operation to get the hash value using the API Secret as the key, and the constructed message string as the value for the HMAC operation. Then encode this hash value with BASE-64.  This output becomes the signature for the specified authenticated REST API method. 

The signature must then be included in the header of the REST API call like so:

`header = {'Content-Type': 'application/json', 'AccessKey': API-KEY, 'Timestamp': TIME-STAMP, 'Signature': SIGNATURE, 'Nonce': NONCE}`

## Deposits & Withdrawals


### GET `/v3/deposit-addresses`

Deposit addresses

> **Request**

```
GET /v3/deposit-addresses?asset={asset}&network={network}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "address":"0xD25bCD2DBb6114d3BB29CE946a6356B49911358e"
    }
}
```

<sub>**Request Parameters**</sub> 

Parameter | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | 
network | STRING | YES |

<sub>**Response**</sub> 

Field | Type | Value | Description | 
------------------ | ---- | -------- | ----------- |
address | STRING | 0xD25bCD2DBb6114d3BB29CE946a6356B49911358e | Deposit address
memo | STRING | | Memo (tag) if applicable


### GET `/v3/deposit`

Deposit history

> **Request**

```
GET /v3/deposit?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "flexUSD",
            "network": "SLP",
            "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
            "quantity": "1000.0",
            "id": "651573911056351237",
            "status": "COMPLETED",
            "txId": "38c09755bff75d33304a3cb6ee839fcb78bbb38b6e3e16586f20852cdec4886d",
            "creditedAt": "1617940800000"
        }
    ]
}
```

<sub>**Request Parameters**</sub> 

Parameter | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | NO |  Default all assets |
limit | LONG | NO | Max 200, Default 50 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other|
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other|

<sub>**Response**</sub> 

Field | Type | Value | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | flexUSD | 
network | STRING | SLP |
address | STRING | 0xD25bCD2DBb6114d3BB29CE946a6356B49911358e | Deposit address
memo | STRING | | Memo (tag) if applicable
quantity | STRING | 1000.0 | 
id | STRING | 651573911056351237 | |
status | STRING | COMPLETED |
txId | STRING | 38c09755bff75d33304a3cb6ee839fcb78bbb38b6e3e16586f20852cdec4886d
creditedAt | STRING | 1617940800000 | Millisecond timestamp


### GET `/v3/withdrawal-addresses`

Withdrawal addresses

> **Request**

```
GET /v3/withdrawal-addresses?asset={asset}?network={network}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "FLEX",
            "network": "ERC20",
            "address": "0x047a13c759D9c3254B4548Fc7e784fBeB1B273g39",
            "label": "farming",
            "whitelisted": true
        },
        ...
    ]
}
```

Provides a list of all saved withdrawal addresses along with their respected labels, network, and whitelist status

<sub>**Request Parameters**</sub> 

Parameters | Type | Required | Description | 
-------------- | ---- | -------- | ----------- |
asset | STRING | NO |  Default all assets |
network | STRING | NO | Default all networks |

<sub>**Response**</sub>

Field | Type | Value | Description| 
----- | ---- | ----  | ---------- |
asset | STRING | FLEX
network | STRING | ERC20
address | STRING | 0x047a13c759D9c3254B4548Fc7e784fBeB1B273g39
memo | STRING | | Memo (tag) if applicable
label | STRING | farming | Withdrawal address label
whitelisted |BOOL | true 


### GET `/v3/withdrawal`

Withdrawal history

> **Request**

```
GET /v3/withdrawal?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "flexUSD",
            "network": "SLP",
            "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
            "quantity": "1000.0",
            "fee": "0.000000000",
            "id": "651573911056351237",
            "status": "COMPLETED",
            "txId": "38c09755bff75d33304a3cb6ee839fcb78bbb38b6e3e16586f20852cdec4886d",
            "requestedAt": "1617940800000",
            "completedAt": "16003243243242"
        },
        ...
    ]
}
```

<sub>**Request Parameters**</sub> 

Parameter | Type | Required | Description | 
--------- | ---- | -------- | ----------- |
asset | STRING | NO |  Default all assets |
limit | INT | NO | Max 200, Default 50 |
startTime | INT | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other|
endTime | INT | NO |  Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

<sub>**Response**</sub> 

Field | Type | Value | Description | 
--------- | ---- | -------- | ----------- |
asset | STRING | flexUSD |  |
network | STRING | SLP |  |
address | STRING | simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7 | |
memo | STRING | | Memo (tag) if applicable|
quantity | STRING | 1000.0 |   |
fee | STRING | 0 |
id | STRING | 651573911056351237 | |
status | STRING | COMPLETED | COMPLETED, PROCESSING, PENDING, ON HOLD, CANCELED, or FAILED| 
txId | STRING | 38c09755bff75d33304a3cb6ee839fcb78bbb38b6e3e16586f20852cdec4886d |
requestedAt | STRING | 1617940800000 | Millisecond timestamp
completedAt | STRING | 1617941600000 | Millisecond timestamp


### POST `/v3/withdrawal`

Withdrawal request

> **Request**

```
POST /v3/withdrawal
```
```json
{
    "asset": "flexUSD",
    "network": "SLP",
    "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
    "quantity": "100",
    "externalFee": true,
    "2faType": "GOOGLE",
    "code": "743249"
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "asset": "flexUSD",
        "network": "SLP",
        "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
        "quantity": "1000.0",
        "externalFee": true,
        "fee": "0",
        "status": "PENDING",
        "requestedAt": "1617940800000"
    }
}
```

<sub>**Request Parameters**</sub> 

Parameter |Type |Required| Description| 
-------------------------- | -----|---|--------- |
asset | STRING | YES |
network | STRING | YES |
address | STRING | YES |
memo | STRING | NO |Memo is required for chains that support memo tags
quantity | STRING | YES |
externalFee | BOOL |NO | Default false. If false, then the fee is taken from the quantity
2faType | STRING | NO | GOOGLE, or AUTHY_SECRET, or YUBIKEY
code | STRING | NO | 2fa code if required by the account

<sub>**Response**</sub> 

Field |Type | Value | Description| 
------ | -----|------ |----|
asset | STRING | flexUSD |
network | STRING | SLP |
address | STRING | simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7 |
memo | STRING |  | Memo (tag) if applicable
quantity | STRING | 1000.0 |
externalFee | BOOL | true | Default false. If false, then the fee is taken from the quantity
fee | STRING | 0 |
status | STRING | PENDING |
requestedAt | STRING | 1617940800000 | Millisecond timestamp


### GET `/v3/withdrawal-fee`

Withdrawal fee estimate

> **Request**

```
GET /v3/withdrawal-fee?asset={asset}&network={network}&address={address}&memo={memo}&quantity={quantity}&externalFee={externalFee}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "asset": "flexUSD",
        "network": "SLP",
        "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
        "quantity": "1000.0",
        "externalFee": true,
        "estimatedFee": "0"
    }
}
```

<sub>**Request Parameters**</sub> 

Parameter | Type | Required | Description | 
------------------ | ---- | -------- | --------- |
asset | STRING | YES | |
network | STRING | YES | |
address | STRING | YES | |
memo | STRING | NO | Required only for 2 part addresses (tag or memo)|
quantity | STRING | YES | |
externalFee | BOOL | NO | Default false. If false, then the fee is taken from the quantity|

<sub>**Response**</sub> 

Field | Type | Value | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | flexUSD | |
network | STRING | SLP | |
address | STRING | simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7 | |
memo | STRING |  | Memo (tag) if applicable|
quantity | STRING | 1000 | |
externalFee | BOOL | true | If false, then the fee is taken from the quantity|
estimatedFee | STRING | 0 | 


### POST `/v3/transfer`

Sub-account balance transfer

<aside class="notice">
Transferring funds between sub-accounts is restricted to API keys linked to the main account.
</aside>

> **Request**

```
POST /v3/transfer
```
```json
{
  "asset": "flexUSD",
  "quantity": "1000",
  "fromAccount": "14320",
  "toAccount": "15343"
}
```

> **Successful response format**

```json
{
  "success": true,
  "data": {
    "asset": "flexUSD", 
    "quantity": "1000",
    "fromAccount": "14320",
    "toAccount": "15343",
    "transferredAt": "1635038730480"
  }
}
```

<sub>**Request Parameters**</sub> 

Parameter | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | YES | |
quantity | STRING | YES | |
fromAccount | STRING | YES | |
toAccount | STRING | YES | |

<sub>**Response**</sub> 

Field | Type | Value | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | flexUSD | |
quantity | STRING | 1000 | |
fromAccount | STRING | 14320 | |
toAccount | STRING | 15343 | |
transferredAt | STRING | 1635038730480 | Millisecond timestamp



### GET `/v3/transfer`

Sub-account balance transfer history

> **Request**

```url
GET /v3/transfer?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "flexUSD", 
            "quantity": "1000",
            "fromAccount": "14320",
            "toAccount": "15343",
            "id": "703557273590071299",
            "status": "COMPLETED",
            "transferredAt": "1634779040611"
        },
        ...
    ]
}
```

<aside class="notice">
API keys linked to the main account can get all account transfers, 
while API keys linked to a sub-account can only see transfers where the sub-account is either the "fromAccount" or "toAccount".
</aside>

<sub>**Request Parameters**</sub> 

Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | NO | Default all assets |
limit | LONG | NO | Max 200, default 50 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other|
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other|

<sub>**Response**</sub> 

Field | Type | Value | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | flexUSD | |
quantity | STRING | 1000 | |
fromAccount | STRING | 14320 | |
toAccount | STRING | 15343 | |
id | STRING | 703557273590071299 ||
status | STRING | COMPLETED ||
transferredAt | STRING | 1635038730480 | Millisecond timestamp
