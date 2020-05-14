---
title: API Reference

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - auth
  - bist
  - contracts
  - event_stream
  - futures
  - impl_guid
  - limits
  - rest
  - scale
  - websocket_readme

search: true
---

# CoinFLEX Trade Engine APIs

CoinFLEX's application programming interfaces (APIs) provide our clients programmatic access to control aspects of their accounts and to place orders on CoinFLEX's trading platforms.

CoinFLEX provides several APIs:

* our native [WebSocket API][]
* a [RESTful API](#rest-api)
* an [Event Stream resource](#event-stream)
* a second futures [Event Stream resource](#futures-api-specification-get-borrower-events) for your collateral, leverage and margin

Using these interfaces it is possible to make both authenticated and unauthenticated API calls, with the exception of the Futures Event Stream which is authenticated only.

Access keys are available on the CoinFLEX logged-in dashboard page for verified users which, in conjunction with your account password, allow authenticated use these APIs.


## General notes

To protect the performance of the system, CoinFLEX imposes certain limits on the rates at which you may issue commands to the API. Please see [Request Limits](#request-limits).

All quantities and prices are transmitted and received as integers with implicit scale factors. For scale information, please see [Scaling](#scaling).

CoinFLEX has published [client libraries][] for several popular languages to aid you in implementing your client application.


## Getting started with the WebSocket API

**The [WebSocket API][] is accessible via [WebSocket][] connection to the following URLs:**


DEMO/STAGE site

`wss://stgapi.coinflex.com/v1   (encrypted)`

LIVE site

`wss://api.coinflex.com/v1      (encrypted)`

Commands, replies, and notifications traverse the WebSocket in text frames with [JSON][]-formatted payloads.

<aside class="success">
We strongly recommend that you use TLS transport for all connections.
</aside>

[Click here for more details on how to use the WebSocket API][WebSocket API]



[WebSocket API]: #websocket-api-specification
[JSON]: https://tools.ietf.org/html/rfc4627 (IETF RFC 4627)
[WebSocket]: https://tools.ietf.org/html/rfc6455 (IETF RFC 6455)
[client libraries]: https://github.com/coinflex-exchange/

