# REST API V3

**TEST** 

* `https://v2stgapi.coinflex.com`

**LIVE** site

* `https://v2api.coinflex.com`

For clients who do not wish to take advantage of CoinFLEX's native WebSocket API, CoinFLEX offers a RESTful API that implements much of the same functionality.

## Error Codes

Code | Description |
---- | ----------- |
429 | Rate limit reached |
10001 | General networking failure |
20001 | Invalid parameter |
30001 | Missing parameter |
40001 | Alert from the server |
50001 | Unknown server error |

## Rate Limits

Each IP is limited to:

* 100 requests per second
* 20 POST v3/orders requests per second
* 2500 requests over 5 minutes

Certain endpoints have extra IP restrictions:

* `s` denotes a second
* Requests limited to `1/s` & `2/10s`
  * Only 1 request is permitted per second and only 2 requests are permitted within 10 seconds
* Request limit `1/10s`
  * The endpoint will block for 10 seconds after an incorrect 2FA code is provided (if the endpoint requires a 2FA code)

Affected APIs:

* [POST /v3/withdrawal](?json#rest-api-v3-deposits-and-withdrawals-post-v3-withdrawal)
* [POST /v3/transfer](?json#rest-api-v3-deposits-and-withdrawals-post-v3-transfer)
* [POST /v3/flexasset/mint](?json#rest-api-v3-flex-assets-post-v3-flexasset-mint)
* [POST /v3/flexasset/redeem](?json#rest-api-v3-flex-assets-post-v3-flexasset-redeem)

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
import json


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
body = json.dumps({'key1': 'value1', 'key2': 'value2'})

if body:
    path = method + '?' + body
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, body)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
# When calling an endpoint that uses body
# resp = requests.post(rest_url + method, data=body, headers=header)
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

## Account & Wallet - Private


### GET `GET v3/account `

Get account information

<aside class="notice">
If the user is calling this endpoint using an API-KEY from a master account then the parameter "subAcc" will allow the user to list which sub-accounts in addition to the default account will be returned. If an API-KEY from a sub-account is used then this parameter has no control.
</aside>

> **Request**

```
GET v3/account?subAcc={subAcc},{subAcc}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "accountId": "21213",
            "accountName": "main",
            "accountType": "LINEAR",
            "marginCurrency": "USD",
            "balances": [
                {
                    "asset": "BTC",
                    "total": "4468.823",
                    "available": "4468.823",
                    "reserved": "0",
                    "updatedAt": "1593627415234"
                },
                {
                    "asset": "FLEX",
                    "total": "1585.890",
                    "available": "325.890",
                    "reserved": "1260",
                    "updatedAt": "1593627415123"
                }
            ],
            "positions": [
                {
                    "marketCode": "BTC-USD-SWAP-LIN",
                    "baseAsset": "BTC",
                    "counterAsset": "USD",
                    "position": "0.94",
                    "entryPrice": "7800.00",
                    "markPrice": "33000.00",
                    "positionPnl": "200.3",
                    "estLiquidationPrice": "12000.05",
                    "updatedAt": "1592486212218"
                }
            ],
            "loans": [
                {
                    "borrowedAsset": "USD",
                    "borrowedAmount": "100000.0",
                    "collateralAsset": "BTC",
                    "marketCode": "BTC-USD-SWAP-LIN",
                    "position": "1.6",
                    "entryPrice": "50000.0",
                    "markPrice": "62500.0",
                    "estLiquidationPrice": "40000.0",
                    "lastUpdatedAt": "1592486212218"
                }
            ],
            "collateral": "1231231",
            "notionalPositionSize": "5000.00",
            "portfolioVarMargin": "500",
            "riskRatio": "20.0000",
            "maintenanceMargin": "1231",
            "marginRatio": "12.3179",
            "liquidating": false,
            "feeTier": "6",
            "createdAt": "1611665624601"
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- |----- | -------- | ----------- |
subAcc | STRING | NO | Sub account, only the default account for the API-Key would be returned if no subcc present |

Response Field | Type | Description |
-------------- | ---- | ----------- |
accountId | STRING | Account ID |
accountName | STRING | Account name |
accountType | STRING | Account type |
marginCurrency | STRING | Margin currency |
balances | LIST of dictionaries | |
asset | STRING | Asset name |
total | STRING | Total balance|
available | STRING | Available balance |
reserved | STRING | Reserved balance |
updatedAt | STRING | Timestamp of updated at |
positions | LIST of dictionaries | Positions |
marketCode | STRING | Market code |
baseAsset | STRING | Base asset |
counterAsset | STRING | Counter asset |
position | STRING | Position size |
entryPrice | STRING | Entry price |
markPrice | STRING | Mark price |
positionPnl | STRING | Position PNL |
estLiquidationPrice | STRING | Estimated liquidation price |
loans | LIST of dictionaries | Loans |
borrowedAsset | STRING | Borrowed asset |
borrowedAmount | STRING | Borrowed amount |
collateralAsset | STRING | Collateral asset |
marketCode | STRING | Market code |
lastUpdatedAt | STRING | Timestamp of last updated at |
collateral | STRING | Collateral |
notionalPositionSize | STRING | Notional position size |
portfolioVarMargin | STRING | Portfolio margin |
riskRatio | STRING | collateralBalance / portfolioVarMargin, Orders are rejected/cancelled if the risk ratio drops below 1 and liquidation occurs if the risk ratio drops below 0.5 |
maintenanceMargin | STRING | Maintenance margin |
marginRatio | STRING | Margin ratio |
liquidating | BOOLEAN | Available values: `true` and `false` |
feeTier | STRING | Fee tier |
createdAt | STRING | Timestamp of created at |


## Deposits & Withdrawals - Private

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

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | YES |
network | STRING | YES |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
address | STRING | Deposit address |
memo | STRING | Memo (tag) if applicable |


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

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Default all assets |
limit | LONG | NO | Default 50, max 200 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | | 
network | STRING | |
address | STRING | Deposit address |
memo | STRING | Memo (tag) if applicable |
quantity | STRING | |
id | STRING | |
status | STRING | |
txId | STRING | |
creditedAt | STRING | Millisecond timestamp |


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
        }
    ]
}
```

Provides a list of all saved withdrawal addresses along with their respected labels, network, and whitelist status

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | NO |  Default all assets |
network | STRING | NO | Default all networks |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | |
network | STRING | |
address | STRING | |
memo | STRING | Memo (tag) if applicable |
label | STRING | Withdrawal address label |
whitelisted | BOOL | |


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
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | NO |  Default all assets |
limit | LONG | NO | Default 50, max 200 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other. This filter applies to "requestedAt"|
endTime | LONG | NO |  Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other. This filter applies to "requestedAt" |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | |
network | STRING | |
address | STRING | |
memo | STRING | Memo (tag) if applicable |
quantity | STRING | |
fee | STRING | |
id | STRING | |
status | STRING | COMPLETED, PROCESSING, PENDING, ON HOLD, CANCELED, or FAILED| 
txId | STRING | |
requestedAt | STRING | Millisecond timestamp |
completedAt | STRING | Millisecond timestamp |


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
    "tfaType": "GOOGLE",
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
Withdrawals may only be initiated by API keys that are linked to the main account and have withdrawals enabled. If the wrong 2fa code is provided the endpoint will block for 10 seconds.

Request Parameter | Type | Required | Description | 
----------------- | ---- |--------- | ----------- |
asset | STRING | YES |
network | STRING | YES |
address | STRING | YES |
memo | STRING | NO |Memo is required for chains that support memo tags |
quantity | STRING | YES |
externalFee | BOOL |YES | If false, then the fee is taken from the quantity |
tfaType | STRING | NO | GOOGLE, or AUTHY_SECRET, or YUBIKEY |
code | STRING | NO | 2fa code if required by the account |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | |
network | STRING | |
address | STRING | |
memo | STRING | | 
quantity | STRING | |
externalFee | BOOL | If false, then the fee is taken from the quantity |
fee | STRING | |
status | STRING | |
requestedAt | STRING | Millisecond timestamp |


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

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | YES | |
network | STRING | YES | |
address | STRING | YES | |
memo | STRING | NO | Required only for 2 part addresses (tag or memo)|
quantity | STRING | YES | |
externalFee | BOOL | NO | Default false. If false, then the fee is taken from the quantity|

Response Field | Type | Description | 
---------------| ---- | ----------- |
asset | STRING | |
network | STRING | |
address | STRING | |
memo | STRING | Memo (tag) if applicable|
quantity | STRING | |
externalFee | BOOL | If false, then the fee is taken from the quantity|
estimatedFee | STRING | |


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

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | YES | |
quantity | STRING | YES | |
fromAccount | STRING | YES | |
toAccount | STRING | YES | |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | |
quantity | STRING | |
fromAccount | STRING | |
toAccount | STRING | |
transferredAt | STRING | Millisecond timestamp |


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
        }
    ]
}
```

<aside class="notice">
API keys linked to the main account can get all account transfers, 
while API keys linked to a sub-account can only see transfers where the sub-account is either the "fromAccount" or "toAccount".
</aside>

Request Parameters | Type | Required | Description | 
------------------ | ---- | -------- | ----------- |
asset | STRING | NO | Default all assets |
limit | LONG | NO | Default 50, max 200 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | |
quantity | STRING | |
fromAccount | STRING | |
toAccount | STRING | |
id | STRING | |
status | STRING | |
transferredAt | STRING | Millisecond timestamp |



## Flex Assets - Private

### POST `/v3/flexasset/mint`

Mint

> **Request**

```
POST /v3/flexasset/mint
```
```json
{
    "asset": "flexUSD",
    "quantity": "100"
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "asset": "flexUSD",
        "quantity": "100"
    }
}
```
<aside class="notice">
Minting is restricted starting 1 minute before an interest payment until 1 minute after the interest payment.
Interest payments occur at 04:00, 12:00, and 20:00 UTC.
</aside>

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
asset | STRING | YES | Asset name, available assets e.g. `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING | YES | Quantity to mint |

Response Field | Type | Description |
-------------- | ---- | ----------- |
asset | STRING |  |
quantity | STRING | |


### POST `/v3/flexasset/redeem`

Redeem

> **Request**

```
POST /v3/flexasset/redeem
```
```json
{
    "asset": "flexUSD",
    "quantity": "100",
    "type": "NORMAL"
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "asset": "flexUSD",
        "quantity": "100",
        "type": "NORMAL",
        "redemptionAt": "1617940800000"
    }
}
```

<aside class="notice">
Redemptions of any type are restricted starting 1 minute before an interest payment until 1 minute after the interest payment.
Interest payments occur at 04:00, 12:00, and 20:00 UTC.
</aside>

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | YES | Asset name, available assets e.g. `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
quantity | STRING | YES | Quantity to redeem |
type | STRING | YES | `NORMAL` queues a redemption until the following interest payment and incurs no fee. `INSTANT` instantly redeems into the underlying asset and charges a fee equal to the sum of the two prior interest payments


Response Field | Type | Description |
-------------- | ---- | ----------- |
asset | STRING |  |
quantity | STRING | |
type | STRING | Available types: `NORMAL`, `INSTANT` |
redemptionAt | STRING | Millisecond timestamp indicating when redemption will take place|


### GET `/v3/flexasset/mint`

Get mint history by asset and sorted by time in descending order.

> **Request**

```
GET /v3/flexasset/mint?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
          "asset": "flexUSD",
          "quantity": "1000.0",
          "mintedAt": "16003243243242"
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
limit | LONG | NO | Default 50, max 200 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Field | Type | Description |
-------------- | ---- | ----------- |
asset | STRING | |
quantity | STRING | |
mintedAt | STRING | |


### GET `/v3/flexasset/redeem`

Get redemption history by asset and sorted by time in descending order.

> **Request**

```
GET /v3/flexasset/redeem?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
          "asset": "flexUSD",
          "quantity": "1000.0",
          "requestedAt": "16003243243242",
          "redeemedAt": "16003243243242"
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
limit | LONG | NO | Default 50, max 200 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other. Here startTime and endTime refer to the "requestedAt" timestamp |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other. Here startTime and endTime refer to the "requestedAt" timestamp |


Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | |
quantity | STRING | |
requestedAt | STRING | Millisecond timestamp indicating when redemption was requested|
redeemedAt | STRING | Millisecond timestamp indicating when the flexAssets were redeemed |


### GET `/v3/flexasset/earned`

> **Request**

```
GET /v3/flexasset/earned?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "flexUSD",
            "snapshotQuantity": "10000",
            "apr": "25",
            "rate": "0.00022831",
            "amount": "2.28310502",
            "paidAt": "1635822660847"
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Asset name, available assets: `flexUSD`, `flexBTC`, `flexETH`, `flexFLEX` |
limit | LONG | NO | Default 50, max 200 |
startTime | LONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | LONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | Asset name |
snapshotQuantity | STRING |
apr | STRING | Annualized APR (%) = rate * 3 * 365 * 100 |
rate | STRING | Period interest rate |
amount | STRING | |
paidAt | STRING | |

## AMM - Private

### POST `/v3/AMM/create`

Create AMM

* Leveraged buy or sell or neutral

> **Request**

```
POST /v3/AMM/create
```

> **Parameters of leveraged buy or sell or neutral**

```json
{
    "leverage": "5",
    "direction": "BUY",
    "marketCode": "BCH-USD-SWAP-LIN",
    "collateralAsset": "BCH",
    "collateralQuantity": "50",
    "minPriceBound": "200",
    "maxPriceBound": "800"
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "hashToken": "CF-BCH-AMM-ABCDE3iy",
        "leverage": "5",
        "direction": "BUY",
        "marketCode": "BCH-USD-SWAP-LIN",
        "collateralAsset": "BCH",    
        "collateralQuantity": "50",                
        "minPriceBound": "200",
        "maxPriceBound": "800"
    }
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
leverage | STRING | YES | String from 1 to 10 |
direction | STRING | YES | Availbale values: `BUY` and `SELL` |
marketcode | STRING | YES | Market code |
collateralAsset | STRING | YES | Collateral asset |
collateralQuantity | STRING | YES | Collateral quantity, minimum notional $200 |
minPriceBound | STRING | YES | Minimum price bound |
maxPriceBound | STRING | YES | Maximum price bound |

Response Field | Type | Description |
-------------- | ---- | ----------- |
hashToken | STRING | Hash token |
leverage | STRING | Leverage, string from 1 to 10 |
direction | STRING | Direction, availbale values: `BUY` and `SELL` |
marketcode | STRING | Market code |
collateralAsset | STRING | Collateral asset |
collateralQuantity | STRING | Collateral quantity, minimum notional $200 |
minPriceBound | STRING | YES | Minimum price bound |
maxPriceBound | STRING | YES | Maximum price bound |

* Unleveraged buy or sell

> **Parameters of unleveraged buy or sell**

```json
{
    "direction": "BUY",
    "marketCode": "BCH-USD-SWAP-LIN",
    "baseQuantity": "250",
    "minPriceBound": "200",
    "maxPriceBound": "800"
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "hashToken": "CF-BCH-AMM-ABCDE3iy",
        "direction": "BUY",
        "marketCode": "BCH-USD-SWAP-LIN",
        "baseAsset": "BCH",
        "baseQuantity": "50",
        "minPriceBound": "200",
        "maxPriceBound": "800"
    }
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
direction | STRING | YES | Availbale values: `BUY` and `SELL` |
marketcode | STRING | YES | |
baseQuantity | STRING | YES when BUY | Quantity of the base asset |
counterQuantity | STRING | YES when SELL | Quantity of the counter asset |
minPriceBound | STRING | YES | Minimum price bound |
maxPriceBound | STRING | YES | Maximum price bound |

Response Field | Type | Description |
-------------- | ---- | ----------- |
hashToken | STRING | Hash token |
direction | STRING | Availbale values: `BUY` and `SELL` |
marketcode | STRING | Market code |
baseAsset | STRING | Base asset |
counterAsset | STRING | Counter asset |
baseQuantity | STRING | YES when BUY | Quantity of the base asset |
counterQuantity | STRING | YES when SELL | Quantity of the counter asset |
minPriceBound | STRING | YES | Minimum price bound |
maxPriceBound | STRING | YES | Maximum price bound |


* Unleveraged neutral

> **Parameters of unleveraged neutral**

```json
{
    "direction": "NEUTRAL",
    "marketCode": "BCH-USD-SWAP-LIN",
    "baseQuantity": "3",
    "counterQuantity": "500",
    "minPriceBound": "200",
    "maxPriceBound": "800"
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "hashToken": "CF-BCH-AMM-ABCDE3iy",
        "direction": "BUY",
        "marketCode": "BCH-USD-SWAP-LIN",
        "baseAsset": "BCH",
        "counterAsset": "USD",
        "baseQuantity": "3",
        "counterQuantity": "500",
        "minPriceBound": "200",
        "maxPriceBound": "800"
    }
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
direction | STRING | YES | Availbale values: `NEUTRAL` |
marketcode | STRING | YES | Market code |
baseQuantity | STRING | YES | Quantity of the base asset |
counterQuantity | STRING | YES | Quantity of the counter asset |
minPriceBound | STRING | YES | Minimum price bound |
maxPriceBound | STRING | YES | Maximum price bound |

Response Field | Type | Description |
-------------- | ---- | ----------- |
hashToken | STRING | Hash token |
direction | STRING | Availbale values: `NEUTRAL` |
marketcode | STRING | Market code |
baseAsset | STRING | Base asset |
counterAsset | STRING | Counter asset |
baseQuantity | STRING | YES when BUY | Quantity of the base asset |
counterQuantity | STRING | YES when SELL | Quantity of the counter asset |
minPriceBound | STRING | YES | Minimum price bound |
maxPriceBound | STRING | YES | Maximum price bound |


### POST `/v3/AMM/redeem`

Redeem AMM

> **Request**

```
POST /v3/AMM/redeem
```

```json
{
    "hashToken": "CF-BCH-AMM-ABCDE3iy",
    "type":"DELIVER"
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "hashToken": "CF-BCH-AMM-ABCDE3iy",
        "type":"DELIVER"
    }
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
hashToken | STRING | YES | Hash token |
type | STRING | YES | Available values: `DELIVER` and `MANUAL` |

Response Field | Type | Description |
-------------- | ---- | ----------- |
hashToken | STRING | Hash token |
type | STRING | Available values: `DELIVER` and `MANUAL` |


### GET `/v3/AMM`

Get AMM information

> **Request**

```
GET /v3/AMM?hashToken={hashToken},{hashToken}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "hashToken": "CF-BCH-AMM-ABCDE3iy",
        "leverage": "3",
        "direction": "BUY",
        "marketCode": "BCH-USD-SWAP-LIN",
        "initialCollateral": {
            "BCH": "123"
        },
        "minPriceBound": "200",
        "maxPriceBound": "800",
        "status": "ENDED",
        "positions": [
            {
                "marketCode": "BTC-USD-SWAP-LIN",
                "baseAsset": "BTC",
                "counterAsset": "USD",
                "position": "0.94",
                "entryPrice": "7800.00",
                "markPrice": "33000.00",
                "positionPnl": "200.3",
                "estLiquidationPrice": "12000.05",
                "lastUpdatedAt": "1592486212218"
            }
        ],
        "balances": [
            {
                "asset": "BTC",
                "total": "4468.823",
                "available": "4468.823",
                "reserved": "0",
                "lastUpdatedAt": "1593627415223",
                "asset": "FLEX",
                "total": "1585.890",
                "available": "325.890",
                "reserved": "1260",
                "lastUpdatedAt": "1593627415123",
                "usdReward": "200",
                "flexReward": "200",
                "interestPaid": "123",
                "apr": "0.1",
                "collateral": "1231231",
                "notionalPositionSize": "5000.00",
                "portfolioVarMargin": "500",
                "riskRatio": "20.0000",
                "maintenanceMargin": "1231",
                "marginRatio": "12.3179",
                "liquidating": "false",
                "feeTier": "6",
                "createdAt": "1623042343252",
                "lastUpdatedAt": "1623142532134"
            }
        ],
        "loans": [
            {
                "borrowedAsset": "USD",
                "borrowedAmount": "100000.0",
                "collateralAsset": "BTC",
                "marketCode": "BTC-USD-SWAP-LIN",
                "position": "1.6",
                "entryPrice": "50000.0",
                "markPrice": "62500.0",
                "estLiquidationPrice": "40000.0",
                "lastUpdatedAt": "1592486212218"
            }
        ],
        "usdReward": "200",
        "flexReward": "200",
        "interestPaid": "123",
        "apr": "120",
        "collateral": "1231231",
        "portfolioVarMargin": "500",
        "riskRatio": "20.0000",
        "maintenanceMargin": "1231",
        "marginRatio": "12.3179",
        "liquidating": false,
        "feeTier": "6",
        "createdAt": "1623042343252",
        "lastUpdatedAt": "1623142532134"
    }
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
hashToken | STRING | YES | List filter, multiple hashToken should be seprated with comma, maximum 5 hashToken supported, e.g. `CF-BCH-AMM-ABCDE3iy,CF-BCH-AMM-ABCDE4iy`|

Response Field | Type | Description |
-------------- | ---- | ----------- |
hashToken | STRING | Hash token |
leverage | STRING | Leverage |
direction | STRING | Direction |
marketCode | STRING | Market code |
initialCollateral | JSON | Initial collateral |
minPriceBound | STRING | Minimum price bound |
maxPriceBound | STRING | Maximum price bound |
status | STRING | Status |
positions | LIST of dictionaries | Positions |
baseAsset | STRING | Base asset |
counterAsset | STRING | Counter asset |
position | STRING | Position |
entryPrice | STRING | Entry price |
markPrice | STRING | Mark price |
positionPnl | STRING | Position PNL |
estLiquidationPrice | STRING | Estimated liquidation price |
lastUpdatedAt | STRING | Timestamp of last updated at |
balances | LIST of dictionaries | Balances |
asset | STRING | Asset |
total | STRING | Total balance |
available | STRING | Available balance |
reserved | STRING | Reserved balance |
usdReward | STRING | USD reward |
flexReward | STRING | FLEX reward |
interestPaid | STRING | Interest paid |
apr | STRING | APR denotes annual percentage rate |
collateral | STRING | Collateral |
notionalPositionSize | STRING | Notional position size |
portfolioVarMargin | STRING | Portfolio margin |
riskRatio | STRING | collateralBalance / portfolioVarMargin, Orders are rejected/cancelled if the risk ratio drops below 1 and liquidation occurs if the risk ratio drops below 0.5 |
maintenanceMargin | STRING | Maintenance margin |
marginRatio | STRING | Margin ratio |
liquidating | BOOLEAN | Available values: `true` and `false` |
feeTier | STRING | Fee tier |
createdAt | STRING | Timestamp of created at |
lastUpdatedAt | STRING | Timestamp of last updated at |
loans | LIST of dictionaries | Loans |
borrowedAsset | STRING | Borrowed asset |
borrowedAmount | STRING | Borrowed amount |
collateralAsset | STRING | Collateral asset |

### GET `/v3/AMM/positions`

Get AMM positions

> **Request**

```
GET /v3/AMM/positions?hashToken={hashToken},{hashToken}&marketCode={marketCode}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "hashToken": "CF-BCH-AMM-ABCDE3iy",
            "positions": [
                {
                    "marketCode": "BTC-USD-SWAP-LIN",
                    "baseAsset": "BTC",
                    "counterAsset": "USD",
                    "position": "0.94",
                    "entryPrice": "7800.00", 
                    "markPrice": "33000.00", 
                    "positionPnl": "200.3",
                    "estLiquidationPrice": "12000.05",
                    "lastUpdatedAt": "1592486212218"
                }
            ]
        }
    ]
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
hashToken | STRING | YES | List filter, multiple hashToken should be seprated with comma, maximum 5 hashToken supported, e.g. `CF-BCH-AMM-ABCDE3iy,CF-BCH-AMM-ABCDE4iy` |
marketCode | STRING | NO | Market code |

Response Field | Type | Description |
-------------- | ---- | ----------- |
hashToken | STRING | Hash token |
positions | LIST of dictionaries | Positions |
marketCode | STRING | Market code |
baseAsset | STRING | Base asset |
counterAsset | STRING | Counter asset |
position | STRING | Position size |
entryPrice | STRING | Entry price |
markPrice | STRING | Mark price |
positionPnl | STRING | Position PNL |
estLiquidationPrice | STRING | Estimated liquidation price |
lastUpdatedAt | STRING | Timestamp of last updated at |


### GET `/v3/AMM/trades`

Get AMM trades

> **Request**

```
GET /v3/AMM/trades?hashToken={hashToken},{hashToken}&marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{    
    "success": true,
    "data": [
        {         
            "orderId": "160067484555913076",
            "clientOrderId": "123",
            "matchId": "160067484555913077",
            "marketCode": "FLEX-USD",
            "side": "SELL",
            "matchedQuantity": "0.1",
            "matchPrice": "0.065",
            "total": "0.0065",
            "leg1Price": "0.001",
            "leg2Price": "0.001",
            "orderMatchType": "TAKER",
            "feeAsset": "FLEX",
            "fee": "0.0096",
            "lastMatchedAt": "1595514663626"
        }
    ]
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
hashToken | STRING | YES | List filter, multiple hashToken should be seprated with comma, maximum 5 hashToken supported, e.g. `CF-BCH-AMM-ABCDE3iy,CF-BCH-AMM-ABCDE4iy` |
marketCode | STRING | NO | Market code |

Response Field | Type | Description |
-------------- | ---- | ----------- |
hashToken | STRING | Hash token |
orderId | STRING | Order ID |
clientOrderId | STRING | Client order ID |
matchId | STRING | Match ID |
marketCode | STRING | Market code |
side | STRING | Available values: `BUY` and `SELL` |
matchedQuantity | STRING | Matched quantity |
matchPrice | STRING | Match price |
total | STRING | Total price |
leg1Price | STRING | |
leg2Price | STRING | |
orderMatchType | STRING | Available values: `TAKER` and `MAKER` |
feeAsset | STRING | Fee asset |
fee | STRING | Fee |
lastMatchedAt | STRING | Timestamp of last matched at |


### GET `/v3/AMM/orders`

Get AMM orders

> **Request**

```
GET /v3/AMM/orders?hashToken={hashToken}

```

> **Successful response format**

```json
{ 
    "success": true,
    "data": [
        {
            "orderId": "304354590153349202",
            "clientOrderId": "1",
            "marketCode": "BTC-USD-SWAP-LIN", 
            "status": "OPEN",
            "side": "BUY", 
            "price": "1.0",
            "stopPrice": "0.9",
            "isTriggered": "true",
            "quantity": "0.001",
            "remainQuantity": "0.001",
            "matchedQuantity": "0",
            "orderType": "LIMIT", 
            "timeInForce": "GTC",
            "createdAt": "1613089383656", 
            "lastModifiedAt": "1613089393622",
            "lastMatchedAt": "1613099383613"
        }
    ]
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
hashToken | STRING | YES | maximum 1 hashToken supported, e.g. `CF-BCH-AMM-ABCDE3iy` |

Response Field | Type | Description |
-------------- | ---- | ----------- |
orderId | STRING | Order ID |
clientOrderId | STRING | Client order ID |
marketCode | STRING | Market code |
status | STRING | Available values: `PARTIALLY_FILLED` and `OPEN` |
side | STRING | Available values: `BUY` and `SELL` |
price | STRING | Limit price |
stopPrice | STRING | Stop price for the stop-limit order |
isTriggered | BOOLEAN | Available values: `true` and `false` |
quantity | STRING | Quantity |
remainQuantity | STRING | Remaining order quantity |
matchedQuantity | STRING | Matched quantity |
orderType | STRING | Available values: `LIMIT` or `STOP` |
timeInForce | STRING | Time in force |
createdAt | STRING | Timestamp of created at |
lastModifiedAt | STRING | Timestamp of last modified at |
lastMatchedAt | STRING | Timestamp of last matched at |


### GET `/v3/AMM/balances`

Get AMM balances

> **Request**

```
GET /v3/AMM/balances?hashToken={hashToken},{hashToken}&asset={asset}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "hashToken": "CF-BCH-AMM-ABCDE3iy",
            "balances": [
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
                }
            ]
        }
    ]
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
hashToken | STRING | YES | List filter, multiple hashToken should be seprated with comma, maximum 5 hashToken supported, e.g. `CF-BCH-AMM-ABCDE3iy,CF-BCH-AMM-ABCDE4iy` |
asset | STRING | NO | |

Response Field | Type | Description |
-------------- | ---- | ----------- |
hashToken | STRING | Hash token |
balances | LIST of dictionaries | Balances |
asset | STRING | Asset |
total | STRING | Total balance |
available | STRING | Available balance |
reserved | STRING | Reserved balance |
lastUpdatedAt | STRING | Timestamp of last updated at |

## Market Data - Public

### GET `/v3/markets`

Get a list of markets on CoinFlex.

> **Request**

```
GET /v3/markets?marketCode={marketCode}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "marketCode": "BTC-USD",
            "name": "BTC/USD",
            "referencePair": "BTC/USD",
            "base": "BTC",
            "counter": "USD",
            "type": "SPOT",
            "tickSize": "0.1",
            "minSize": "0.001",
            "listedAt": "1593345600000",
            "upperPriceBound": "65950.5",
            "lowerPriceBound": "60877.3",
            "markPrice": "63413.9",
            "lastUpdatedAt": "1635848576163"
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
marketCode | STRING | NO | |

Response Field | Type | Description |
-------------- | ---- | ----------- |
marketCode | STRING | Market Code |
name | STRING | Name of the contract |
referencePair | STRING | Reference pair |
base | STRING | Base asset |
counter | STRING | Counter asset |
type | STRING | Type of the contract |
tickSize | STRING | Tick size of the contract |
minSize | STRING | Minimum quantity |
listedAt | STRING | Listing date of the contract |
settlementAt | STRING | Timestamp of settlement if applicable i.e. Quarterlies and Spreads |
upperPriceBound | STRING | Sanity bound |
lowerPriceBound | STRING | Sanity bound |
markPrice | STRING | Mark price |
lastUpdatedAt | STRING | |

### GET `/v3/assets`

Get a list of assets supported on CoinFLEX

> **Request**

```
GET /v3/assets?asset={asset}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "name": "FLEX",
            "canDeposit": true,
            "canWithdraw": true,
            "isCollateral": true,
            "loanToValue": "0.900000000",
            "minDeposit": "0.0001",
            "minWithdrawal": "0.0001",
            "network": [
                "SLP",
                "ERC20",
                "SEP20"
            ]
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Name of the asset |

Response Field | Type | Description |
-------------- | ---- | ----------- |
name | STRING | Name of the asset |
canDeposit | BOOL | |
canWithdraw | BOOL | |
isCollateral | BOOL | |
loanToValue | STRING | Loan to value of the asset |
minDeposit | STRING | Minimum deposit amount |
minWithdrawal | STRING | Minimum withdrawal amount |
network | LIST | Available networks for deposits and withdrawals |
