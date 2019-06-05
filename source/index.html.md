---
title: WebSocket API

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - 
includes:

search: True
---


# General

### SSL Websocket Connection

> wss://{HOST}/ws

### Host structure

> Authenticated APIs:www.xxxx.com

> Public APIs: www.xxxx.com/api

### Data

All return data of websocket APIs needs to be unzipped.

Recommend: [pako](https://github.com/nodeca/pako)

### Library

Recommend: [ws](https://github.com/websockets/ws) by Node.js

### Topic

Requesting and subscribing to the data need to use the `topic`

| type      | topic                     | description                                       |
| ------------- | ---------------------------- | ---------------------------------------- |
| KLine         | market.$symbol.kline.$period | $period ：{ 1min, 5min, 15min, 30min, 60min, 1day, 1mon, 1week, 1year } |
| Market Depth  | market.$symbol.depth.$type   | $type ：{ step0, step1, step2, step3, step4, step5 } （depths 0-5）|
| Trade Detail  | market.$symbol.trade.detail  |                                          |
| Market Detail | market.$symbol.detail        |                                          |
| Market Tickers | market.tickers        |
`$symbol` ： { ethbtc, ltcbtc, etcbtc, bccbtc ... }


### Heartbeat

```
//WebSocket Client Request
{
    "ping": 18212558000
}

//WebSocket Server response
{
    "pong": 18212558000
}
```

If the type of request message is not `LONG`, websocket server will response:

```
{
  "ts": 1492420473027,
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "invalid ping"
}
```

### Request


```
//request
{
  "req": "topic to req",
  "id": "id generate by client"
}
```
`"req"` use the **topic**,as above "Topic"

**correct example**

```
//request
{
  "req": "market.ethbtc.kline.1min",
  "id": "id10"
}
```


```
//response
{
  "status": "ok",
  "rep": "market.btccny.kline.1min",
  "tick": [
    {
      "amount": 1.6206,
      "count":  3,
      "id":     1494465840,
      "open":   9887.00,
      "close":  9885.00,
      "low":    9885.00,
      "high":   9887.00,
      "vol":    16021.632026
    },
    {
      "amount": 2.2124,
      "count":  6,
      "id":     1494465900,
      "open":   9885.00,
      "close":  9880.00,
      "low":    9880.00,
      "high":   9885.00,
      "vol":    21859.023500
    }
  ]
}
```

**error request**

```
//request
{
  "req": "market.invalidsymbo.kline.1min",
  "id": "id10"
}
```

```
//response
{
  "status": "error",
  "id": "id10",
  "err-code": "bad-request",
  "err-msg": "invalid topic market.invalidsymbol.trade.detail",
  "ts": 1494483996521
}
```


### Subscribe

To receive data you have to send a "sub" message first.

```
//request
{
  "sub": "topic to sub",
  "id": "id generate by client"
}
```

**correct sample**


```
//request
{
  "sub": "market.btccny.kline.1min",
  "id": "id1"
}
```

`"req"` use the **topic**,as above "Topic".

```
//response
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.btccny.kline.1min",
  "ts": 1489474081631
}
```

After subscribe,you will receive updates upon any change.

```
//example
{
  "ch": "market.btccny.kline.1min",
  "ts": 1489474082831,
  "tick": {
    "id": 1489464480,
    "amount": 0.0,
    "count": 0,
    "open": 7962.62,
    "close": 7962.62,
    "low": 7962.62,
    "high": 7962.62,
    "vol": 0.0
  }
}
```


**error example**

wrong symbol


```
//response
{
  "id": "id2",
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "invalid topic market.invalidsymbol.kline.1min",
  "ts": 1494301904959
}
```

wrong topic

```
//response
{
  "id": "id3",
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "invalid topic market.btccny.kline.3min",
  "ts": 1494310283622
}
```

### unsubscribe

To stop receiving data from a channel you have to send a "unsubscribe" message.

```
//request
{
  "unsub": "topic to unsub",
  "id": "id generate by client"
}
```

**correct sample**

```
//request
{
  "unsub": "market.btccny.trade.detail",
  "id": "id4"
}
```

```
//response
{
  "id": "id4",
  "status": "ok",
  "unsubbed": "market.btccny.trade.detail",
  "ts" 1494326028889
}
```

**error unsubscribe**

Unsubscribe an unsubscribed topic

```
//response
{
  "id": "id5",
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "unsub with not subbed topic market.btccny.trade.detail",
  "ts": 1494326217428
}
```

Unsubscribe a nonexistent topic


```
//response
{
  "id": "id5",
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "unsub with not subbed topic not-exists-topic",
  "ts": 1494326318809
}
```

# Reference

## Candlestick Data

**market.$symbol$.kline.$period$**

| key         | necessary  | type     | description   | value |
| --- | ---  | --- | --- | --- |
| symbol | true  | string | Pairs | ethbtc, ltcbtc...|
| period | true | string | KLine period | 1min, 5min, 15min, 30min, 60min, 1day, 1mon, 1week, 1year |


```
//request
{
  "sub": "market.symbol.kline.period",
  "id": "id generate by client"
}
```


```
//request
{
  "sub": "market.ethbtc.kline.1min",
  "id": "id1"
}
```

```
//response
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.ethbtc.kline.1min",
  "ts": 1489474081631
}
```

After subscribe,you will receive updates upon any change.

```
{
  "ch": "market.ethbtc.kline.1min",
  "ts": 1489474082831,
  "tick": {
    "id": 1489464480,
    "amount": 0.0,
    "count": 0,
    "open": 7962.62,
    "close": 7962.62,
    "low": 7962.62,
    "high": 7962.62,
    "vol": 0.0
  }
}
```

## Request KLine 

### market.$symbol$.kline.$period$  


```
//request
{
  "req": "market.$symbol.kline.$period",
  "id": "id generated by client",
  "from": "$optional",
  "to": "$optional"
}
```

`"from"` `"to"` optional

time range:[t2,t5]

* from: t1, to: t5, return [t1, t5].
* from: t5, to: t1, which t5 > t1, return [].
* from: t5, return [t5].
* from: t3, return [t3, t5].
* to: t5, return [t1, t5].
* from: t which t3 < t <t4, return [t4, t5].
* to: t which t3 < t <t4, return [t1, t3].
* from: t1 and to: t2, should satisfy 1325347200 < t1 < t2 < 2524579200.

#### example

```
//request
{
  "req": "market.ethbtc.kline.1min",
  "id": "id10"
}
```

```
//response
{
  "rep": "market.ethbtc.kline.1min",
  "status": "ok",
  "id": "id10",
  "tick": [
    {
      "amount": 17.4805,
      "count":  27,
      "id":     1494478080,
      "open":   10050.00,
      "close":  10058.00,
      "low":    10050.00,
      "high":   10058.00,
      "vol":    175798.757708
    },
    {
      "amount": 15.7389,
      "count":  28,
      "id":     1494478140,
      "open":   10058.00,
      "close":  10060.00,
      "low":    10056.00,
      "high":   10065.00,
      "vol":    158331.348600
    },
    // more KLine data here
  ]
}
```

## Subscribe Market Depth 

market.$symbol$.depth.$type$ 

```
{
  "sub": "market.$symbol.depth.$type",
  "id": "id generated by client"
}
```
| key | necessary| type | description | value |
| --- | --- | --- | --- | --- |
| symbol | true  | string | Pairs | ethbtc, ltcbtc, etcbtc, bccbtc... |
| type | true| string | Market depth | step0, step1, step2, step3, step4, step5 |

**example**

```
//request
{
  "sub": "market.ethbtc.depth.step0",
  "id": "id1"
}
```

```
//response
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.ethbtc.depth.step0",
  "ts": 1489474081631
}
```

After subscribe,you will receive updates upon any change.

```
{
  "ch": "market.ethbtc.depth.step0",
  "ts": 1489474082831,
  "data": [
    "bids": [
    [9999.3900,0.0098], // [price, amount]
    [9992.5947,0.0560],
    // more Market Depth data here
    ]
    "asks": [
    [10010.9800,0.0099]
    [10011.3900,2.0000]
    //more data here
    ]
  ]
  }
}
```

`data` description

```
  "data": [
    "bids": [
    [buy1 price,buy1 volume]
    [buy2 price,buy2 volume]
    //more data here
    ]
    "asks": [
    [sell1 price,sell1 volume]
    [sell2 price,sell2 volume]
    //more data here
    ]
  ]
```

## Request Market Depth 

market.$symbol$.depth.$type$ 

```
{
  "req": "market.$symbol.depth.$type",
  "id": "id generated by client"
}
```

**example**

```
//request
{
  "req": "market.ethbtc.depth.step0",
  "id": "id10"
}
```

```
//response
{
  "rep": "market.ethbtc.depth.step0",
  "status": "ok",
  "id": "id10",
  "data": [
    "bids": [
    [9999.3900,0.0098], // [price, amount]
    [9992.5947,0.0560],
    // more Market Depth data here
    ]
    "asks": [
    [10010.9800,0.0099]
    [10011.3900,2.0000]
    //more data here
    ]
  ]
}
```

## Subscribe Trade Detail

 market.$symbol$.trade.detail  

```
//request
{
  "sub": "market.ethbtc.trade.detail",
  "id": "id1"
}
```

```
//response
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.ethbtc.trade.detail",
  "ts": 1489474081631
}
```

After subscribe,you will receive updates upon any change.

```
{
  "ch": "market.ethbtc.trade.detail",
  "ts": 1489474082831,
  "data": [
            {
                "amount": 0.0099,
                "ts": 1533265950234,
                "id": 146507451359183894799,
                "price": 401.74,
                "direction": "buy"
            },
            // more Trade Detail data here
        ]
}
```


## Request Trade Detail

 market.$symbol$.trade.detail 

```
//request
{
  "req": "market.$symbol.trade.detail",
  "id": "id generated by client"
}
```

Max data amount: 300

**example**

```
//request
{
  "req": "market.ethbtc.trade.detail",
  "id": "id11"
}
```

```
//response
{
  "rep": "market.ethbtc.trade.detail",
  "status": "ok",
  "id": "id11",
  "data": [
    {
      "id":        601595424,
      "price":     10195.64,
      "time":      1494495766,
      "amount":    0.2943,
      "direction": "buy",
      "tradeId":   601595424,
      "ts":        1494495766000
    },
    {
      "id":        601595423,
      "price":     10195.64,
      "time":      1494495711,
      "amount":    0.2430,
      "direction": "buy",
      "tradeId":   601595423,
      "ts":        1494495711000
    },
    // more Trade Detail data here
  ]
}
```

## Request Market Detail

 market.$symbol$.detail 
 
```
//request
{
  "req": "market.$symbol.detail",
  "id": "id generated by client"
}
```

Response recent Market Detail

**example**

```
//request
{
  "req": "market.ethbtc.detail",
  "id": "id12"
}
```

```
//response
{
  "rep": "market.ethbtc.detail",
  "status": "ok",
  "id": "id12",
  "tick": {
    "amount": 12224.2922,
    "open":   9790.52,
    "close":  10195.00,
    "high":   10300.00,
    "ts":     1494496390000,
    "id":     1494496390,
    "count":  15195,
    "low":    9657.00,
    "vol":    121906001.754751
  }
}
```








<br>
<br>
<br>
<br>
<br>
