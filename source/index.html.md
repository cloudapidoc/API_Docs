---
title: API 文档

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - 
includes:

search: true
---

# Signing
Important Note: Do not reveal your 'AccessKey' and 'SecretKey' to anyone. They are as important as your password.

## Components of a Query Request

Each HTTPS request formatted for Signature should contain the following:

* AccessKeyId: The 'AccessKey' distributed by Server when you applied for APIKEY.
* SignatureMethod: The hash-based protocol used to calculate the signature. This should be HmacSHA256.
* SignatureVersion: The version of the signature protocol.This shoule be 2.
* Timestamp: The time at which you make the request. Include this in the Query request to help prevent third parties from intercepting your request.This should be formatted in UTC time, like '2017-05-11T16:22:06'.
* Required and optional parameters: Each action has a set of required and optional parameters that define the API call.This shows in the API reference.Important Reminder: Every param in a GET method should be included in signature calculation, and only AccessKeyId, SignatureMethod, SignatureVersion, Timestamp should be included in signature calculation for a POST method.
* Signature: The calculated value that ensures the signature is valid and is not tampered.

For example:
```
https://{HOST}/api/v1/order/orders?
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
&SignatureMethod=HmacSHA256
&SignatureVersion=2
&Timestamp=2017-05-11T15%3A19%3A30
&order-id=1234567890
&Signature=calculated value
```

## How to Generate a Signature

Web service requests are sent across the Internet and are vulnerable to tampering. For security reasons,  Server requires a signature as part of every request.

### Task 1: Format the Query Request

Before you can sign the Query request,  normalize the request in a standardized (canonical) format. This is necessary because different formats will result in different HMAC signatures. Normalize the request in a canonical format before signing. This ensures your application and server will calculate the same signature for a request.

The following example generates the string to sign for the following call to the server.

```
https://{HOST}/api/v1/order/orders?
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
&SignatureMethod=HmacSHA256
&SignatureVersion=2
&Timestamp=2017-05-11T15:19:30
&order-id=1234567890
```

1. Start with the request method (either GET or POST),  followed by a newline character. For human readability,  the newline character is represented as \n.

```
GET\n
```

2. Add the root url in lowercase,  followed by a newline character.

```
{HOST}\n
```

3. Add the method,  followed by a newline character. 

```
{HOST}\n
```

4. Add the query string components,  as UTF-8 characters which are URL encoded (hexadecimal characters must be uppercase). Please do not encode the initial question mark character (?) in the request.Then sort the query string components by byte order. Byte ordering is case sensitive.

For example, this is the original order for the query string components.

```
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
order-id=1234567890
SignatureMethod=HmacSHA256
SignatureVersion=2
Timestamp=2017-05-11T15%3A19%3A30
```
The query string components would be reorganized as the following:

```
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
SignatureMethod=HmacSHA256
SignatureVersion=2
Timestamp=2017-05-11T15%3A19%3A30
order-id=1234567890
```

5. Separate parameter names from their values with the equal sign character (=),  even if the value is empty. Separate parameter and value pairs with the ampersand character (&). Concatenate the parameters and their values to make one long string with no spaces. Spaces within a parameter value are allowed,  but must be URL encoded as %20. In the concatenated string,  period characters (.) are not escaped. 

The following example shows the query string components,  with the parameters concatenated with the ampersand character (&),  and sorted by byte order.

```
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890
```
To construct the finished canonical request,  combine all the components from each step. As shown,  each component ends with a newline character.

```
GET\n
{HOST}\n
/v1/order/orders\n
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890
```

Task 2: Calculate the Signature

1. The signature is calculated with the following canonical string and secret key as inputs to a keyed hash function:

* Canonical query string:

```
GET\n
{HOST}\n
/v1/order/orders\n
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890
```

*  Secret key(The 'SecretKey' distributed by server when you applied for APIKEY.):

```
b0xxxxxx-c6xxxxxx-94xxxxxx-dxxxx
```

* The resulting signature must be base-64 encoded.

```
4F65x5A2bLyMWVQj3Aqp+B4w+ivaA7n5Oi2SuYtCJ9o=
```

2. Add the resulting value to the query request as a Signature parameter. When you add this parameter to the request,  you must URI encode it just like any other parameter. You can use the signed request in an HTTP or HTTPS call.

3. The final request ：

```
https://{HOST}/v1/order/orders?AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&order-id=1234567890&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&Signature=4F65x5A2bLyMWVQj3Aqp%2BB4w%2BivaA7n5Oi2SuYtCJ9o%3D
```

# Request Process

1.  rest public api：www.xxxx.com/api without verify
    ；rest private api：www.xxxx.com/api verify host:www.xxxx.com
    ；ws public api：www.xxxx.com/api/ws without verify
    ；ws private api：www.xxxx.com/ws/v1 verify host：www.xxxx.com ；verify path：ws/v1.
2. Host Structure: www.xxxx.com/api (Should you have any question about the Host, please consult with your host provider.)
3. The field 'Content-Type' in request header should be 'application/x-www-form-urlencoded' for a GET method, and 'application/x-www-form-urlencoded' for a POST method. 
4. Request parameter: Construct the request parameters according to each API interface.
5. Submit request: Use POST or GET method to send request to server.
6. HTTPS is required.
7. Server respond: Server processes request and returns data to users in JSON format after authentication check.

# API Reference

``` Composition rules of 'symbol' ： 'base-currency' + 'quote-currency'.For 'BTC/USDT' trading,the 'symbol' is btcusdt, and so on.```

## List of Requests
| Content Type | Request Method | Request Type    | Description  | Signature Required |Sub-UID Allowed|
| ----------- | ----- | ------ | ----- | ----- | ---|
| Market Data       | [GET /market/history/kline](#get-markethistorykline--get-candlestick-data)  | GET | K-Line | N |Y|
| Market Data      | [GET /market/detail/merged](#get-marketdetailmerged-get-ticker)  | GET | 24 hours trade summary and best bid/ask for a symbol | N |Y|
|Market Data       | [GET /market/tickers](#get-markettickers)  | GET | Trade summary of a trading day for all symbols | N |Y|
|Market Data       | [GET /market/depth](#get-marketdepth---market-depth)  | GET | Market Depth of a symbol | N |Y|
|Market Data      | [GET /market/trade](#get-markettrade--trade-detail)  | GET | The last trade of a symbol | N |Y|
|Market Data      | [GET /market/history/trade](#get-markethistorytrade-orderbook)  | GET | Request a batch of trade records of a symbol, up to 2000 records| N |Y|
| Symbols&Currency      | [GET /v1/common/symbols]  | GET | Reference information of trading instrument, including base currency, quote precision, etc.  |N |Y|
| Symbols&Currency       | [GET /v1/common/currencys]  | GET | The list of currencies available |N |Y|
|Account	|[GET /v1/account/accounts] |	GET| get the status of an account| Y|Y|
|Account	|[GET /v1/account/accounts/{account-id}/balance]|GET|	Get the balance of an account	|Y|Y|
|Trade	|[POST/v1/order/orders/place]|POST|Place an order	|Y|Y|
|Trade	|[POST/v1/order/orders/{order-id}/submitcancel](#post-v1orderordersorder-idsubmitcancel--request-for-cancelling-an-order)|POST|	Request to cancel an order	|Y|Y|
|Trade	|[POST /v1/order/orders/batchcancel](#post-v1orderordersbatchcancel--batch-cancel)|POST|Request to cancel a batch of orders, up to 50 orders	|Y|Y|
|Trade	|[POST /v1/order/orders/batchCancelOpenOrders](#post--v1orderbatchcancelopenorders--cancel-a-batch-of-orders-with-certain-criteria)|POST|	Request to cancel a batch of orders, which meet certain criteria, up to 100 orders	|Y|Y|
|User Order Info	|[GET /v1/order/orders/{order-id}](#get-v1orderordersorder-id----get-order-info)|GET|Get the details of an order|Y|Y|
|User Order Info	|[GET /v1/order/orders/{order-id}/matchresults](#get-v1orderordersorder-idmatchresults--get-order-matchresult)	|GET| Get detail match results of an order	|Y|Y|
|User Order Info	|[GET /v1/order/orders](#get-v1orderorders--get-order-list)	|GET|Search for a group of orders, which meet certain criteria (up to 100)	|Y|Y|
|User Order Info	|[GET /v1/order/matchresults](#get-v1ordermatchresults----get-order-matchresults)	|GET|Search for the trade records of an account|Y|Y|
|User Order Info	|[GET /v1/order/openOrders](#get-v1orderopenorders-provide-open-orders-of-a-symbol-for-an-account)	|GET|Get the open orders of an account (up to 500)|Y|	Y|
|Withdraw/Deposit	|[POST /v1/dw/withdraw/api/create](#post-v1dwwithdrawapicreate---create-a-withdraw-application)	|POST|Submit a request to withdraw some asset from an account|	Y|N|
|Withdraw/Deposit	|[POST /v1/dw/withdraw-virtual/{withdraw-id}/cancel](#post-v1dwwithdraw-virtualwithdraw-idcancel-cancel-a-withdraw)|POST| 	Cancel an withdraw request|Y|N|
|Withdraw/Deposit	|[GET /v1/query/deposit-withdraw](#get-v1querydeposit-withdraw---get-deposit-or-withdraw-records)	|GET|Get the withdraw and deposit records of an account|Y|N|

### Market Data API

#### GET /market/history/kline  Get Candlestick Data

Request: 

| Param | required  | data type      | description  | default   | value range  |
| ------------ | ----- | ------ | ----- | ----- | ------- |
| symbol       | true  | string | trading asset  |  | btcusdt, bccbtc, rcneth ...   |
| period       | true  | string |    |    | 1min, 5min, 15min, 30min, 60min, 1day, 1mon, 1week, 1year |
| size | false | integer |   | 150 | [1,2000] |

Response: 

| Param   | required | data type    | description   | value range   |
| ------ | ---- | ------ | ----------- | ------ |
| status | true | string |      | "ok" , "error" |
| ts     | true | number | timestamp in millisecond  |    |
| tick   | true | object |     |      |
| ch     | true | string |   channel,format： market.$symbol.kline.$period |    |

data : 

```
  "data": [
{
    "id": kline id,
    "amount": trading amount,
    "count": 
    "open": Open price,
    "close": Closing price.If this is a latest kline,this shows the current price
    "low":  
    "high": 
    "vol": volume
  }
]
```

Example: 

```
/* GET /market/history/kline?period=1day&size=200&symbol=btcusdt */
{
  "status": "ok",
  "ch": "market.btcusdt.kline.1day",
  "ts": 1499223904680,
  “data”: [
{
    "id": 1499184000,
    "amount": 37593.0266,
    "count": 0,
    "open": 1935.2000,
    "close": 1879.0000,
    "low": 1856.0000,
    "high": 1940.0000,
    "vol": 71031537.97866500
  },
// more data here
]
}

/* GET /market/history/kline?period=not-exist&size=200&symbol=ethusdt */
{
  "ts": 1490758171271,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid period"
}

/* GET /market/history/kline?period=1day&size=not-exist&symbol=ethusdt */
{
  "ts": 1490758221221,
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "invalid size, valid range: [1,2000]"
}

/* GET /market/history/kline?period=1day&size=200&symbol=not-exist */
{
  "ts": 1490758171271,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid symbol"
}
```

#### GET /market/detail/merged Get Ticker

Request: 

| Param   | required  | data type      | description  | default   | value range   |
| ------------ | ----- | ------ | -----  | ---  |  ----------- |
| symbol    | true  | string | trading asset   |   | btcusdt, bccbtc, rcneth ...|

Response: 

| Param   | required | data type    | description   | value range   |
| ------ | ---- | ------ | -------  | ----  |
| status | true | string |    | "ok" , "error" |
| ts     | true | number | timestamp in millisecond    |     |
| tick   | true | object | kline data    |      |
| ch     | true | string | channel, format： market.$symbol.detail.merged |     |

tick : 

```
  "tick": {
    "id":  
    "amount":  
    "count":  
    "open":  
    "close": Closing price.If this is a latest kline,this shows the current price
    "low":  
    "high":  
    "vol":  volume
    "bid": [bid1 price, volume],
    "ask": [ask1 price, volume]
  }

```

Example: 

```
/* GET /market/detail/merged?symbol=ethusdt */
{
"status":"ok",
"ch":"market.ethusdt.detail.merged",
"ts":1499225276950,
"tick":{
  "id":1499225271,
  "ts":1499225271000,
  "close":1885.0000,
  "open":1960.0000,
  "high":1985.0000,
  "low":1856.0000,
  "amount":81486.2926,
  "count":42122,
  "vol":157052744.85708200,
  "ask":[1885.0000,21.8804],
  "bid":[1884.0000,1.6702]
  }
}

/* GET /market/detail/merged?symbol=not-exist */
{
  "ts": 1490758171271,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid symbol”
}

```

#### GET /market/tickers
will get something like this
```
{  
    "status":"ok",
    "ts":1510885463001,
    "data":[  
        {  
            "open":0.044297,      // daily Kline,opennig price
            "close":0.042178,     // daily Kline,closing price
            "low":0.040110,       // daily Kline,the minimum price
            "high":0.045255,      // daily Kline,the maxmum price
            "amount":12880.8510,  
            "count":12838,        
            "vol":563.0388715740, 
            "symbol":"ethbtc"     
        },
        {  
            "open":0.008545,
            "close":0.008656,
            "low":0.008088,
            "high":0.009388,
            "amount":88056.1860,
            "count":16077,
            "vol":771.7975953754,
            "symbol":"ltcbtc"
        }
    ]
}
```
#### GET /market/depth   Market Depth  

Request:

| Param   | required  | data type      | description    | default   | value range   |
| ------  | ----- | ------ | ------  | ----- | -------  |
| symbol     | true  | string | trading asset    |       | btcusdt, bccbtc, rcneth ... |
| type    | true  | string | Depth data type      |       | step0, step1, step2, step3, step4, step5 |


Response:

| Param   | required | data type    | description    | value range    |
| ------ | ---- | ------ | -------  | ---  |
| status | true | string |       | "ok" or "error" |
| ts     | true | number | timestamp in millisecond    |     |
| tick   | true | object | Depth     |     |
| ch     | true | string | channel, format： market.$symbol.depth.$type |  |

tick :

```
  "tick": {
    "id": 
    "ts": millisecond,
    "bids":  [price , amount ] 
    "asks":  [price , amount ] 
  }
```

Example:

```
/* GET /market/depth?symbol=ethusdt&type=step1 */
{
  "status": "ok",
  "ch": "market.btcusdt.depth.step1",
  "ts": 1489472598812,
  "tick": {
    "id": 1489464585407,
    "ts": 1489464585407,
    "bids": [
      [7964, 0.0678], // [price, amount]
      [7963, 0.9162],
      [7961, 0.1],
      [7960, 12.8898],
      [7958, 1.2],
      [7955, 2.1009],
      [7954, 0.4708],
      [7953, 0.0564],
      [7951, 2.8031],
      [7950, 13.7785],
      [7949, 0.125],
      [7948, 4],
      [7942, 0.4337],
      [7940, 6.1612],
      [7936, 0.02],
      [7935, 1.3575],
      [7933, 2.002],
      [7932, 1.3449],
      [7930, 10.2974],
      [7929, 3.2226]
    ],
    "asks": [
      [7979, 0.0736],
      [7980, 1.0292],
      [7981, 5.5652],
      [7986, 0.2416],
      [7990, 1.9970],
      [7995, 0.88],
      [7996, 0.0212],
      [8000, 9.2609],
      [8002, 0.02],
      [8008, 1],
      [8010, 0.8735],
      [8011, 2.36],
      [8012, 0.02],
      [8014, 0.1067],
      [8015, 12.9118],
      [8016, 2.5206],
      [8017, 0.0166],
      [8018, 1.3218],
      [8019, 0.01],
      [8020, 13.6584]
    ]
  }
}

/* GET /market/depth?symbol=ethusdt&type=not-exist */
{
  "ts": 1490759358099,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid type"
}
```


#### GET /market/trade  Trade Detail 

Request:

| Param   | required  | data type   | description   | default   | value range  |
| -------  | ----- | ------ | ------ | ----- | ---- |
| symbol   | true  | string | trading asset   |   | btcusdt, bccbtc, rcneth ... |

Response:

| Param   | required | data type    | description   | value range    |
| ------ | ---- | ------ | ----------| --------------- |
| status | true | string |    | "ok" or "error" |
| ts     | true | number | timestamp in millisecond    |      |
| tick   | true | object | Trade        |     |
| ch     | true | string | channel, format： market.$symbol.trade.detail |     |

tick ：

```
  "tick": {
    "id":  
    "ts": millisecond
    "data": [
      {
        "id":  
        "price":  
        "amount":  
        "direction":  
        "ts":  
      }
    ]
  }
```

Example:

```
/* GET /market/trade?symbol=ethusdt */
{
  "status": "ok",
  "ch": "market.btcusdt.trade.detail",
  "ts": 1489473346905,
  "tick": {
    "id": 600848670,
    "ts": 1489464451000,
    "data": [
      {
        "id": 600848670,
        "price": 7962.62,
        "amount": 0.0122,
        "direction": "buy",
        "ts": 1489464451000
      }
    ]
  }
}

/* GET /market/trade?symbol=not-exist */
{
  "ts": 1490759506429,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid symbol"
}
```
#### GET /market/history/trade orderbook

Request:

| Param   | required  | data type    | description   | default   | value range    |
| ------- | ----- | ------ | ---- | ----- | ---  |
| symbol   | true  | string | trading asset   |       | btcusdt, bccbtc, rcneth ... |
| size  | false  |integer|      |  1   | [1, 2000]    |

Response:

| Param   | required  | data type   | description  | default   | value range   |
| -------- | ----- | ------ | --------  | ----- | ----  |
| status   | true  | string |    |    | ok, error   |
| ch     | true | string | channel, format： market.$symbol.trade.detail  |    |
| ts    | true  |integer|  timestamp  |    |      |
| data  | true  | object |     |    |    |

data ：

```
  "data": {
    "id":  
    "ts":  
    "data": [
      {
        "id":  
        "price":  
        "amount":  
        "direction":  
        "ts":  
      }
    ]
  }
```

Example:

```
/* GET /market/history/trade?symbol=ethusdt */
{
    "status": "ok",
    "ch": "market.ethusdt.trade.detail",
    "ts": 1502448925216,
    "data": [
        {
            "id": 31459998,
            "ts": 1502448920106,
            "data": [
                {
                    "id": 17592256642623,
                    "amount": 0.04,
                    "price": 1997,
                    "direction": "buy",
                    "ts": 1502448920106
                }
            ]
        }
    ]
}
```


### Public API


####  GET /v1/common/symbols     Get all the trading assets

Request:
none

Response:

| Param    | required | data type    | description    | value range |
| -------------- | ---- | ------ | ----- | ---- |
| base-currency  | true | string |    |      |
| quote-currency | true | string |   |      |
| price-precision | true | string |   |      |
| amount-precision | true | string |   |      |
| symbol-partition | true | string |   | main ,innovation ,bifurcation  |

 Example:

```
/* GET /v1/common/symbols */
{
  "status": "ok",
  "data": [
    {
      "base-currency": "eth",
      "quote-currency": "usdt",
      "symbol": "ethusdt"
    }
    {
      "base-currency": "etc",
      "quote-currency": "usdt",
      "symbol": "etcusdt"
    }
  ]
}
```

####  GET /v1/common/currencys     Get currencys supported

 Request:

none

 Response:

```
currency list
```

Example:

```
/* GET /v1/common/currencys */
{
  "status": "ok",
  "data": [
    "usdt",
    "eth",
    "etc"
  ]
}
```

####  GET /v1/common/timestamp    Get system timestamp

Request:

none

 Response:

```
system timestamp
```

Example

```
/* GET /v1/timestamp */
{
  "status": "ok",
  "data": 1494900087029
}
```

### Account API

####  GET /v1/account/accounts Get all the accounts

Request:



Response:

| Param  | required | data type    | description   | value range    |
| ----- | ---- | ------ | ----- | ----  |
| id    | true | long   | account-id |    |
| state | true | string |    | working , lock  |
| type  | true | string |     | spot, margin    |

Example:

```
/* GET /v1/account/accounts */
{
  "status": "ok",
  "data": [
    {
      "id": 100009,
      "type": "spot",
      "state": "working",
      "user-id": 1000
    }
  ]
}
```

####  GET /v1/account/accounts/{account-id}/balance     Get balance

Request

| Param   | required | data type    | description   | default  | value range |
| ---------- | ---- | ------ | --------------- | ---- | ---- |
| account-id | true | string | account-id, not UID, filled in the methos path |      |      |

* Please get your accountid by 'GET /v1/account/accounts'.

Response:

| Param  | required  | data type    | description    | value range   |
| ----- | ----- | ------ | ----- | ----- |
| id    | true  | long   |   |      |
| state | true  | string |    | working, lock  |
| type  | true  | string |     | spot, margin             |
| list  | false | Array  |   |     |

list

| Param   | required | data type    | description   | value range    |
| -------- | ---- | ------ | ---- |  ------ |
| balance  | true | string |     |    |
| currency | true | string |     |    |
| type     | true | string |      | trade ,frozen  |

Example:

```
/* GET /v1/account/accounts/{account-id}/balance */
{
  "status": "ok",
  "data": {
    "id": 100009,
    "type": "spot",
    "state": "working",
    "list": [
      {
        "currency": "usdt",
        "type": "trade",
        "balance": "500009195917.4362872650"
      },
      {
        "currency": "usdt",
        "type": "frozen",
        "balance": "328048.1199920000"
      },
     {
        "currency": "etc",
        "type": "trade",
        "balance": "499999894616.1302471000"
      },
      {
        "currency": "etc",
        "type": "frozen",
        "balance": "9786.6783000000"
      }
     {
        "currency": "eth",
        "type": "trade",
        "balance": "499999894616.1302471000"
      },
      {
        "currency": "eth”,
        "type": "frozen",
        "balance": "9786.6783000000"
      }
    ],
    "user-id": 1000
  }
}
```

## Trade API

#### POST /v1/order/orders/place  Make an order

Request

| Param   | required  | data type      | description    | default  | value range    |
| ----- | ----- | ------ | --------  | ---- | -------  |
| account-id | true  | string |  accountid in ‘spot’ for spot trade, ‘margin’ for margin trade |      |     |
| amount     | true  | string | Amount in buy-limit or sell-limitorder. It presents how much quote-currency you trade in buy-market order, and how many base-currency you trade in sell-market order |   |   |
| price      | false | string | don't use this param in buy-market or sell-market order   |      |       |
| source     | false | string |  'api' for spot trade and 'margin-api' for margin trade |    |
| symbol     | true  | string | trading asset    |      | btcusdt, bccbtc, rcneth ...   |
| type       | true  | string |      |    | buy-market , sell-market , buy-limit , sell-limit , buy-ioc, sell-ioc,  buy-limit-maker, sell-limit-maker |

**buy-limit-maker**

When a user place a buy_limit_maker order, if the order price >= the actual top sell price, the order will be submitted and then be rejected by the system. If the order price < the actual top sell price, then the order will be submitted and accepted by the system.
 
**sell-limit-maker**

When a user place a sell_limit_maker order, if the order price <= the actual top buy price, the order will be submitted and then be rejected by the system. If the order price > the actual top buy price, then the order will be submitted and accepted by the system.


Response:

| Param | required | data type  | description   | value range |
| ---- | ---- | ---- | ---- | ---- |
| data | false | string | order id  |      |

Example:

```
/* POST /v1/order/orders/place {
   "account-id": "100009",
   "amount": "10.1",
   "price": "100.1",
   "source": "api",
   "symbol": "ethusdt",
   "type": "buy-limit"
 } */
{
  "status": "ok",
  "data": "59378"
}
```

####  GET /v1/order/openOrders Provide open orders of a symbol for an account

#### Request parameter(s): 

`When neither account-id nor symbol defined in the request, the system will return all open orders (max 500) for all symbols and all accounts of the user, sorted by order ID in descending`

| Parameter     | Required | Data Type    | Description      | default value  | Value Range |
| -------- | ---- | ------ | ------------ | ---- | ---- |
| account-id | true | string | Account ID |      |  |
| symbol | true | string | Instrument symbol |      |    |
| side | false | string | The side of the orders |      |   “buy” or “sell”, if no side defined, will return all open orders of the account  |
| size | false | int | The number of records to return |   10   |    [0,500]  |

####  Response: 

| Parameter | Required | Data Type   | Description    | Value Range |
| ---- | ---- | ------ | ----- | ---- |
| id | true | long | Order ID |  |
| symbol| true | string | Instrument symbol |     |
| account-id| true | string | Account ID |     |
| price | true | string | Order price |    |
| created-at | true | int | Order Created Time (ms) |  |
| type | true | string | Order type |  buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc  |
| filled-amount | true | string | The order filled amount. When an order is partially matched, this field will be the filled amount. And amount-filledAmount would be the order remaining amount. |  For the orders not partial filled, the value will be 0.   |
| filled-cash-amount | true | string | The order filled value. (filled amount*order price) |  For the orders not partial filled, the value will be 0.   |
| filled-fees | true | string | The fee for the filled amount. |  For the orders not partial filled, the value will be 0.  |
| source | true | string | Order source type |  sys, web, api, app  |
| state | true | string | Current Order state |  Submitted, partial-filled, cancelling. ** Cancelling is an interim state between cancel request submitted and cancel confirmed. Once cancellation confirmed, the order is no longer an ‘open’ order, so will not be included in the response.  |

####  Response Sample:

```json
/* GET /v1/orders/openOrders */
{
  "status": "ok",
  "data": [
    {
      "id": 5454937,
      "symbol": "ethusdt",
      "account-id": 30925,
      "amount": "1.000000000000000000",
      "price": "0.453000000000000000",
      "created-at": 1530604762277,
      "type": "sell-limit",
      "filled-amount": "0.0",
      "filled-cash-amount": "0.0",
      "filled-fees": "0.0",
      "source": "web",
      "state": "submitted"
    }
  ]
}
```

####  POST /v1/order/orders/{order-id}/submitcancel  Request for cancelling an order

Request: 

| Param     | required | data type      | description           | default  | value range |
| -------- | ---- | ------ | ------------ | ---- | ---- |
| order-id | true | string | order id,filled in method path  |      |      |

Response: 

| Param | required | data type    | description    | value range |
| ---- | ---- | ------ | ----- | ---- |
| data | true | string | order id |      |

Example:

```
/* POST /v1/order/orders/{order-id}/submitcancel */
{
  "status": "ok",
  "data": "59378"
}
```

#### POST /v1/order/orders/batchcancel  Batch Cancel 

Request:

| Param  | required | data type    | description   | default  | value range |
| ---- | ---- | ---- | ----  | ---- | ---- |
| order-ids | true | list | order id list |  | maximum is 50 |

Response:

| Param | required  | data type  | description    | value range |
| ---- | ----- | ---- | ----- | ---- |
| data | false | map | status list |      |

Example:

```
/* POST /v1/order/orders/batchcancel */
{
  "order-ids": [
    "1", "2", "3"
  ]
}

-----

{
  "status": "ok",
  "data": {
    "success": [
      "1",
      "3"
    ],
    "failed": [
      {
        "err-msg": "记录无效",
        "order-id": "2",
        "err-code": "base-record-invalid"
      }
    ]
  }
}
```

####  POST  /v1/order/orders/batchCancelOpenOrders  Cancel a batch of orders with certain criteria

Request Parameter(s): 

| Parameter | Required | Data Type   | Description   | Default Value | Value Range |
| -------- | ---- | ------ | ------------ | ---- | ---- |
| account-id | true  | string | Account ID     |     |      | 
| symbol     | false | string | Instrument symbol  |      |   One instrument symbol. When no symbol defined, will return open orders of the account for all symbols.  |
| side | false | string | The side of the orders |      |   “buy” or “sell”, if no side defined, will return all open orders of the account  |
| size | false | int | The number of records to return |  100 |   [0,100]   |

Response: 

| Param     | required  | data type    | description   | value range     |
| ---- | ---- | ------ | ----- | ---- |
| success-count | true | int | Number of orders successfully cancelled. |     |
| failed-count | true | int | Number of orders failed in cancellation |     |
| next-id | true | long | The next order id which meets cancellation request criteria |    |

####  Response Example:

```json
/* GET /v1/order/orders/batchCancelOpenOrders */
{
  "status": "ok",
  "data": {
    "success-count": 2,
    "failed-count": 0,
    "next-id": 5454600
  }
}
```


####  GET /v1/order/orders/{order-id}    Get Order Info

Request: 

| Param     | required | data type   | description   | default  | value range |
| -------- | ---- | ------ | -----  | ---- | ---- |
| order-id | true | string | order id,filled in method path |      |      |

Response: 

| Param     | required  | data type    | description   | value range     |
| ----------------- | ----- | ------ | -------  | ----  |
| account-id        | true  | long   |      |       |
| amount            | true  | string |               |    |
| canceled-at       | false | long   |     |     |
| created-at        | true  | long   |    |   |
| field-amount      | true  | string |     |     |
| field-cash-amount | true  | string |       |      |
| field-fees        | true  | string |   |     |
| finished-at       | false | long   |      |     |
| id                | true  | long   |     |     |
| price             | true  | string |         |     |
| source            | true  | string |   | api |
| state             | true  | string |     | pre-submitted , submitting , submitted , partial-filled , partial-canceled , filled, canceled |
| symbol            | true  | string | trading asset   | btcusdt, bccbtc, rcneth ... |
| type              | true  | string |      | buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc |


Example:

```
/* GET /v1/order/orders/{order-id} */
{
  "status": "ok",
  "data": {
    "id": 59378,
    "symbol": "ethusdt",
    "account-id": 100009,
    "amount": "10.1000000000",
    "price": "100.1000000000",
    "created-at": 1494901162595,
    "type": "buy-limit",
    "field-amount": "10.1000000000",
    "field-cash-amount": "1011.0100000000",
    "field-fees": "0.0202000000",
    "finished-at": 1494901400468,
    "user-id": 1000,
    "source": "api",
    "state": "filled",
    "canceled-at": 0,
    "exchange": "xxx",
    "batch": ""
  }
}
```

####  GET /v1/order/orders/{order-id}/matchresults  Get Order matchresult

Request:

| Param  | required | data type   | description  | default  | value range |
| -------- | ---- | ------ | -----  | ---- | ---- |
| order-id | true | string | order id,filled in method path |      |      |

Response:

| Param    | required | data type    | description   | value range     |
| ------------- | ---- | ------ | -------- | -------- |
| created-at    | true | long   |      |    |
| filled-amount | true | string |     |    |
| filled-fees   | true | string |     |     |
| id            | true | long   |   |     |
| match-id      | true | long   |    |     |
| order-id      | true | long   |    |      |
| price         | true | string |   |    |
| source        | true | string |   | api      |
| symbol        | true | string | trading asset   | btcusdt, bccbtc, rcneth ...  |
| type          | true | string |     | buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc |

Example:

```
/* GET /v1/order/orders/{order-id}/matchresults */
{
  "status": "ok",
  "data": [
    {
      "id": 29553,
      "order-id": 59378,
      "match-id": 59335,
      "symbol": "ethusdt",
      "type": "buy-limit",
      "source": "api",
      "price": "100.1000000000",
      "filled-amount": "9.1155000000",
      "filled-fees": "0.0182310000",
      "created-at": 1494901400435
    }
  ]
}
```

####  GET /v1/order/orders  Get Order list

Request:

| Param   | required  | data type      | description   | default  | value range   |
| ---------- | ----- | ------ | ------  | ---- | ----  |
| symbol     | true  | string | trading asset      |      |btcusdt, bccbtc, rcneth ...  |
| types      | false | string | separate by ',' |      | buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc |
| start-date | false | string | yyyy-mm-dd |      |      |
| end-date   | false | string |  yyyy-mm-dd |      |    |
| states     | true  | string | separate by ','  |      | pre-submitted, submitted , partial-filled , partial-canceled , filled , canceled |
| from       | false | string |    |      |    |
| direct     | false | string |    |      | prev  ,next      |
| size       | false | string |        |      |         |

Response: 

| Param    | required  | data type    | description   | value range   |
| ----------------- | ----- | ------ | ----------------- | ----  |
| account-id        | true  | long   |    |     |
| amount            | true  | string |    |   |
| canceled-at       | false | long   |   |    |
| created-at        | true  | long   |  |    |
| field-amount      | true  | string |    |    |
| field-cash-amount | true  | string |    |    |
| field-fees        | true  | string |   |       |
| finished-at       | false | long   |      |   |
| id                | true  | long   |      |    |
| price             | true  | string |    |    |
| source            | true  | string |     | api  |
| state             | true  | string |      | pre-submitted , submitting , submitted , partial-filled , partial-canceled , filled, canceled |
| symbol            | true  | string | trading asset    | btcusdt, bccbtc, rcneth ... |
| type              | true  | string |     | submit-cancel  ,buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc |


Example:

```
/* GET /v1/order/orders */
{
  "status": "ok",
  "data": [
    {
      "id": 59378,
      "symbol": "ethusdt",
      "account-id": 100009,
      "amount": "10.1000000000",
      "price": "100.1000000000",
      "created-at": 1494901162595,
      "type": "buy-limit",
      "field-amount": "10.1000000000",
      "field-cash-amount": "1011.0100000000",
      "field-fees": "0.0202000000",
      "finished-at": 1494901400468,
      "user-id": 1000,
      "source": "api",
      "state": "filled",
      "canceled-at": 0,
      "exchange": "xxx",
      "batch": ""
    }
  ]
}
```

####  GET /v1/order/matchresults    Get order matchresults

Request:

| Param   | required  | data type   | description   | default  | value range    |
| ---------- | ----- | ------ | ------ | ---- | ----------- |
| symbol     | true  | string | trading asset   | btcusdt, bccbtc, rcneth ... |    |
| types      | false | string | separate by ','   |   All   | buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc |
| start-date | false | string |  yyyy-mm-dd |   -61 days   | [-61day, now] |
| end-date   | false | string |  yyyy-mm-dd |   Now   |  [start-date, now]   |
| from       | false | string |    |  max ID(order record ID)   |     |
| direct     | false | string |    |   next   | prev ,next   |
| size       | false | string |    |   100   |   <=100   |

Response: 

| Param   | required | data type    | description   | value range   |
| ------------- | ---- | ------ | -------- | ------- |
| created-at    | true | long   |    |    |
| filled-amount | true | string |    |    |
| filled-fees   | true | string |     |    |
| id            | true | long   |   |    |
| match-id      | true | long   |  |    |
| order-id      | true | long   |  |    |
| price         | true | string |    |    |
| source        | true | string |    | api   |
| symbol        | true | string | trading asset      | btcusdt, bccbtc, rcneth ...  |
| type          | true | string |      | buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc |

Example:

```
/* GET /v1/orders/matchresults */
{
  "status": "ok",
  "data": [
    {
      "id": 29555,
      "order-id": 59378,
      "match-id": 59335,
      "symbol": "ethusdt",
      "type": "buy-limit",
      "source": "api",
      "price": "100.1000000000",
      "filled-amount": "0.9845000000",
      "filled-fees": "0.0019690000",
      "created-at": 1494901400487
    }
  ]
}
```


## MARGIN API (Please fill in the field 'source' in method '/v1/order/orders/place' with 'margin-api', and the field 'accountid' should be a 'margin' accountid queried in method '/v1/account/accounts')

` only usdt-trading assets(except btc) are supported  `

#### POST /v1/dw/transfer-in/margin  Transfer asset from spot account to margin account
#### POST /v1/dw/transfer-out/margin  Transfer asset from margin account to spot account

Request

| Param  | required  | data type      | description     | default  | value range |
| ----- | ----- | ------ | ----- | ---- | -------- |
| symbol | true  | string | trading asset like 'btcusdt'  |      |      |
| currency  | true  | string |   |      |    |
| amount      | true | string |      |      |    |


Response:

| Param | required | data type  | description   | value range |
| ---- | ---- | ---- | ---- | ---- |
| data | true | long | transfer id |      |

Example:

```
/* POST /v1/dw/transfer-in/margin
{
  "symbol": "ethusdt",
  "currency": "eth",
  "amount": "1.0"
} */
{
  "status": "ok",
  "data": 1000
}
```


#### POST /v1/margin/orders  Margin application

Request

| Param   | required  | data type      | description    | default  | value range  |
| ----- | ----- | ------ |  --------------- | ---- | -------- |
| symbol | true  | string |    |      |      |
| currency  | true  | string |   |      |    |
| amount  | true | string |        |      |   |


Response:

| Param | required | data type  | description   | value range |
| ---- | ---- | ---- | ---- | ---- |
| data | true | long | margin order id |      |

Example:

```
/* POST /v1/margin/orders {
   "amount": "10.1",
   "symbol": "ethusdt",
   "currency": "eth"
} */
{
  "status": "ok",
  "data": 59378
}
```


#### POST /v1/margin/orders/{order-id}/repay     Repayment

Request

| Param       | required  | data type      | description   | default  | value range   |
| ----- | ----- | ------ | -----  | ---- | --------- |
| order-id | true  | long | margin order id, filled in method path  |      |      |
| amount   | true | string |   |      |       |


Response:

| Param | required | data type  | description   | value range |
| ---- | ---- | ---- | ---- | ---- |
| data | true | long | margin order id  |      |

Example:

```
/* POST /v1/margin/orders/59378/repay {
   "amount": "10.1"
}*/
{
  "status": "ok",
  "data": 59378
}
```


#### GET /v1/margin/loan-orders  Margin order list

Request

| Param       | required  | data type      | description    | default  | value range   |
| ----- | ----- | ------ |  -------  | ---- |  ----  |
| symbol | true | string |    |  |  |
| start-date | false | string |  yyyy-mm-dd  |     |    |
| end-date | false | string |  yyyy-mm-dd  |    |    |
| states | false | string |   |     |   |
| from   | false | string |    |    |     |
| direct | false | string |      |    | prev  ,next  |
| size   | false | string |    |    |     |


Response:

| Param | required | data type  | description | value range |
|-----|-----|-----|-----|------|
|   id  |  true  |  long  |    | |
|   user-id  |  true  |  long  |  | |
|   account-id  |  true  |  long  |    | |
|   symbol  |  true  |  string  |    | |
|   currency  |  true  |  string  |    | |
| loan-amount | true |string |   | |
| loan-balance | true | string |   | |
| interest-rate | true | string |   | |
| interest-amount | true | string |   | |
| interest-balance | true | string |   | |
| created-at | true | long |   | |
| accrued-at | true | long |   | |
| state | true | string |   |created  ,accrual ,cleared ,invalid |

Example:

```
/* GET /v1/margin/orders?symbol=btcusdt

*/
{
    "status": "ok",
    "data": [
        {
            "currency": "btc",
            "symbol": "btcusdt",
            "accrued-at": 1511760873587,
            "loan-amount": "0.333000000000000000",
            "loan-balance": "0.333000000000000000",
            "interest-balance": "0.000000000000000000",//unrepaid
            "created-at": 1511762673587,
            "interest-amount": "0.000000000000000000",
            "interest-rate": "0.000000000000000000",
            "account-id": 18298,
            "user-id": 111899,
            "updated-at": 1511762673654,
            "id": 232,
            "state": "accrual"
        }
      ]
}

```


#### GET /v1/margin/accounts/balance?symbol={symbol}  Margin account info

Request

| Param | required | data type  | description | default | value range |
|-----|-----|-----|-----|-----|-----|
| symbol | false | string |  filled in method path   |  |  |


Response:

| Param | required | data type  | description | value range |
|-----|-----|-----|-----|------|
| symbol  |  true  |  string  |  trading asset | |
| state  |  true  |  string  |    |working,fl-sys,fl-mgt,fl-end |
| risk-rate | true | object |   | |
| fl-price | true | string |   | |
| list | true | array | subaccount list   | |

Example:

```
/* GET /v1/margin/accounts/balance?symbol=btcusdt

*/
{
    "status": "ok",
    "data": [
        {
            "id": 18264,
            "type": "margin",
            "state": "working",
            "symbol": "btcusdt",
            "fl-price": "0",
            "fl-type": "safe",
            "risk-rate": "475.952571086994250554",
            "list": [
                {
                    "currency": "btc",
                    "type": "trade",
                    "balance": "1168.533000000000000000"
                },
                {
                    "currency": "btc",
                    "type": "frozen",
                    "balance": "0.000000000000000000"
                },
                {
                    "currency": "btc",
                    "type": "loan",
                    "balance": "-2.433000000000000000"
                },
                {
                    "currency": "btc",
                    "type": "interest",
                    "balance": "-0.000533000000000000"
                },
                {
                    "currency": "usdt",
                    "type": "trade",
                    "balance": "1313.534000000000000000"
                },
                {
                    "currency": "usdt",
                    "type": "frozen",
                    "balance": "0.000000000000000000"
                },
                {
                    "currency": "usdt",
                    "type": "loan",
                    "balance": "-140.234099999999999920"
                },
                {
                    "currency": "usdt",
                    "type": "interest",
                    "balance": "-0.931206660000000000"
                },
                {
                    "currency": "btc",
                    "type": "transfer-out-available",
                    "balance": "1163.872174670000000000"
                },
                { "currency": "usdt",
                    "type": "transfer-out-available",
                    "balance": "1313.534000000000000000"
                },
                {
                    "currency": "btc",
                    "type": "loan-available",
                    "balance": "8161.876538350676000000"
                },
                {
                    "currency": "usdt",
                    "type": "loan-available",
                    "balance": "49859.765900000000000080"
                }
            ]
        }
    ]
}

```



<br>
<br>
<br>
<br>
<br>
