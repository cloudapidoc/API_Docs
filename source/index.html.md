---
title:  API 文档

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://www.hbg.com/zh-cn/apikey/'>创建 API Key </a>
includes:

search: False
---

# 简介

## API 简介

API实现程序化交易。

通过API可以实现以下功能：

- 市场行情信息查询（K线、深度、实时成交、24小时行情）
- 账户资产信息查询
- 下单、撤单操作
- 订单信息查询


# 接入说明


## 限频规则

- 现货 / 杠杆（api.huobi.pro）：10秒100次

<aside class="notice">
单个 API Key 维度限制。行情 API 访问无需签名。
</aside>


## 安全认证

目前关于apikey申请和修改，请在“账户 - API管理”页面进行相关操作。其中AccessKey为API 访问密钥，SecretKey为用户对请求进行签名的密钥（仅申请时可见）。

**重要提示：这两个密钥与账号安全紧密相关，无论何时都请勿向其它人透露。**

### 合法请求结构

- 基于安全考虑，除行情API 外的 API 请求都必须进行签名运算。一个合法的请求由以下几部分组成：

  - 方法请求地址 即访问服务器地址：HOST+/api+方法名，比如{HOST}/api/v1/order/orders。

  - Host的常用格式：www.xxxx.com 如果有host格式相关问题，请咨询Host提供方）

  - API 访问密钥（AccessKeyId） 您申请的 APIKEY 中的AccessKey。

  - 签名方法（SignatureMethod） 用户计算签名的基于哈希的协议，此处使用 HmacSHA256。

  - 签名版本（SignatureVersion） 签名协议的版本，此处使用2。

  - 时间戳（Timestamp） 您发出请求的时间 (UTC 时区) (UTC 时区) (UTC 时区) 。在查询请求中包含此值有助于防止第三方截取您的请求。如：2017-05-11T16:22:06。再次强调是 (UTC 时区) 。

  - 必选和可选参数 每个方法都有一组用于定义 API 调用的必需参数和可选参数。可以在每个方法的说明中查看这些参数及其含义。 请一定注意：对于GET请求，每个方法自带的参数都需要进行签名运算； 对于POST请求，每个方法自带的参数不进行签名认证，即POST请求中需要进行签名运算的只有AccessKeyId、SignatureMethod、SignatureVersion、Timestamp四个参数，其它参数放在body中。

  - 签名 签名计算得出的值，用于确保签名有效和未被篡改。

    例：
    
```
https://{HOST}/api/v1/order/orders?
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
&SignatureMethod=HmacSHA256
&SignatureVersion=2
&Timestamp=2017-05-11T15%3A19%3A30
&order-id=1234567890
&Signature=calculated value
```

## 签名运算

API 请求在通过 Internet 发送的过程中极有可能被篡改。为了确保请求未被更改，我们会要求用户在每个请求中带上签名（行情 API 除外），来校验参数或参数值在传输途中是否发生了更改。

### 计算签名所需的步骤

规范要计算签名的请求 因为使用 HMAC 进行签名计算时，使用不同内容计算得到的结果会完全不同。所以在进行签名计算前，请先对请求进行规范化处理。下面以查询某订单详情请求为例进行说明：

查询某订单详情

```
https://{HOST}/api/v1/order/orders?
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
&SignatureMethod=HmacSHA256
&SignatureVersion=2
&Timestamp=2017-05-11T15:19:30
&order-id=1234567890
```

#### 1. 请求方法（GET 或 POST），后面添加换行符 “\n”

`GET\n`

#### 2. 添加小写的访问地址，后面添加换行符 “\n”

`
{host}\n
`

#### 3. 访问方法的路径，后面添加换行符 “\n”

`
/v1/order/orders\n
`

#### 4. 按照ASCII码的顺序对参数名进行排序。例如，下面是请求参数的原始顺序，进行过编码后


`AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx`

`order-id=1234567890`

`SignatureMethod=HmacSHA256`

`SignatureVersion=2`

`Timestamp=2017-05-11T15%3A19%3A30`

<aside class="notice">
使用 UTF-8 编码，且进行了 URI 编码，十六进制字符必须大写，如 “:” 会被编码为 “%3A” ，空格被编码为 “%20”。
</aside>
<aside class="notice">
时间戳（Timestamp）需要以YYYY-MM-DDThh:mm:ss格式添加并且进行 URI 编码。
</aside>


#### 5. 经过排序之后

`AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx`

`SignatureMethod=HmacSHA256`

`SignatureVersion=2`

`Timestamp=2017-05-11T15%3A19%3A30`

`order-id=1234567890`

#### 6. 按照以上顺序，将各参数使用字符 “&” 连接

`AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890`

#### 7. 组成最终的要进行签名计算的字符串如下

`GET\n`

`{host}\n`

`/v1/order/orders\n`

`AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890`


#### 8. 计算签名，将以下两个参数传入加密哈希函数： 要进行签名计算的字符串进行签名的密钥（SecretKey）

```
b0xxxxxx-c6xxxxxx-94xxxxxx-dxxxx
4F65x5A2bLyMWVQj3Aqp+B4w+ivaA7n5Oi2SuYtCJ9o=
```

#### 9.得到签名计算结果并进行 Base64编码

#### 10. 将上述值作为参数Signature的取值添加到 API 请求中。 将此参数添加到请求时，必须将该值进行 URI 编码。

最终，发送到服务器的 API 请求应该为

`https://{host}/v1/order/orders?AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&order-id=1234567890&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&Signature=4F65x5A2bLyMWVQj3Aqp%2BB4w%2BivaA7n5Oi2SuYtCJ9o%3D`

## 请求说明

1. 访问地址：将文档中的{HOST}替换为服务商的host
2. POST请求头信息中必须声明 Content-Type:application/json;GET请求头信息中必须声明 Content-Type:application/x-www-form-urlencoded。(汉语用户建议设置 Accept-Language:zh-cn)
3. 所有请求参数请按照 API 说明进行参数封装。
4. 将封装好参数的 API 请求通过 POST 或 GET 的方式提交到服务器。
5. 服务端处理请求，并返回相应的 JSON 格式结果。
6. 请使用 https 请求。
7. 限制频率（每个接口，只针对交易api，行情api不限制）为10秒100次。
8. 查询资产详情方法调用顺序：查询当前用户的所有账户->查询指定账户的余额



- [x] ### 返回内容格式


> 返回内容将会是以下格式:

```javascript
{
  "status": "ok",
  "ch": "market.btcusdt.kline.1day",
  "ts": 1499223904680,
  "data": //per API response data in nested JSON object
}
```

参数名称| 数据类型 | 描述
--------- | --------- | -----------
status    | string    | API接口返回状态
ch        | string    | 接口数据对应的数据流。部分接口没有对应数据流因此不返回此字段
ts        | int       | 接口返回的调整为北京时间的时间戳，单位毫秒
data      | object    | 接口返回数据主体

## 错误信息

### 行情 API 错误信息

| 错误码  |  描述 |
|-----|-----|
| bad-request | 错误请求 |
| invalid-parameter | 参数错误 |
| invalid-command | 指令错误 |
code 的具体解释, 参考对应的 `err-msg`.

### 交易 API 错误信息

| 错误码  |  描述 |
|-----|-----|
| base-symbol-error |  交易对不存在 |
| base-currency-error |  币种不存在 |
| base-date-error | 错误的日期格式 |
| account for id `12,345` and user id `6,543,210` does not exist| `account-id` 错误，请使用GET `/v1/account/accounts` 接口查询 |
| account-frozen-balance-insufficient-error | 余额不足 |
| account-transfer-balance-insufficient-error | 余额不足无法冻结 |
| bad-argument | 无效参数 |
| api-signature-not-valid | API签名错误 |
| gateway-internal-error | 系统繁忙，请稍后再试|
| ad-ethereum-addresss| 请输入有效的以太坊地址|
| order-accountbalance-error| 账户余额不足|
| order-limitorder-price-error|限价单下单价格超出限制 |
|order-limitorder-amount-error|限价单下单数量超出限制 |
|order-orderprice-precision-error|下单价格超出精度限制 |
|order-orderamount-precision-error|下单数量超过精度限制|
|order-marketorder-amount-error|下单数量超出限制|
|order-queryorder-invalid|查询不到此条订单|
|order-orderstate-error|订单状态错误|
|order-datelimit-error|查询超出时间限制|
|order-update-error|订单更新出错|

## 常见问题 Q & A

### 经常断线或者丢数据

* 请确认是否使用正确域名访问火币 云API
* 请使用日本云服务器

### 签名失败

* 检查 API Key 是否有效，是否复制正确，是否有绑定 IP 白名单
* 检查时间戳是否是 UTC 时间
* 检查参数是否按字母排序
* 检查编码
* 检查签名是否有 base64 编码
* 检查 GET 是否以表单方式提交
* 检查 POST 的 url 是否带着签名字段，POST 的数据格式是否是 json 格式
* 检查签名结果是否有进行 URI 编码

### 返回 login-required

* 检查参数 `account-id` 是否是由 GET `/v1/account/accounts` 接口返回的，而不是填的 UID
* 检查是否 POST 请求是否把业务参数也计算进签名
* 检查 GET 请求是否将参数按照 ASCII 码表顺序排序

### 返回 gateway-internal-error

* 检查 POST 请求是否在 header 中声明 Content-Type:application/json




# 基础信息

## 获取所有交易对

此接口返回所有火币全球站支持的交易对。

```shell
curl "https://{HOST}/v1/common/symbols"
```


### HTTP 请求

- GET `/v1/common/symbols`

### 请求参数

此接口不接受任何参数。

> Responds:

```json
  "data": [
    {
        "base-currency": "btc",
        "quote-currency": "usdt",
        "price-precision": 2,
        "amount-precision": 4,
        "symbol-partition": "main",
        "symbol": "btcusdt"
    }
    {
        "base-currency": "eth",
        "quote-currency": "usdt",
        "price-precision": 2,
        "amount-precision": 4,
        "symbol-partition": "main",
        "symbol": "ethusdt"
    }
  ]
```

### 返回字段

响应数据:

| 参数名称         | 是否必须 | 数据类型 | 描述                    | 取值范围                                      |
| :--------------- | :------- | :------- | :---------------------- | :-------------------------------------------- |
| base-currency    | true     | string   | 基础币种                |                                               |
| quote-currency   | true     | string   | 计价币种                |                                               |
| price-precision  | true     | string   | 价格精度位数（0为个位） |                                               |
| amount-precision | true     | string   | 数量精度位数（0为个位)  |                                               |
| symbol-partition | true     | string   | 交易区                  | main主区，innovation创新区，bifurcation分叉区 |

请求响应例子:

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

## 获取所有币种

此接口返回所有支持的币种。


```shell
curl "https://{HOST}/v1/common/currencys"
```

### HTTP 请求

- GET `/v1/common/currencys`


### 请求参数

此接口不接受任何参数。


> Response:

```json
  "data": [
    "usdt",
    "eth",
    "etc"
  ]
```

### 返回字段


<aside class="notice">返回的“data”对象是一个字符串数组，每一个字符串代表一个支持的币种。</aside>


## 获取当前系统时间

此接口返回当前的系统时间，时间是调整为北京时间的时间戳，单位毫秒。

```shell
curl "https://{HOST}/v1/common/timestamp"
```

### HTTP 请求

- GET `/v1/common/timestamp`

### 请求参数

此接口不接受任何参数。

> Response:

```json
  "data": 1494900087029
```

### 响应数据

返回的“data”对象是一个整数表示返回的调整为北京时间的时间戳，单位毫秒。

# 行情数据

## K 线数据（蜡烛图）

此接口返回历史K线数据。

### HTTP 请求

- GET `/market/history/kline`

```shell
curl "https://{HOST}/market/history/kline?period=1day&size=200&symbol=btcusdt"
```

### 请求参数

参数       | 数据类型 | 是否必须 | 默认值 | 描述 | 取值范围
--------- | --------- | -------- | ------- | ------ | ------
symbol    | string    | true     | NA      | 交易对 | btcusdt, ethbtc... 
period    | string    | true     | NA      | 返回数据时间粒度，也就是每根蜡烛的时间区间 | 1min, 5min, 15min, 30min, 60min, 1day, 1mon, 1week, 1year
size      | integer   | false    | 150     | 返回 K 线数据条数 | [1, 2000]

<aside class="notice">当前 REST API 不支持自定义时间区间，如需要历史固定时间范围的数据，请参考 Websocket API 中的 K 线接口。</aside>

<aside class="notice">获取 hb10 净值时， symbol 请填写 “hb10”。</aside>

> Response:

```json
[
  {
    "id": 1499184000,
    "amount": 37593.0266,
    "count": 0,
    "open": 1935.2000,
    "close": 1879.0000,
    "low": 1856.0000,
    "high": 1940.0000,
    "vol": 71031537.97866500
  }
]
```

### 响应数据

字段名称      | 数据类型 | 描述
--------- | --------- | -----------
id        | integer   | 调整为北京时间的时间戳，单位秒，并以此作为此K线柱的id
amount    | float     | 以基础币种计量的交易量
count     | integer   | 交易次数
open      | float     | 本阶段开盘价
close     | float     | 本阶段收盘价
low       | float     | 本阶段最低价
high      | float     | 本阶段最高价
vol       | float     | 以报价币种计量的交易量



## 聚合行情（Ticker）

此接口获取ticker信息同时提供最近24小时的交易聚合信息。

### HTTP 请求

- GET `/market/detail/merged`

```shell
curl "https://{HOST}/market/detail/merged?symbol=ethusdt"
```

### 请求参数

参数      | 数据类型   | 是否必须  | 默认值  | 描述 | 取值范围
--------- | --------- | -------- | ------- | ------| -----
symbol    | string    | true     | NA      | 交易对 | btcusdt, ethbtc 


> Response:

```json
{
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
```

### 响应数据

响应数据:

| 参数名称 | 是否必须 | 数据类型 | 描述                                                    | 取值范围       |
| :------- | :------- | :------- | :------------------------------------------------------ | :------------- |
| status   | true     | string   | 请求处理结果                                            | "ok" , "error" |
| ts       | true     | number   | 响应生成时间点，单位：毫秒                              |                |
| tick     | true     | object   | KLine 数据                                              |                |
| ch       | true     | string   | 数据所属的 channel，格式： market.$symbol.kline.$period |                |

字段名称      | 数据类型 | 描述
--------- | --------- | -----------
id        | integer   | NA
amount    | float     | 以基础币种计量的交易量
count     | integer   | 交易次数
open      | float     | 本阶段开盘价
close     | float     | 本阶段最新价
low       | float     | 本阶段最低价
high      | float     | 本阶段最高价
vol       | float     | 以报价币种计量的交易量
bid       | object    | 当前的最高卖价 [price, quote volume]
ask       | object    | 当前的最低买价 [price, quote volume]


## 所有交易对的最新 Tickers

获得所有交易对的 tickers，数据取值时间区间为24小时滚动。

<aside class="notice">此接口返回所有交易对的 ticker，因此数据量较大。</aside>

### HTTP 请求

- GET `/market/tickers`

```shell
curl "https://{HOST}/market/tickers"
```

注：当交易对尚未产生成交时，返回的数据里面 `open` `close` `high` `low` `amount``count` `vol` 的值都为 `null`

### 请求参数

此接口不接受任何参数。

> Response:

```javascript
[  
    {  
        "open":0.044297,      // 开盘价
        "close":0.042178,     // 收盘价
        "low":0.040110,       // 最高价
        "high":0.045255,      // 最低价
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
```

### 响应数据

核心响应数据为一个对象列，每个对象包含下面的字段

字段名称      | 数据类型   | 描述
--------- | --------- | -----------
amount    | float     | 以基础币种计量的交易量
count     | integer   | 交易笔数
open      | float     | 开盘价
close     | float     | 最新价
low       | float     | 最低价
high      | float     | 最高价
vol       | float     | 以报价币种计量的交易量
symbol    | string    | 交易对，例如btcusdt, ethbtc 

## 市场深度数据

此接口返回指定交易对的当前市场深度数据。

### HTTP 请求

- GET `/market/depth`

```shell
curl "https://{HOST}/market/depth?symbol=btcusdt&type=step2"
```

### 请求参数

参数      | 数据类型   | 必须     | 默认值 
--------- | --------- | -------- | ------
symbol    | string    | true     | NA    
depth     | integer   | false    | 20    
type      | string    | true     | step0 

<aside class="notice">当type值为‘step0’时，‘depth’的默认值为150而非20。 
    </aside>

<aside class="notice">用户选择“合并深度”时，一定报价精度内的市场挂单将予以合并显示。合并深度仅改变显示方式，不改变实际成交价格。</aside>

**参数type的各值说明**

取值      | 说明
--------- | ---------
step0     | 无聚合
step1     | 聚合度为报价精度*10
step2     | 聚合度为报价精度*100
step3     | 聚合度为报价精度*1000
step4     | 聚合度为报价精度*10000
step5     | 聚合度为报价精度*100000

> Response:

```javascript
{
    "version": 31615842081,
    "ts": 1489464585407,
    "bids": [
      [7964, 0.0678], // [price, amount]
      [7963, 0.9162],
      [7961, 0.1],
      [7960, 12.8898],
      [7958, 1.2],
      ...
    ],
    "asks": [
      [7979, 0.0736],
      [7980, 1.0292],
      [7981, 5.5652],
      [7986, 0.2416],
      [7990, 1.9970],
      ...
    ]
  }
```

### 响应数据

| 参数名称 | 是否必须 | 数据类型 | 描述                                                  | 取值范围          |
| :------- | :------- | :------- | :---------------------------------------------------- | :---------------- |
| status   | true     | string   | 请求处理结果                                          | "ok" 或者 "error" |
| ts       | true     | number   | 响应生成时间点，单位：毫秒                            |                   |
| tick     | true     | object   | Depth 数据                                            |                   |
| ch       | true     | string   | 数据所属的 channel，格式： market.$symbol.depth.$type |                   |

<aside class="notice">返回的JSON顶级数据对象名为'tick'而不是通常的'data'。</aside>

字段名称      | 数据类型    | 描述
--------- | --------- | -----------
ts        | integer   | 调整为北京时间的时间戳，单位毫秒
version   | integer   | 内部字段
bids      | object    | 当前的所有买单 [price, quote volume]
asks      | object    | 当前的所有卖单 [price, quote volume]

## 最近市场成交记录

此接口返回指定交易对最新的一个交易记录。

### HTTP 请求

- GET `/market/trade`

```shell
curl "https://{HOST}/market/trade?symbol=ethusdt"
```

### 请求参数

参数      | 数据类型   | 是否必须  | 默认值   | 描述
--------- | --------- | -------- | ------- | -----------
symbol    | string    | true     | NA      | 交易对，例如btcusdt, ethbtc 

> Response:

```json
{
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
```

### 响应数据

| 参数名称 | 是否必须 | 数据类型 | 描述                                                   | 取值范围          |
| :------- | :------- | :------- | :----------------------------------------------------- | :---------------- |
| status   | true     | string   | 请求处理结果                                           | "ok" 或者 "error" |
| ts       | true     | number   | 响应生成时间点，单位：毫秒                             |                   |
| tick     | true     | object   | Trade 数据                                             |                   |
| ch       | true     | string   | 数据所属的 channel，格式： market.$symbol.trade.detail |                   |

<aside class="notice">返回的JSON顶级数据对象名为'tick'而不是通常的'data'。</aside>

字段名称       | 数据类型 | 描述
--------- | --------- | -----------
id        | integer   | 唯一交易id
amount    | float     | 以基础币种为单位的交易量
price     | float     | 以报价币种为单位的成交价格
ts        | integer   | 调整为北京时间的时间戳，单位毫秒
direction | string    | 交易方向：“buy” 或 “sell”, “buy” 即买，“sell” 即卖

## 获得近期交易记录

此接口返回指定交易对近期的所有交易记录。

### HTTP 请求

- GET `/market/history/trade`

```shell
curl "https://{HOST}/market/history/trade?symbol=ethusdt&size=2"
```

### 请求参数

参数       | 数据类型  | 是否必须   | 默认值 | 描述
--------- | --------- | -------- | ------- | -----------
symbol    | string    | true     | NA      | 交易对，例如 btcusdt, ethbtc 
size      | integer   | false    | 1       | 返回的交易记录数量，最大值2000

> Response:

```json
[  
   {  
      "id":31618787514,
      "ts":1544390317905,
      "data":[  
         {  
            "amount":9.000000000000000000,
            "ts":1544390317905,
            "id":3161878751418918529341,
            "price":94.690000000000000000,
            "direction":"sell"
         },
         {  
            "amount":73.771000000000000000,
            "ts":1544390317905,
            "id":3161878751418918532514,
            "price":94.660000000000000000,
            "direction":"sell"
         }
      ]
   },
   {  
      "id":31618776989,
      "ts":1544390311353,
      "data":[  
         {  
            "amount":1.000000000000000000,
            "ts":1544390311353,
            "id":3161877698918918522622,
            "price":94.710000000000000000,
            "direction":"buy"
         }
      ]
   }
]
```

### 响应数据

据:

| 参数名称 | 是否必须 | 类型    | 描述                                                   | 取值范围  |      |
| :------- | :------- | :------ | :----------------------------------------------------- | :-------- | :--- |
| status   | true     | string  | 请求处理结果                                           | ok, error |      |
| ch       | true     | string  | 数据所属的 channel，格式： market.$symbol.trade.detail |           |      |
| ts       | true     | integer | 发送时间                                               |           |      |
| data     | true     | object  | 成交记录                                               |           |      |

<aside class="notice">返回的数据对象是一个对象数组，每个数组元素为一个调整为北京时间的时间戳（单位毫秒）下的所有交易记录，这些交易记录以数组形式呈现。</aside>

参数      | 数据类型 | 描述
--------- | --------- | -----------
id        | integer   | 唯一交易id
amount    | float     | 以基础币种为单位的交易量
price     | float     | 以报价币种为单位的成交价格
ts        | integer   | 调整为北京时间的时间戳，单位毫秒
direction | string    | 交易方向：“buy” 或 “sell”, “buy” 即买，“sell” 即卖

## 最近24小时行情数据

此接口返回最近24小时的行情数据汇总。

### HTTP 请求

- GET `/market/detail`

```shell
curl "https://{HOST}/market/detail?symbol=ethusdt"
```

### 请求参数

参数      | 数据类型 | 是否必须 | 默认值 | 描述
--------- | --------- | -------- | ------- | -----------
symbol    | string    | true     | NA      | 交易对，例如btcusdt, ethbtc 

> Response:

```json
{  
   "amount":613071.438479561,
   "open":86.21,
   "close":94.35,
   "high":98.7,
   "id":31619471534,
   "count":138909,
   "low":84.63,
   "version":31619471534,
   "vol":5.6617373443873316E7
}
```

### 响应数据

<aside class="notice">返回的JSON顶级数据对象名为'tick'而不是通常的'data'。</aside>

字段名称      | 数据类型   | 描述
--------- | --------- | -----------
id        | integer   | 响应id
amount    | float     | 以基础币种计量的交易量
count     | integer   | 交易次数
open      | float     | 本阶段开盘价
close     | float     | 本阶段收盘价
low       | float     | 本阶段最低价
high      | float     | 本阶段最高价
vol       | float     | 以报价币种计量的交易量
version   | integer   | 内部数据

# 账户相关

<aside class="notice">访问账户相关的接口需要进行签名认证。</aside>

## 账户信息 

API Key 权限：读取

查询当前用户的所有账户 ID `account-id` 及其相关信息

### HTTP 请求

- GET `/v1/account/accounts`

### 请求参数

无

> Response:

```json
{
  "data": [
    {
      "id": 100001,
      "type": "spot",
      "subtype": "",
      "state": "working"
    }
    {
      "id": 100002,
      "type": "margin",
      "subtype": "btcusdt",
      "state": "working"
    },
    {
      "id": 100003,
      "type": "otc",
      "subtype": "",
      "state": "working"
    }
  ]
}
```

### 响应数据

| 参数名称  | 是否必须 | 数据类型 | 描述 | 取值范围 |
| ----- | ---- | ------ | ----- | ----  |
| id    | true | long   | account-id |    |
| state | true | string | 账户状态  | working：正常, lock：账户被锁定 |
| type  | true | string | 账户类型  | spot：现货账户， margin：杠杆账户，otc：OTC 账户，point：点卡账户  |

<aside class="notice">杠杆账户（margin）会在第一次划转资产时创建，如果未划转过资产则不会有杠杆账户。</aside>

## 账户余额

API Key 权限：读取

查询指定账户的余额，支持以下账户：

spot：现货账户， margin：杠杆账户，otc：OTC 账户。

### HTTP 请求

- GET `/v1/account/accounts/{account-id}/balance`

### 请求参数

| 参数名称   | 是否必须 | 类型   | 描述   | 默认值  | 取值范围 |
| ---------- | ---- | ------ | --------------- | ---- | ---- |
| account-id | true | string | account-id，填在 path 中，可用 GET /v1/account/accounts 获取 |  |      |

> Response:

```json
{
  "data": {
    "id": 100009,
    "type": "spot",
    "state": "working",
    "list": [
      {
        "currency": "usdt",
        "type": "trade",
        "balance": "5007.4362872650"
      },
      {
        "currency": "usdt",
        "type": "frozen",
        "balance": "348.1199920000"
      }
    ],
    "user-id": 10000
  }
}
```

### 响应数据

| 参数名称  | 是否必须  | 数据类型   | 描述    | 取值范围   |
| ----- | ----- | ------ | ----- | ----- |
| id    | true  | long   | 账户 ID |      |
| state | true  | string | 账户状态  | working：正常  lock：账户被锁定 |
| type  | true  | string | 账户类型  | spot：现货账户， margin：杠杆账户，otc：OTC 账户 |
| list  | false | Array  | 子账号数组 |     |

list字段说明

| 参数名称   | 是否必须 | 数据类型   | 描述   | 取值范围    |
| -------- | ---- | ------ | ---- |  ------ |
| balance  | true | string | 余额   |    |
| currency | true | string | 币种   |    |
| type     | true | string | 类型   | trade: 交易余额，frozen: 冻结余额 |

## 资产划转（母子账号之间）

API Key 权限：交易

母账户执行母子账号之间的划转

### HTTP 请求

- POST ` /v1/subuser/transfer`

### 请求参数

参数|是否必填 | 数据类型 | 说明 | 取值范围 
-----------|------------|-----------|------------|----------
sub-uid	|true|	long|子账号uid	|-
currency|true|	string|币种	|-
amount|true|	decimal|划转金额|-
type|true|string|划转类型| master-transfer-in（子账号划转给母账户虚拟币）/ master-transfer-out （母账户划转给子账号虚拟币）/master-point-transfer-in （子账号划转给母账户点卡）/master-point-transfer-out（母账户划转给子账号点卡） 

> Response:

```json
{
  "data":123456,
  "status":"ok"
}
```

### 响应数据

参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 
-----------|------------|-----------|------------|----------|--
data | true| int | - | 划转订单id|   - 
status | true|   | - |  状态| "OK" or "Error"    

### 错误码

error_code|	说明|	类型
------------------|------------|-----------
account-transfer-balance-insufficient-error|	账户余额不足|	string
base-operation-forbidden|	禁止操作（母子账号关系错误时报）	|string

## 子账号余额（汇总）

API Key 权限：读取

母账户查询其下所有子账号的各币种汇总余额

### HTTP 请求

- GET `/v1/subuser/aggregate-balance`

### 请求参数

无

> Response:

```javascript
{
  "status": "ok",
  "data": [
      {
        "currency": "eos",
        "balance": "1954559.809500000000000000"
      },
      {
        "currency": "btc",
        "balance": "0.000000000000000000"
      },
      {
        "currency": "usdt",
        "balance": "2925209.411300000000000000"
      },
      ...
   ]
}
```

### 响应数据

参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 
-----------|------------|-----------|------------|----------|--
status | true|   | - |  状态| "OK" or "Error"    
data | true| list | - | |   - 

- data 

参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 
-----------|------------|-----------|------------|----------|--
currency|	是|	string|	-|	子账号币名|-
balance|	是|	string|	-|	子账号下该币种所有余额（可用余额和冻结余额的总和）|-

## 子账号余额

API Key 权限：读取

母账户查询子账号各币种账户余额

### HTTP 请求

- GET `/v1/account/accounts/{sub-uid}`

### 请求参数

参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 
-----------|------------|-----------|------------|----------|--
sub-uid|true|	long|	-|	子用户的 UID|-

> Response:

```json
{
  "status": "ok",
	"data": [
    {
      "id": 9910049,
      "type": "spot",
      "list": 
      [
        {
          "currency": "btc",
          "type": "trade",
          "balance": "1.00"
        },
        {
          "currency": "eth",
          "type": "trade",
          "balance": "1934.00"
        }
      ]
    },
    {
      "id": 9910050,
      "type": "point",
      "list": []
    }
	]
}
```

### 响应数据


参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 
-----------|------------|-----------|------------|----------|--
id|	-	|long|	-	|子账号 UID|-
type|	-	|string|	-	|账户类型| Spot：现货账户， 
list|	-	|object|	-	|-|-

- list
	
参数|是否必填 | 数据类型 | 长度 | 说明 | 取值范围 
-----------|------------|-----------|------------|----------|--
currency|	-	|string|	-	|币种	|-
type|	-	|string|	-	|账户类型	|trade：交易账户，frozen：冻结账户
balance|-|decimal|-		|账户余额	|-

# 钱包（充值与提现）

<aside class="notice">访问钱包相关的接口需要进行签名认证。</aside>

## 虚拟币提现

API Key 权限：提币

<aside class="notice">仅支持在官网上相应币种地址列表 </a> 中的地址。</aside>

### HTTP 请求

- POST ` /v1/dw/withdraw/api/create`

```json
{
  "address": "0xde709f2102306220921060314715629080e2fb77",
  "amount": "0.05",
  "currency": "eth",
  "fee": "0.01"
}
```

### 请求参数

| 参数名称       | 是否必须 | 类型     | 描述     |取值范围 |
| ---------- | ---- | ------ | ------ | ---- |
| address | true | string   | 提现地址 |仅支持在官网上相应币种  |
| amount     | true | string | 提币数量   |      |
| currency | true | string | 资产类型   |  btc, ltc, bch, eth, etc ...支持的币种) |
| fee     | false | string | 转账手续费  |     |
| chain   | false | string | 提 USDT-ERC20 时需要设置此参数为 "usdterc20"，其他币种提现不需要设置此参数  |     |
| addr-tag|false | string | 虚拟币共享地址tag，适用于xrp，xem，bts，steem，eos，xmr | 格式, "123"类的整数字符串|


> Response:

```json
{
  "data": 700
}
```

### 响应数据


| 参数名称 | 是否必须  | 数据类型 | 描述   | 取值范围 |
| ---- | ----- | ---- | ---- | ---- |
| data | false | long | 提现 ID |      |


## 取消提现

API Key 权限：提币

### HTTP 请求

- POST ` /v1/dw/withdraw-virtual/{withdraw-id}/cancel`

### 请求参数

| 参数名称        | 是否必须 | 类型   | 描述 | 默认值  | 取值范围 |
| ----------- | ---- | ---- | ------------ | ---- | ---- |
| withdraw-id | true | long | 提现 ID，填在 path 中 |      |      |


> Response:

```json
{
  "data": 700
}
```

### 响应数据


| 参数名称 | 是否必须  | 数据类型 | 描述    | 取值范围 |
| ---- | ----- | ---- | ----- | ---- |
| data | false | long | 提现 ID |      |

## 充提记录

API Key 权限：读取

查询充提记录

### HTTP 请求

- GET `/v1/query/deposit-withdraw`

### 请求参数

| 参数名称        | 是否必须 | 类型   | 描述 | 默认值  | 取值范围 |
| ----------- | ---- | ---- | ------------ | ---- | ---- |
| currency | false | string | 币种  |  |缺省时，返回所有币种 |
| type | true | string | 充值或提现 |     |  deposit 或 withdraw |
| from   | false | string | 查询起始 ID  |缺省时，默认值direct相关。当direct为‘prev’时，from 为1 ，从旧到新升序返回；当direct为’next‘时，from为最新的一条记录的ID，从新到旧降序返回    |     |
| size   | false | string | 查询记录大小  | 100   |1-500     |
| direct  | false | string | 返回记录排序方向  | 缺省时，默认为“prev” （升序）  |“prev” （升序）or “next” （降序）    |

> Response:

```javascript
{
  "data":
    [
      {
        "id": 1171,
        "type": "deposit",
        "currency": "xrp",
        "tx-hash": "ed03094b84eafbe4bc16e7ef766ee959885ee5bcb265872baaa9c64e1cf86c2b",
        "amount": 7.457467,
        "address": "rae93V8d2mdoUQHwBDBdM4NHCMehRJAsbm",
        "address-tag": "100040",
        "fee": 0,
        "state": "safe",
        "created-at": 1510912472199,
        "updated-at": 1511145876575
      },
      ...
    ]
}
```

### 响应数据


| 参数名称 | 是否必须 | 数据类型 | 描述 | 取值范围 |
|-----|-----|-----|-----|------|
|   id  |  true  |  long  |   | |
|   type  |  true  |  string  | 类型 | 'deposit', 'withdraw' |
|   currency  |  true  |  string  |  币种 | |
| tx-hash | true |string | 交易哈希 | |
| amount | true | long | 个数 | |
| address | true | string | 地址 | |
| address-tag | true | string | 地址标签 | |
| fee | true | long | 手续费 | |
| state | true | string | 状态 | 状态参见下表 |
| created-at | true | long | 发起时间 | |
| updated-at | true | long | 最后更新时间 | |


- 虚拟币充值状态定义：

|状态|描述|
|--|--|
|unknown|状态未知|
|confirming|确认中|
|confirmed|确认中|
|safe|已完成|
|orphan| 待确认|

- 虚拟币提现状态定义：

| 状态 | 描述  |
|--|--|
| submitted | 已提交 |
| reexamine | 审核中 |
| canceled  | 已撤销 |
| pass    | 审批通过 |
| reject  | 审批拒绝 |
| pre-transfer | 处理中 |
| wallet-transfer | 已汇出 |
| wallet-reject   | 钱包拒绝 |
| confirmed      | 区块已确认 |
| confirm-error  | 区块确认错误 |
| repealed       | 已撤销 |



# 现货 / 杠杆交易

<aside class="notice">访问交易相关的接口需要进行签名认证。</aside>

<aside class="warning">杠杆交易时，“account-id” 参数需设置为 “margin” 的 account-id， “source”参数需设置为 “margin-api”。</aside>

## 下单

API Key 权限：交易

发送一个新订单到火币以进行撮合。

### HTTP 请求

- POST ` /v1/order/orders/place`

```json
{
  "account-id": "100009",
  "amount": "10.1",
  "price": "100.1",
  "source": "api",
  "symbol": "ethusdt",
  "type": "buy-limit"
}
```

### 请求参数

参数名称 | 数据类型 | 是否必需 | 默认值 | 描述
---------  | --------- | -------- | ------- | -----------
account-id | string    | true     | NA      | 账户 ID，使用 GET /v1/account/accounts 接口查询。现货交易使用 ‘spot’ 账户的 account-id；杠杆交易，请使用 ‘margin’ 账户的 account-id
symbol     | string    | true     | NA      | 交易对, 例如btcusdt, ethbtc 
type       | string    | true     | NA      | 订单类型，包括buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc, buy-limit-maker, sell-limit-maker（说明见下文）
amount     | string    | true     | NA      | 订单交易量
price      | string    | false    | NA      | limit order的交易价格
source     | string    | false    | api     | 现货交易填写“api”，杠杆交易填写“margin-api”
client-order-id |string	|false |	NA	 |用户自编订单号（须在24小时内保持唯一性）

**buy-limit-maker**

当“下单价格”>=“市场最低卖出价”，订单提交后，系统将拒绝接受此订单；

当“下单价格”<“市场最低卖出价”，提交成功后，此订单将被系统接受。

**sell-limit-maker**

当“下单价格”<=“市场最高买入价”，订单提交后，系统将拒绝接受此订单；

当“下单价格”>“市场最高买入价”，提交成功后，此订单将被系统接受。

> Response:

```json
{  
  "data": "59378"
}
```

### 响应数据

返回的主数据对象是一个对应下单单号的字符串。

| 参数名称 | 是否必须 | 数据类型 | 描述   | 取值范围 |
| :------- | :------- | :------- | :----- | :------- |
| data     | false    | string   | 订单ID |          |

## 撤销订单

API Key 权限：交易

此接口发送一个撤销订单的请求。

<aside class="warning">此接口只提交取消请求，实际取消结果需要通过订单状态，撮合状态等接口来确认。</aside>


### HTTP 请求

- POST ` /v1/order/orders/{order-id}/submitcancel`


### 请求参数

| 参数名称     | 是否必须 | 类型     | 描述           | 默认值  | 取值范围 |
| -------- | ---- | ------ | ------------ | ---- | ---- |
| order-id | true | string | 订单ID，填在path中 |      |      |


> Response:

```json
{  
  "data": "59378"
}
```

### 响应数据

返回的主数据对象是一个对应下单单号的字符串。

| 参数名称 | 是否必须 | 数据类型 | 描述    | 取值范围 |
| :------- | :------- | :------- | :------ | :------- |
| data     | true     | string   | 订单 ID |          |


## 查询当前未成交订单

API Key 权限：读取

查询已提交但是仍未完全成交或未被撤销的订单。

```json
{
   "account-id": "100009",
   "amount": "10.1",
   "price": "100.1",
   "source": "api",
   "symbol": "ethusdt",
   "type": "buy-limit"
}
```

### HTTP 请求

- GET `/v1/order/openOrders`

### 请求参数

参数名称 | 数据类型 | 是否必需 | 默认值 | 描述
---------  | --------- | -------- | ------- | -----------
account-id | string    | true    | NA      | 账户 ID，使用 GET /v1/account/accounts 接口获得。现货交易使用‘spot’账户的 account-id；杠杆交易，请使用 ‘margin’ 账户的 account-id
symbol     | string    | ture    | NA      | 交易对, 例如btcusdt, ethbtc
side       | string    | false    | both    | 指定只返回某一个方向的订单，可能的值有: buy, sell. 默认两个方向都返回。
size       | int       | false    | 10      | 返回订单的数量，最大值2000。

<aside class="warning">“account-id” 和 “symbol” 需同时指定或者二者都不指定。如果二者都不指定，返回最多500条尚未成交订单，按订单号降序排列。</aside>

> Response:

```json
{  
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

### 响应数据

字段名称          | 数据类型 | 描述
---------           | --------- | -----------
id                  | integer   | 订单id
symbol              | string    | 交易对, 例如btcusdt, ethbtc 
price               | string    | limit order的交易价格
created-at          | int       | 订单创建的调整为北京时间的时间戳，单位毫秒
type                | string    | 订单类型,buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc 
filled-amount       | string    | 订单中已成交部分的数量,对于非“部分成交”订单，此字段为 0 
filled-cash-amount  | string    | 订单中已成交部分的总价格,对于非“部分成交”订单，此字段为 0 
filled-fees         | string    | 已成交交易手续费总额，对于非“部分成交”订单，此字段为 0 
source              | string    | 现货交易填写“api”（sys, web, api, app） 
state               | string    | 订单状态，包括submitted（已提交）, partial-filled（部分成交）, cancelling（正在取消） 

## 批量撤销订单（open orders）

API Key 权限：交易

此接口发送批量撤销订单的请求。

<aside class="warning">此接口只提交取消请求，实际取消结果需要通过订单状态，撮合状态等接口来确认。</aside>

### HTTP 请求

- POST ` /v1/order/orders/batchCancelOpenOrders`


### 请求参数

| 参数名称     | 是否必须 | 类型     | 描述           | 默认值  | 取值范围 |
| -------- | ---- | ------ | ------------ | ---- | ---- |
| account-id | true  | string | 账户ID     |     |      |
| symbol     | false | string | 交易对     |      |   单个交易对字符串，缺省将返回所有符合条件尚未成交订单  |
| side | false | string | 主动交易方向 |      |   “buy”或“sell”，缺省将返回所有符合条件尚未成交订单   |
| size | false | int | 所需返回记录数  |  100 |   [0,100]   |


> Response:

```json
{
  "status": "ok",
  "data": {
    "success-count": 2,
    "failed-count": 0,
    "next-id": 5454600
  }
}
```


### 响应数据


| 参数名称 | 是否必须 | 数据类型   | 描述    | 取值范围 |
| ---- | ---- | ------ | ----- | ---- |
| success-count | true | int | 成功取消的订单数 |     |
| failed-count | true | int | 取消失败的订单数 |     |
| next-id | true | long | 下一个符合取消条件的订单号 |    |

## 批量撤销订单

API Key 权限：交易

此接口同时为多个订单（基于id）发送取消请求。

### HTTP 请求

- POST ` /v1/order/orders/batchcancel`

```json
{
  "order-ids": [
    "1", "2", "3"
  ]
}
```

### 请求参数

| 参数名称  | 是否必须 | 类型   | 描述   | 默认值  | 取值范围 |
| ---- | ---- | ---- | ----  | ---- | ---- |
| order-ids | true | list | 撤销订单ID列表 |  |单次不超过50个订单id|


> Response:

```json
{  
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

### 响应数据

| 字段名称 | 数据类型 | 描述 |
| ---- | ----- | ---- |
| data | map | 撤单结果 |

## 查询订单详情

API Key 权限：读取

此接口返回指定订单的最新状态和详情。

### HTTP 请求

- GET `/v1/order/orders/{order-id}`


### 请求参数

| 参数名称     | 是否必须 | 类型  | 描述   | 默认值  | 取值范围 |
| -------- | ---- | ------ | -----  | ---- | ---- |
| order-id | true | string | 订单ID，填在path中 |      |      |


> Response:

```json
{  
  "data": 
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
    "exchange": "huobi",
    "batch": ""
  }
}
```

### 响应数据

| 字段名称     | 是否必须  | 数据类型   | 描述   | 取值范围     |
| ----------------- | ----- | ------ | -------  | ----  |
| account-id        | true  | long   | 账户 ID    |       |
| amount            | true  | string | 订单数量              |    |
| canceled-at       | false | long   | 订单撤销时间    |     |
| created-at        | true  | long   | 订单创建时间    |   |
| field-amount      | true  | string | 已成交数量    |     |
| field-cash-amount | true  | string | 已成交总金额     |      |
| field-fees        | true  | string | 已成交手续费（买入为币，卖出为钱） |     |
| finished-at       | false | long   | 订单变为终结态的时间，不是成交时间，包含“已撤单”状态    |     |
| id                | true  | long   | 订单ID    |     |
| price             | true  | string | 订单价格       |     |
| source            | true  | string | 订单来源   | api |
| state             | true  | string | 订单状态   | submitting , submitted 已提交, partial-filled 部分成交, partial-canceled 部分成交撤销, filled 完全成交, canceled 已撤销 |
| symbol            | true  | string | 交易对   | btcusdt, ethbtc, rcneth ... |
| type              | true  | string | 订单类型   | buy-market：市价买, sell-market：市价卖, buy-limit：限价买, sell-limit：限价卖, buy-ioc：IOC买单, sell-ioc：IOC卖单 |

## 成交明细

API Key 权限：读取

此接口返回指定订单的成交明细。

### HTTP 请求

- GET `/v1/order/orders/{order-id}/matchresults`



### 请求参数

| 参数名称  | 是否必须 | 类型  | 描述  | 默认值  | 取值范围 |
| -------- | ---- | ------ | -----  | ---- | ---- |
| order-id | true | string | 订单ID，填在path中 |      |      |


> Response:

```javascript
{  
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
    ...
  ]
}
```

### 响应数据

<aside class="notice">返回的主数据对象为一个对象数组，其中每一个元件代表一个交易结果。</aside>

| 字段名称    | 是否必须 | 数据类型   | 描述   | 取值范围     |
| ------------- | ---- | ------ | -------- | -------- |
| created-at    | true | long   | 成交时间     |    |
| filled-amount | true | string | 成交数量     |    |
| filled-fees   | true | string | 成交手续费    |     |
| id            | true | long   | 订单成交记录ID |     |
| match-id      | true | long   | 撮合ID     |     |
| order-id      | true | long   | 订单 ID    |      |
| price         | true | string | 成交价格  |    |
| source        | true | string | 订单来源  | api      |
| symbol        | true | string | 交易对   | btcusdt, ethbtc, rcneth ...  |
| type          | true | string | 订单类型   | buy-market：市价买, sell-market：市价卖, buy-limit：限价买, sell-limit：限价卖, buy-ioc：IOC买单, sell-ioc：IOC卖单 |

## 搜索历史订单

API Key 权限：读取

此接口基于搜索条件查询历史订单。

### HTTP 请求

- GET `/v1/order/orders`

```json
{
   "account-id": "100009",
   "amount": "10.1",
   "price": "100.1",
   "source": "api",
   "symbol": "ethusdt",
   "type": "buy-limit"
}
```


### 请求参数

| 参数名称   | 是否必须  | 类型     | 描述   | 默认值  | 取值范围   |
| ---------- | ----- | ------ | ------  | ---- | ----  |
| symbol     | true  | string | 交易对      |      |btcusdt, ethbtc, rcneth ...  |
| types      | false | string | 查询的订单类型组合，使用','分割  |      | buy-market：市价买, sell-market：市价卖, buy-limit：限价买, sell-limit：限价卖, buy-ioc：IOC买单, sell-ioc：IOC卖单 |
| start-date | false | string | 查询开始日期, 日期格式yyyy-mm-dd。 以订单生成时间进行查询 | -180 days     | [-180 days, end-date] （自6月10日起， start-date与end-date的查询窗口最大为2天，如果超出范围，接口会返回错误码。 |
| end-date   | false | string | 查询结束日期, 日期格式yyyy-mm-dd。 以订单生成时间进行查询 | today     | [start-date, today] （自6月10日起， start-date与end-date的查询窗口最大为2天，如果超出范围，接口会返回错误码。   |
| states     | true  | string | 查询的订单状态组合，使用','分割  |      | submitted 已提交, partial-filled 部分成交, partial-canceled 部分成交撤销, filled 完全成交, canceled 已撤销 |
| from       | false | string | 查询起始 ID   |      |    |
| direct     | false | string | 查询方向   |      | prev 向前，时间（或 ID）正序；next 向后，时间（或 ID）倒序）    |
| size       | false | string | 查询记录大小      | 100     |  [1, 1000]       |


> Response:

```javascript
{  
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
      "exchange": "huobi",
      "batch": ""
    }
    ...
  ]
}
```

### 响应数据

| 参数名称    | 是否必须  | 数据类型   | 描述   | 取值范围   |
| ----------------- | ----- | ------ | ----------------- | ----  |
| account-id        | true  | long   | 账户 ID    |     |
| amount            | true  | string | 订单数量    |   |
| canceled-at       | false | long   | 接到撤单申请的时间   |    |
| created-at        | true  | long   | 订单创建时间   |    |
| field-amount      | true  | string | 已成交数量   |    |
| field-cash-amount | true  | string | 已成交总金额    |    |
| field-fees        | true  | string | 已成交手续费（买入为基础币，卖出为计价币） |       |
| finished-at       | false | long   | 最后成交时间    |   |
| id                | true  | long   | 订单ID    |    |
| price             | true  | string | 订单价格  |    |
| source            | true  | string | 订单来源   | api  |
| state             | true  | string | 订单状态    | submitting , submitted 已提交, partial-filled 部分成交, partial-canceled 部分成交撤销, filled 完全成交, canceled 已撤销 |
| symbol            | true  | string | 交易对    | btcusdt, ethbtc, rcneth ... |
| type              | true  | string | 订单类型  | submit-cancel：已提交撤单申请  ,buy-market：市价买, sell-market：市价卖, buy-limit：限价买, sell-limit：限价卖, buy-ioc：IOC买单, sell-ioc：IOC卖单 |

### start-date, end-date相关错误码 （自6月10日生效）

|错误码|对应错误场景|
|------------|----------------------------------------------|
|invalid_interval| start date小于end date; 或者 start date 与end date之间的时间间隔大于2天|
|invalid_start_date|start date是一个180天之前的日期；或者start date是一个未来的日期|
|invalid_end_date|end date 是一个180天之前的日期；或者end date是一个未来的日期|


## 搜索最近48小时内历史订单

API Key 权限：读取

此接口基于搜索条件查询最近48小时内历史订单。

### HTTP 请求

- GET `/v1/order/history`

```json
{
   "symbol": "btcusdt",
   "start-time": "1556417645419",
   "end-time": "1556533539282",
   "direct": "prev",
   "size": "10"
}
```


### 请求参数

| 参数名称   | 是否必须  | 类型     | 描述   | 默认值  | 取值范围   |
| ---------- | ----- | ------ | ------  | ---- | ----  |
| symbol     | false  | string | 交易对      |all      |btcusdt, ethbtc, rcneth ...  |
| start-time      | false | long | 查询起始时间（含）  |48小时前的时刻      |UTC time in millisecond |
| end-time | false | long | 查询结束时间（含） | 查询时刻     |UTC time in millisecond |
| direct   | false | string | 订单查询方向（注：仅在检索出的总条目数量超出size字段限定时起作用；如果检索出的总条目数量在size 字段限定内，direct 字段不起作用。） | next     |prev, next   |
| size     | false  | int | 每次返回条目数量  |100      | [10,1000] |



> Response:

```json
{
    "status": "ok",
    "data": [
        {
            "id": 31215214553,
            "symbol": "btcusdt",
            "account-id": 4717043,
            "amount": "1.000000000000000000",
            "price": "1.000000000000000000",
            "created-at": 1556533539282,
            "type": "buy-limit",
            "field-amount": "0.0",
            "field-cash-amount": "0.0",
            "field-fees": "0.0",
            "finished-at": 1556533568953,
            "source": "web",
            "state": "canceled",
            "canceled-at": 1556533568911
        }
    ]
}
```

### 响应数据

| 参数名称    | 是否必须  | 数据类型   | 描述   | 取值范围   |
| ----------------- | ----- | ------ | ----------------- | ----  |
| {account-id        | true  | long   | 账户 ID    |     |
| amount            | true  | string | 订单数量    |   |
| canceled-at       | false | long   | 接到撤单申请的时间   |    |
| created-at        | true  | long   | 订单创建时间   |    |
| field-amount      | true  | string | 已成交数量   |    |
| field-cash-amount | true  | string | 已成交总金额    |    |
| field-fees        | true  | string | 已成交手续费（买入为基础币，卖出为计价币） |       |
| finished-at       | false | long   | 最后成交时间    |   |
| id                | true  | long   | 订单ID    |    |
| price             | true  | string | 订单价格  |    |
| source            | true  | string | 订单来源   | api  |
| state             | true  | string | 订单状态    | partial-canceled 部分成交撤销, filled 完全成交, canceled 已撤销 |
| symbol            | true  | string | 交易对    | btcusdt, ethbtc, rcneth ... |
| type}              | true  | string | 订单类型  | buy-market：市价买, sell-market：市价卖, buy-limit：限价买, sell-limit：限价卖, buy-ioc：IOC买单, sell-ioc：IOC卖单, buy-limit-maker, sell-limit-maker |
| next-time            | false  | long |下一查询起始时间（当请求字段”direct”为”prev”时有效）, 下一查询结束时间（当请求字段”direct”为”next”时有效）。注：仅在检索出的总条目数量超出size字段限定时，此返回字段存在。 |UTC time in millisecond   |

## 当前和历史成交

API Key 权限：读取

此接口基于搜索条件查询当前和历史成交记录。

### HTTP 请求

- GET `/v1/order/matchresults`


### 请求参数

| 参数名称   | 是否必须  | 类型  | 描述   | 默认值  | 取值范围    |
| ---------- | ----- | ------ | ------ | ---- | ----------- |
| symbol     | true  | string | 交易对   | NA |  btcusdt, ethbtc, rcneth ...  |
| types      | false | string | 查询的订单类型组合，使用','分割   |      | buy-market：市价买, sell-market：市价卖, buy-limit：限价买, sell-limit：限价卖, buy-ioc：IOC买单, sell-ioc：IOC卖单 |
| start-date | false | string | 查询开始日期, 日期格式yyyy-mm-dd | -61 days     | [-61day, today] （自6月10日起， start-date与end-date的查询窗口最大为2天，如果超出范围，接口会返回错误码。 |
| end-date   | false | string | 查询结束日期, 日期格式yyyy-mm-dd |   today   |  [start-date, today] （自6月10日起， start-date与end-date的查询窗口最大为2天，如果超出范围，接口会返回错误码。 |
| from       | false | string | 查询起始 ID    |   订单成交记录ID（最大值）   |     |
| direct     | false | string | 查询方向    |   默认 next， 成交记录 ID 由大到小排序   | prev 向前，时间（或 ID）正序；next 向后，时间（或 ID）倒序）   |
| size       | false | string | 查询记录大小    |   100   | [1，100]  |


> Response:

```javascript
{  
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
    ...
  ]
}
```

### 响应数据

<aside class="notice">返回的主数据对象为一个对象数组，其中每一个元件代表一个交易结果。</aside>

| 参数名称   | 是否必须 | 数据类型   | 描述   | 取值范围   |
| ------------- | ---- | ------ | -------- | ------- |
| created-at    | true | long   | 成交时间     |    |
| filled-amount | true | string | 成交数量     |    |
| filled-fees   | true | string | 成交手续费    |    |
| id            | true | long   | 订单成交记录 ID |    |
| match-id      | true | long   | 撮合 ID     |    |
| order-id      | true | long   | 订单 ID    |    |
| price         | true | string | 成交价格     |    |
| source        | true | string | 订单来源     | api   |
| symbol        | true | string | 交易对      | btcusdt, ethbtc, rcneth ...  |
| type          | true | string | 订单类型     | buy-market：市价买, sell-market：市价卖, buy-limit：限价买, sell-limit：限价卖, buy-ioc：IOC买单, sell-ioc：IOC卖单 |

### start-date, end-date相关错误码 （自6月10日生效）

|错误码|对应错误场景|
|------------|----------------------------------------------|
|invalid_interval| start date小于end date; 或者 start date 与end date之间的时间间隔大于2天|
|invalid_start_date|start date是一个61天之前的日期；或者start date是一个未来的日期|
|invalid_end_date|end date 是一个61天之前的日期；或者end date是一个未来的日期|


# 借贷

<aside class="notice">重要：如果使用借贷资产交易，请在下单接口/v1/order/orders/place请求参数source中填写‘margin-api’ 。访问借贷相关的接口需要进行签名认证。</aside>

<aside class="notice">目前杠杆交易仅支持部分以 USDT 和 BTC 为报价币种的交易对。</aside>

## 资产划转

API Key 权限：交易

此接口用于现货账户与杠杆账户的资产互转。

从现货账户划转至杠杆账户 `transfer-in`，从杠杆账户划转至现货账户 `transfer-out`

### HTTP 请求

- POST ` /v1/dw/transfer-in/margin`

- POST ` /v1/dw/transfer-out/margin`

```json
{
  "symbol": "ethusdt",
  "currency": "eth",
  "amount": "1.0"
}
```


### 请求参数

参数名称 | 数据类型 | 是否必需 | 默认值 | 描述
---------  | --------- | -------- | ------- | -----------
symbol     | string    | true     | NA      | 交易对, e.g. btcusdt, ethbtc 
currency   | string    | true     | NA      | 币种
amount     | string    | true     | NA      | 划转数量


> Response:

```json
{  
  "data": 1000
}
```

### 响应数据


参数名称 | 数据类型 | 描述
------ | ------- | -----
data   | integer | Transfer id


## 申请借贷

API Key 权限：交易

此接口用于申请借贷.

### HTTP 请求

- POST ` /v1/margin/orders`

```json
{
  "symbol": "ethusdt",
  "currency": "eth",
  "amount": "1.0"
}
```


### 请求参数

参数名称 | 数据类型 | 是否必需 | 默认值 | 描述
---------  | --------- | -------- | ------- | -----------
symbol     | string    | true     | NA      | 交易对, e.g. btcusdt, ethbtc 
currency   | string    | true     | NA      | 币种
amount     | string    | true     | NA      | 借贷数量

> Response:

```json
{  
  "data": 1000
}
```


### 响应数据

字段名称| 数据类型 | 描述
-------| ------  | ----
data   | integer | Margin order id



## 归还借贷

API Key 权限：交易

此接口用于归还借贷.

### HTTP 请求

- POST ` /v1/margin/orders/{order-id}/repay`

```json
{
  "amount": "1.0"
}
```


### 请求参数

参数名称 | 数据类型 | 是否必需 | 描述
---------  | --------- | -------- | -----------
order-id   | string    | true     | 借贷订单 ID，写在 url path 中
amount     | string    | true     | 归还币种数量


> Response:

```json
{  
  "data": 1000
}
```

### 响应数据


参数名称     | 数据类型 | 描述
-------  | ------- | -----------
data     | integer | Margin order id


## 查询借贷订单

API Key 权限：读取

此接口基于指定搜索条件返回借贷订单。

### HTTP 请求

- GET ` /v1/margin/loan-orders`

### 请求参数

| 参数名称       | 是否必须  | 类型     | 描述    | 默认值  | 取值范围   |
| ----- | ----- | ------ |  -------  | ---- |  ----  |
| symbol | true | string | 交易对  |  |  |
| start-date | false | string | 查询开始日期, 日期格式yyyy-mm-dd  |     |    |
| end-date | false | string | 查询结束日期, 日期格式yyyy-mm-dd  |    |    |
| states | false | string | 状态 |     |   |
| from   | false | string | 查询起始 ID  |    |     |
| direct | false | string | 查询方向     |    | prev 向前，时间（或 ID）正序；next 向后，时间（或 ID）倒序） |
| size   | false | string | 查询记录大小  |    |     |

> Response:

```json
{  
  "data": [
    {
      "loan-balance": "0.100000000000000000",
      "interest-balance": "0.000200000000000000",
      "interest-rate": "0.002000000000000000",
      "loan-amount": "0.100000000000000000",
      "accrued-at": 1511169724531,
      "interest-amount": "0.000200000000000000",
      "symbol": "ethbtc",
      "currency": "btc",
      "id": 394,
      "state": "accrual",
      "account-id": 17747,
      "user-id": 119913,
      "created-at": 1511169724531
    }
  ]
}
```

### 响应数据


| 字段名称 | 是否必须 | 数据类型 | 描述 | 取值范围 |
|-----|-----|-----|-----|------|
|   id  |  true  |  long  |  订单号 | |
|   user-id  |  true  |  long  | 用户ID | |
|   account-id  |  true  |  long  |  账户ID | |
|   symbol  |  true  |  string  |  交易对 | |
|   currency  |  true  |  string  |  币种 | |
| loan-amount | true |string | 借贷本金总额 | |
| loan-balance | true | string | 未还本金 | |
| interest-rate | true | string | 利率 | |
| interest-amount | true | string | 利息总额 | |
| interest-balance | true | string | 未还利息 | |
| created-at | true | long | 借贷发起时间 | |
| accrued-at | true | long | 最近一次计息时间 | |
| state | true | string | 订单状态 |created 未放款，accrual 已放款，cleared 已还清，invalid 异常|


## 借贷账户详情

API Key 权限：读取

此接口返回借贷账户详情。

### HTTP 请求

- GET `/v1/margin/accounts/balance`

```json
{
   "account-id": "100009",
   "amount": "10.1",
   "price": "100.1",
   "source": "api",
   "symbol": "ethusdt",
   "type": "buy-limit"
}
```

### 请求参数

| 参数名称 | 是否必须 | 类型 | 描述 | 默认值 | 取值范围 |
|---------|---------|-----|-----|-------|--------|
| symbol | false | string | 交易对，作为get参数  |  |  |

> Response:

```javascript
{  
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
              "currency": "btc",
              "type": "transfer-out-available",//可转btc
              "balance": "1163.872174670000000000"
          },
          {
              "currency": "btc",
              "type": "loan-available",//可借btc
              "balance": "8161.876538350676000000"
          }
      ]
    }
  ]
}
```

### 响应数据

| 字段名称 | 是否必须 | 数据类型 | 描述 | 取值范围 |
|-----|-----|-----|-----|------|
| symbol  |  true  |  string  |  交易对 | |
| state  |  true  |  string  |  账户状态 | working,fl-sys,fl-mgt,fl-end |
| risk-rate | true | object | 风险率 | |
| fl-price | true | string | 爆仓价 | |
| list | true | array | 借贷账户详情列表 | |




<br>
<br>
<br>
