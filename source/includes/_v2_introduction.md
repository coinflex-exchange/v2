# Introduction

Welcome to the CoinFLEX v2 API. Please note this is a beta version and can be considered a testnet.

CoinFLEX offers REST and WebSocket APIs. Both support market data and trading. It is recommend you use Websocket for more consistent and quicker response times
Current server: Alicloud Hong Kong

Quick start:
1. Register for an account at https://test-v2.coinflex-cn.com/login
2. POST vis RESTAPI at
  https://api-test-v2.coinflex-cn.com/v2/account/auth/trading/login
  with the following JSON to obtain your token:

  > **Request format**

Valid options of op can be:  login, subscribe, unsubscribe, placeorder, cancelorder etc.

```json
 
  {
    "email":"<email>",
    "password":"<password>"
  }

```

> **RESPONSE**

```json
  {
      "event": null,
      "success": true,
      "message": null,
      "code": "0000",
      "data": {
          "token": "...",
          "email": "xxxx@xxx.com",
          "accountName": "xxxx@xxx.com",
          "mainLogin": true
      }
  }

```
# Change Log

2020-06-16
- Added general guidance for getting a login/password to create tokens
- Updated some addresses for REST API

2020-06-15
- First version of beta API endpoints - Websocket and REST

