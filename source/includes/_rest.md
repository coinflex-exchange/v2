# REST API

DEMO/STAGE site

`https://stgwebapi.coinflex.com/`

LIVE site

`https://webapi.coinflex.com/`

For clients who do not wish to take advantage of CoinFLEX's native [WebSocket API][], CoinFLEX offers a RESTful API that implements much of the same functionality.

## Authentication

Authenticated requests to the REST API use [HTTP Basic Authentication][], wherein the *user-id* is the concatenation of user's numeric ID, a slash, and the user's Base64-encoded API key, and the *password* is the user's password or the Base64 encoding of the user's 28-byte private key that is derived from the user's ID and password.

The public market data methods do not require authentication.

## Common Record Types

### `<balance>`

```json
{
    "id": <integer>,
    "available": <integer>,
    "reserved": <integer>
}
```

* **`id`:** *(integer)* The numeric asset code of an asset in which the user's balances are reported.
* **`available`:** *(integer)* The [scaled][] amount of the user's available balance in the identified asset.
* **`reserved`:** *(integer)* The [scaled][] amount of the user's reserved balance in the identified asset.

### `<order>`

```json
{
    "id": <integer>,
    "base": <integer>,
    "counter": <integer>,
    "quantity": <integer>,
    "price": <integer>,
    "time": <integer>
}
```

* **`id`:** *(integer)* The numeric identifier of an order.
* **`base`:** *(integer)* The numeric identifier of the base asset of the order.
* **`counter`:** *(integer)* The numeric identifier of the counter asset of the order.
* **`quantity`:** *(integer)* The [scaled][] amount of the base asset that the order offers to trade. It is negative for a sell order and positive for a buy order.
* **`price`:** *(integer)* The [scaled][] price at which the order offers to trade.
* **`time`:** *(integer)* The micro-timestamp at which the order was opened.

### `<ticker>`

```json
{
    "base": <integer>,
    "counter": <integer>,
    "name": <string>,
    "spot_name": <string>
    "last": <integer>|null,
    "bid": <integer>|null,
    "ask": <integer>|null,
    "low": <integer>|null,
    "high": <integer>|null,
    "volume": <integer>,
    "time": <integer>
}
```

* **`base`:** *(integer)* The numeric identifier of the base asset of the ticker.
* **`counter`:** *(integer)* The numeric identifier of the counter asset of the ticker.
* **`name`:** *(string)* The string name of the market consisting of the base asset name and counter asset name.
* **`spot_name`:** *(string)* The string name of the underlying spot market for a futures market.
* **`last`:** *(integer or `null`)* The [scaled][] price of the last trade in the identified market, or `null` if no trade has occurred in the identified market.
* **`bid`:** *(integer or `null`)* The [scaled][] price of the highest-priced open bid order in the identified market, or `null` if no bid orders are open in the identified market.
* **`ask`:** *(integer or `null`)* The [scaled][] price of the lowest-priced open ask order in the identified market, or `null` if no ask orders are open in the identified market.
* **`low`:** *(integer or `null`)* The [scaled][] price of the lowest-priced trade in the identified market in the preceding 24-hour period, or `null` if no trade has occurred in the identified market in the preceding 24-hour period.
* **`high`:** *(integer or `null`)* The [scaled][] price of the highest-priced trade in the identified market in the preceding 24-hour period, or `null` if no trade has occurred in the identified market in the preceding 24-hour period.
* **`volume`:** *(integer)* The [scaled][] quantity of the base asset that has traded in the identified market in the preceding 24-hour period.
* **`time`:** *(integer)* The micro-timestamp at which the last update has occurred.

### `<trade>`

```json
{
    "time": <integer>,
    "base": <integer>,
    "counter": <integer>,
    "quantity": <integer>,
    "price": <integer>,
    "total": <integer>,
    "base_fee": <integer>,
    "counter_fee": <integer>,
    "order_id": <integer>|null
}
```

* **`time`:** *(integer)* The micro-timestamp at which the trade occurred.
* **`base`:** *(integer)* The numeric identifier of the base asset that was traded.
* **`counter`:** *(integer)* The numeric identifier of the counter asset that was traded.
* **`quantity`:** *(integer)* The [scaled][] amount of the base asset that was traded.
* **`price`:** *(integer)* The [scaled][] price at which the trade occurred.
* **`total`:** *(integer)* The [scaled][] amount of the counter asset that was traded.
* **`base_fee`:** *(integer)* The [scaled][] amount of the base asset that was levied as a trading fee.
* **`counter_fee`:** *(integer)* The [scaled][] amount of the counter asset that was levied as a trading fee.
* **`order_id`:** *(integer or `null`)* The numeric identifier of the user's limit order that participated in the trade, or `null` if the user caused the trade by a market order.

## GET `/account_value/`

> **Request**

```javascript
GET '/account_value/' HTTP/1.1
```

> **Response**

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
[{
    "asset": <integer>,
    "spot_value": <integer>,
    "change_in_spot_value": <integer>,
    "futures_value": <integer>,
    "change_in_futures_value": <integer>,
    "total_value": <integer>,
    "change_in_total_value": <integer>
}, …]
```

> * **`asset`:** *(integer)* The numeric asset code of an asset in which the user's account value is reported.
> * **`spot_value`:** *(integer)* The total scaled amount held by the user in this spot asset.
> * **`change_in_spot_value`:** *(integer)* The scaled amount of the diff between the current spot_value and the snapshot spot_value.
> * **`futures_value`:** *(integer)* The total scaled amount held by the user in this futures asset.
> * **`change_in_futures_value`:** *(integer)* The scaled amount of the diff between the current futures_value and the snapshot futures_value.
> * **`total_value`:** *(integer)* The total scaled amount of the sum of spot and futures.
> * **`change_in_total_value`:** *(integer)* The scaled amount of the diff between the total_value and the snapshot total_value.

**Authentication required.**

Returns the user's account value broken down by spot, futures and their value changes since the daily snapshot at 00:00UTC.


## GET `/assets/`

> **Request**

```javascript
GET '/assets/[?activeOnly=<boolean>]' HTTP/1.1
```

> * **`activeOnly`:** *(boolean, optional)* If **true** or omitted this will only return the list of active assets.  If **false**, this will return the entire list of exchange assets, both expired and active assets.

> **Response**

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```javascript
{
    "id": <integer>,
    "spot_id": <integer>,
    "spot_name": <string>,
    "name": <string>,
    "scale": <integer>
}
```

> * **`id`:** *(integer)* The numeric identifier of the asset.
> * **`name`:** *(string)* The string name of the asset.
> * **`spot_id`:** *(integer)* The numeric identifier of the underlying spot asset for a futures asset.  Only applicable for futures assets.
> * **`spot_name`:** *(string)* The string name of the the underlying spot asset for a futures asset.  Only applicable for futures assets.
> * **`scale`:** *(integer)* The scale factor for the asset, applicable to all asset quantities and totals transmitted via the CoinFLEX API.

**Authentication not required.**

Returns a complete list of CoinFLEX assets.

## GET `/markets/`

> **Request**

```javascript
GET '/markets/' HTTP/1.1
```

> **Response**

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
{
    "base": <integer>,
    "counter": <integer>,
    "name": <string>,
    "spot_name": <string>,
    "start": <integer>,
    "expires": <integer>,
    "tenor": <string>,
    "tick": <integer>
}
```

> * **`base`:** *(integer)* The numeric identifier attributed to the base asset of the market.
> * **`counter`:** *(integer)* The numeric identifier attributed to the counter asset of the market.
> * **`name`:** *(string)* The string name of the market consisting of the base asset name and counter asset name.
> * **`spot_name`:** *(string)* The string name of the underlying spot market for a futures market.
> * **`start`:** *(integer)* The UNIX epoch millisecond timestamp of the first trade date of the market.  Only applicable for futures markets.
> * **`expires`:** *(integer)* The UNIX epoch millisecond timestamp of the expiry date of the market.  Only applicable for futures markets.
> * **`tenor`:** *(string)* The tenor code of the market using industry standard codes.
> * **`tick`:** *(integer)* The scaled tick size of the market.

**Authentication not required.**

Returns a complete list of CoinFLEX SPOT and Futures markets.


## GET `/positions/`

> **Request**

```javascript
GET '/positions/' HTTP/1.1
```

> **Response**

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
[{
    "asset": <integer>,
    "position": <integer>,
    "through": <integer>
}, …]
```

> * **`asset`:** *(integer)* The numeric asset code of an asset in which the user's traded net position is reported.
> * **`position`:** *(integer)* The scaled amount of the traded net position in the asset.
> * **`through`:** *(integer)* The micro-timestamp of the last net position change for the identified asset.

**Authentication required.**

Returns the user's traded net position in all assets.


## GET `/tickers/`

> **Request**

```javascript
GET '/tickers/' HTTP/1.1
```

> **Response**

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
[<ticker>, …]
```

**Authentication not required.**

Returns the tickers of all available markets.


## GET `/tickers/<base>:<counter>`

> **Request**

```javascript
GET '/tickers/<base>:<counter>' HTTP/1.1
```

> * **`base`:** *(integer)* The numeric identifier of the base asset of the market whose ticker is to be returned.
> * **`counter`:** *(integer)* The numeric identifier of the counter asset of the market whose ticker is to be returned.

> **Response**

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
<ticker>
```

> **Errors**

> If the specified market was not found, then this method returns:

```yml
HTTP/1.1 404 Not Found
```

**Authentication not required.**

Returns the ticker of the specified market.


## GET `/depth/<base>:<counter>`

> **Request**

```javascript
GET '/depth/<base>:<counter>'
```

> * **`base`:** *(integer)* The numeric identifier of the base asset of the market whose depth information is to be returned.
> * **`counter`:** *(integer)* The numeric identifier of the counter asset of the market whose depth information is to be returned.

> **Response**

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
{
    "bids": [[<price>, <quantity>], …],
    "asks": [[<price>, <quantity>], …]
}
```

> * **`price`:** *(integer)* The [scaled][] price level at which market depth is reported.
> * **`quantity`:** *(integer)* The [scaled][] amount of the base asset representing the total open interest at the specified price level.

> **Errors**

> If the specified market was not found, then this method returns:

```yml
HTTP/1.1 404 Not Found
```

**Authentication not required.**

Returns the amount of open interest at each of the top 20 price levels on each side of the specified order book.

A facility to retrieve full order book listings is intentionally omitted from this API, as such facilities have a tendency to be abused through rapid polling. If you require order book listings, please use the [WebSocket API][], which delivers snapshots followed by incremental updates.


## GET `/balances/`

> **Request**

```javascript
GET '/balances/' HTTP/1.1
```

> **Response**

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
[<balance>, …]
```

**Authentication required.**

Returns the user's available and reserved balances in all assets.


## GET `/balances/<id>`

> **Request**

```javascript
GET '/balances/<id>' HTTP/1.1
```

> * **`id`:** *(integer)* The numeric asset code of the asset in which to report the user's balances.

> **Response**

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
<balance>
```

> **Errors**

> If the specified asset was not found, then this method returns:

```yml
HTTP/1.1 404 Not Found
Content-Length: 0
```

**Authentication required.**

Returns the user's available and reserved balances in the specified asset.


## GET `/orders/`

> **Request**

```javascript
GET '/orders/' HTTP/1.1
```

> **Response**

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
[<order>, …]
```

**Authentication required.**

Returns the user's open limit orders.


## GET `/orders/<id>`

> **Request**

```javascript
GET /orders/<id> HTTP/1.1
```

> **Response**

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
<order>
```

> **Errors**

> If the specified order was not found, then this method returns:

```yml
HTTP/1.1 404 Not Found
Content-Length: 0
```

**Authentication required.**

Returns the specified limit order of the user.


## POST `/orders/`

> **Request**

```javascript
POST '/orders/' HTTP/1.1
Content-Type: application/x-www-form-urlencoded
```

```yml
base=<integer>&counter=<integer>[&price=<integer>][&quantity=<integer>|&total=<integer>]
```

> * **`base`:** *(integer)* The numeric identifier of the base asset of the order.
> * **`counter`:** *(integer)* The numeric identifier of the counter asset of the order.
> * **`price`:** *(integer, optional)* The [scaled][] price at which to offer to trade. If omitted, the submitted order will be executed immediately as a market order.
> * **`quantity`:** *(integer, conditional)* The [scaled][] amount of the base asset to be traded. Negative for a sell order or positive for a buy order. Required if `price` is supplied or `total` is omitted.
> * **`total`:** *(integer, conditional)* The [scaled][] amount of the counter asset to be traded. Negative for a sell order or positive for a buy order. Required if `quantity` is omitted; forbidden if `quantity` is supplied.

> **Response**

> The response for a limit order (`price` supplied) is:

```yml
HTTP/1.1 201 Created
Location: <id>
Content-Location: <id>
Content-Type: application/json; charset=US-ASCII
```

> * **`id`:** *(integer)* The numeric identifier of the newly placed limit order.

```json
<order>
```

> The response for a market order (`price` omitted) is:

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
{"remaining":<integer>}
```

> * **`remaining`:** *(integer)* The [scaled][] amount of the base asset (if `quantity` was supplied in the request), or the [scaled][] amount of the counter asset (if `total` was supplied in the request), that could not be traded, either because insufficient liquidity existed on the order book or because the user's balance was exhausted.

**Authentication required.**

Places a limit order or executes a market order.

`quantity` | `price`  | `total`
-----------|----------|---------
supplied   | supplied | omitted
supplied   | omitted  | omitted
omitted    | omitted  | supplied


## DELETE `/orders/`

> **Request**

```javascript
DELETE '/orders/' HTTP/1.1
```

> **Response**

> The response contains a snapshot of all of the orders that were canceled.

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
[<order>, …]
```

**Authentication required.**

Cancels all of the user's open limit orders.


## DELETE `/orders/<id>`

> **Request**

```javascript
DELETE '/orders/<id>' HTTP/1.1
```

> * **`id`:** *(integer)* The numeric identifier of the order to be canceled.

> **Response**

> The response contains a snapshot of the order that was canceled.

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
<order>
```

> **Errors**

> If the specified order was not found, then this method returns:

```yml
HTTP/1.1 404 Not Found
Content-Length: 0
```

**Authentication required.**

Cancels the specified limit order of the user.


## GET `/trades/`


> **Request**

```javascript
GET '/trades/[?since=<integer>][&until=<integer>][&sort=asc|desc][&limit=<integer>]' HTTP/1.1
```

> * **`since`:** *(integer, optional)* A micro-timestamp by which to constrain the set of returned trade records. If supplied, only records of trades that occurred *strictly after* this time will be returned.
> * **`until`:** *(integer, optional)* A micro-timestamp by which to constrain the set of returned trade records. If supplied, only records of trades that occurred *strictly before* this time will be returned.
> * **`sort`:** *(string, optional)* The order in which to sort records, either `asc` for ascending order or `desc` for descending order. In addition to causing the returned records to be sorted as specified, this also affects which records are selected if not all records in the specified time range are returned. If omitted, the default sort order is ascending if `since` is supplied, or else descending.
> * **`limit`:** *(integer, optional)* The maximum number of trade records to return. Note, however, that the number of records returned may be less than this limit even if more records exist in the specified time range, but at least one record is guaranteed to be returned if any records exist in the specified time range. If omitted, there is no particular limit imposed by the request.

> **Response**

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
[<trade>, …]
```

**Authentication required.**

Returns historical records of the user's past trades.


## GET `/trades/<time>`

> **Request**

```javascript
GET '/trades/<time>' HTTP/1.1
```

> * **`time`:** *(integer)* The micro-timestamp of the trade to return.

> **Response**

```yml
HTTP/1.1 200 OK
Content-Type: application/json; charset=US-ASCII
```

```json
<trade>
```

> **Errors**

> If the specified trade was not found, then this method returns:

```yml
HTTP/1.1 404 Not Found
Content-Length: 0
```

**Authentication required.**

Returns the historical record of a particular past trade of the user.


[HTTP Basic Authentication]: https://tools.ietf.org/html/rfc7617
    (The 'Basic' HTTP Authentication Scheme)
[WebSocket API]: WEBSOCKET-README.md
[scaled]: #scaling
