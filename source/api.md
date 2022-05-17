# 做市商接口文档

**版本**
`1.0.0`

[toc]

* * *

## 更新日志

### 2022-03-02
- 更新服务器地址：https://api.hotx.pro
- 添加两种接口限频模式；单接口模式、多接口共用模式
- 每个接口都有各自的使用权重
- 签名的Timestamp更改为毫秒级单位

* * *

## API 基本信息

- 接口需要用户的 API Key，用户需要在web 端创建API key。
- 本篇列出接口的baseurl: **https://api.hotx.pro**
- 如果上面的baseURL访问有性能问题，请访问下面的API集群:
  - **https://api.hotx.pro**
- 所有接口的响应都是 JSON 格式。
- 响应中如有数组，数组元素以时间**升序**排列，越早的数据越提前。
- 所有时间、时间戳均为UNIX时间，单位为**秒**。


### HTTP STATUS CODE

|  status code   | 介绍|
|  ----   |----  |
| 200  | 一般代表请求处理成功 |
| 400  | 1、头部信息缺失；2、请求方式有误 |
| 401  | 当前请求接口拒绝访问 |
| 403  | 当前请求签名错误 |
| 404  | 请求地址不存在 |
| 503  | 业务逻辑请求失败，需要根据具体业务错误码进行处理 |

### 接口错误代码

- 使用接口时, 每个接口都有可能抛出异常;

> 错误代码返回形式如下:

```
{
  "code": "错误码",
  "message": "错误信息"
}
```

- 具体的错误码及其解释在 [错误代码](#错误代码列表).

### 接口的基本信息

- `POST`, `PUT` 方法的接口,参数必须在 `request body` 中发送，对参数的顺序不做要求。
- `GET`, `DELETE`通过url传参，详见具体接口说明。

### 访问限制

#### MQTT 连接限制
- 每个用户只能连接一个MQTT 连接，其他连接将会被拒绝。

#### API 接口限频
- 接口访问限制，所有接口共用每分钟/1200次；
- 当收到429异常时，您有责任停止发送请求，不得滥用API；
- 收到429后仍然继续违反访问限制，会被封禁IP，并收到418错误码；
- 收到418异常后若还是频繁违反限制，封禁时间会逐渐延长，从最短2分钟到最长30分钟；

### 数据来源

- 因为API系统是异步的, 所以返回的数据有延时很正常, 也在预期之中。
- 在每个接口中，列出了其数据的来源，可以用于理解数据的时效性。

系统一共有3个数据来源，按照更新速度的先后排序。排在前面的数据最新，在后面就有可能存在延迟。

- **撮合引擎** - 表示数据来源于撮合引擎
- **缓存** - 表示数据来源于内部或者外部的缓存
- **数据库** - 表示数据直接来源于数据库

 有些接口有不止一个数据源, 比如 `缓存 => 数据库`, 这表示接口会先从第一个数据源检查，如果没有数据，则检查下一个数据源。


### SIGNED  Endpoint security

- 调用接口时，除了接口本身所需的参数外，还需要在`HEADERS`中传递 `Signature`, 即签名参数。
- 签名使用`HMAC SHA256`算法. API-KEY所对应的API-Secret作为 `HMAC SHA256` 的密钥，其他所有参数作为`HMAC SHA256`的操作对象，得到的输出即为签名。
- `签名` **大小写不敏感**.

### 时间同步安全

- 签名接口均需要在headers中传递 `Timestamp`参数，其值应当是请求发送时刻的unix时间戳(毫秒)。
- 服务器收到请求时会判断请求中的时间戳，如果是5000毫秒之前发出的，则请求会被认为无效。这个时间空窗值可以通过在headers发送可选参数 `RecvWindow`来定义。

> 逻辑伪代码如下:

```
  if (timestamp < (serverTime + 5000) && (serverTime - timestamp) <= RecvWindow)
  {
    // process request
  } 
  else 
  {
    // reject request
  }
```
**关于交易时效性** 互联网状况并不完全稳定可靠,因此你的程序本地到服务器的时延会有抖动。这是我们设置`RecvWindow`的目的所在，如果你从事高频交易，对交易时效性有较高的要求，可以灵活设置`RecvWindow`以达到你的要求。

 推荐使用5000毫秒以下的 RecvWindow! 最多不能超过 60000毫秒!

### POST /v1/exchange/spot/order 的示例

以下是调用接口下单的示例 apikey、secret仅供示范

| Key       | Value                                                        |
| :-------- | :----------------------------------------------------------- |
| apiKey    | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh12 |
| secretKey | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj1j |

HEADERS

| 参数        | 取值           |
| :---------- | :------------ |
| RecvWindow  | 5000          |
| Timestamp   | 1499827319012 |

BODY
```json
{
	"entrustAmount": "1",
	"entrustPrice": "0.1",
	"entrustType": "B",
	"symbol": "bnb_usdt",
	"entrustStrategy": "GTC",
	"entrustMode": "10"
}
```
#### 示例

> **Example **
>
> **HMAC SHA256 signature:**

```
    text:/v1/exchange/spot/order{"entrustAmount":"1","entrustPrice":"0.1","entrustType":"B","symbol":"bnb_usdt","entrustStrategy":"GTC","entrustMode":"10"}1499827319012
	key: NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj1j
    Signature= aebf65dfa1ee661ed85067a9ab09589a1a8230a8e5659fd42e5b444e1c413161
```

> **curl command:**

```
    (HMAC SHA256)
    $ curl -H "Access-Key: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh12" -H "Signature:aebf65dfa1ee661ed85067a9ab09589a1a8230a8e5659fd42e5b444e1c413161" -H "RecvWindow: 5000" -H "Timestamp: 1499827319012" -X POST 'https://api.hotx.pro/v1/exchange/spot/order' -d '{"entrustAmount":"1","entrustPrice":"0.1","entrustType":"B","symbol":"bnb_usdt","entrustStrategy":"GTC","entrustMode":"10"}'
```

- **requestBody:**

```json
{
	"entrustAmount": "1",
	"entrustPrice": "0.1",
	"entrustType": "B",
	"symbol": "bnb_usdt",
	"entrustStrategy": "GTC",
	"entrustMode": "10"
}
```

### 公开 API 参数


#### 枚举定义


**订单状态 (状态 status):**

| 状态               | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| `00`              | 订单等待交易引擎接受                                           |
| `10`              | 订单被交易引擎接受                                           |
| `11` | 部分订单被成交                                               |
| `01`           | 订单完全成交                                                 |
| `21`         | 订单已撤销（包括被用户、交易引擎取消, 比如 LIMIT FOK 订单没有成交 市价单没有完全成交 强平期间被取消的订单 交易所维护期间被取消的订单）                                               |
| `20`   | 撤销中                                         |



**订单类型 (entrustMode):**

- 10 限价单
- 20 市价单
- 11 止盈止损限价

**订单方向 (方向 entrustType):**

- B 买入
- S 卖出

**下单策略 (entrustStrategy):**

这里定义了订单多久能够失效

| Status | Description                                           |
| :----- | :---------------------------------------------------- |
| `GTC`  | 成交为止 订单会一直有效，直到被成交或者取消。         |
| `IOC`  | 无法立即成交的部分就撤销 订单在失效前会尽量多的成交。 |
| `FOK`  | 无法全部立即成交就撤销 如果无法全部成交，订单会取消。 |
| `MAKER`  | 只做Maker(Post Only)订单不会立刻在市场成交，保证用户始终为Maker,如果委托会立即与已有委托成交，那么该委托会被取消。 |

**K线间隔:**

minute -> 分钟; hour -> 小时; day -> 天; week -> 周; month -> 月

- minute1
- minute3
- minute5
- minute15
- minute30
- hour1
- hour2
- hour6
- hour12
- day1
- day3
- week1
- month1


## 现货行情

### 获取服务器时间

> **响应**

```
{
  "serverTime": 1499827319
}
GET /v1/time
```

测试能否联通 Rest API 并 获取服务器时间。

**权重(IP):1**

### 现货规范

> **响应**

```json
[{
	"areaCode": "usdt",
	"areaName": "USDT交易区",
	"areaSymbol": "$",
	"tradePairMarkets": [{
		"areaCode": "usdt",
		"closeTime": "23:59:59",
		"enable": true,
		"exchangeRunningNode": "233e2c62-1b56-45f6-aff0-2bee8f94933c",
		"extendedJson": "",
		"hotSort": 0,
		"indexSort": 1,
		"lineChartUrl": "http:\/\/192.168.1.222:9083\/exchange\/system\/spot\/kline\/minute15\/btc_usdt",
		"matchRunningNode": "1de46260-1a69-49d0-84bb-ff4ad4beb84d",
		"maxCount": "200",
		"maxCountDecimalDigit": 4,
		"maxPrice": "999999",
		"maxPriceDecimalDigit": 4,
		"minCount": "0.0001",
		"minPrice": "1000",
		"offlineTime": 1566554343,
		"onlineTime": 1566557938,
		"openTime": "00:00",
		"priceRange": "1",
		"sort": 1,
		"supportDepth": "1,2,3,4,5,6",
		"symbol": "btc_usdt",
		"tradeBoard": "00",
		"tradeMarket": "spot"
	}, {
		"areaCode": "usdt",
		"closeTime": "23:59:59",
		"enable": true,
		"exchangeRunningNode": "233e2c62-1b56-45f6-aff0-2bee8f94933c",
		"extendedJson": "",
		"hotSort": 0,
		"indexSort": 2,
		"lineChartUrl": "http:\/\/192.168.1.222:9083\/exchange\/system\/spot\/kline\/minute15\/eth_usdt",
		"matchRunningNode": "1de46260-1a69-49d0-84bb-ff4ad4beb84d",
		"maxCount": "100",
		"maxCountDecimalDigit": 4,
		"maxPrice": "99999",
		"maxPriceDecimalDigit": 4,
		"minCount": "0.01",
		"minPrice": "100",
		"offlineTime": 1566471740,
		"onlineTime": 1566558140,
		"openTime": "00:00",
		"priceRange": "1",
		"sort": 2,
		"supportDepth": "1,2,3,4,5,6",
		"symbol": "eth_usdt",
		"tradeBoard": "00",
		"tradeMarket": "spot"
	}, {
		"areaCode": "usdt",
		"enable": true,
		"exchangeRunningNode": "233e2c62-1b56-45f6-aff0-2bee8f94933c",
		"extendedJson": "",
		"hotSort": 0,
		"indexSort": 0,
		"lineChartUrl": "http:\/\/192.168.1.222:9083\/exchange\/system\/spot\/kline\/minute15\/ada_usdt",
		"matchRunningNode": "1de46260-1a69-49d0-84bb-ff4ad4beb84d",
		"maxCount": "500000",
		"maxCountDecimalDigit": 8,
		"maxPrice": "500000",
		"maxPriceDecimalDigit": 8,
		"minCount": "0.0001",
		"minPrice": "0.0001",
		"onlineTime": 1637135705,
		"openTime": "00:00",
		"priceRange": "1",
		"sort": 8,
		"supportDepth": "1",
		"symbol": "ada_usdt",
		"tradeMarket": "spot"
	}, {
		"areaCode": "usdt",
		"enable": true,
		"exchangeRunningNode": "233e2c62-1b56-45f6-aff0-2bee8f94933c",
		"extendedJson": "",
		"hotSort": 0,
		"indexSort": 0,
		"lineChartUrl": "http:\/\/192.168.1.222:9083\/exchange\/system\/spot\/kline\/minute15\/sol_usdt",
		"matchRunningNode": "1de46260-1a69-49d0-84bb-ff4ad4beb84d",
		"maxCount": "500000",
		"maxCountDecimalDigit": 8,
		"maxPrice": "500000",
		"maxPriceDecimalDigit": 8,
		"minCount": "0.0001",
		"minPrice": "0.0001",
		"onlineTime": 1637135776,
		"openTime": "00:00",
		"priceRange": "1",
		"sort": 9,
		"supportDepth": "1",
		"symbol": "sol_usdt",
		"tradeBoard": "00",
		"tradeMarket": "spot"
	}, {
		"areaCode": "usdt",
		"enable": true,
		"exchangeRunningNode": "233e2c62-1b56-45f6-aff0-2bee8f94933c",
		"extendedJson": "",
		"hotSort": 0,
		"indexSort": 0,
		"lineChartUrl": "http:\/\/192.168.1.222:9083\/exchange\/system\/spot\/kline\/minute15\/dot_usdt",
		"matchRunningNode": "1de46260-1a69-49d0-84bb-ff4ad4beb84d",
		"maxCount": "500000",
		"maxCountDecimalDigit": 8,
		"maxPrice": "500000",
		"maxPriceDecimalDigit": 8,
		"minCount": "0.0001",
		"minPrice": "0.0001",
		"onlineTime": 1637135879,
		"openTime": "00:00",
		"priceRange": "1",
		"sort": 10,
		"supportDepth": "1",
		"symbol": "dot_usdt",
		"tradeMarket": "spot"
	}, {
		"areaCode": "usdt",
		"enable": true,
		"exchangeRunningNode": "233e2c62-1b56-45f6-aff0-2bee8f94933c",
		"extendedJson": "",
		"hotSort": 0,
		"indexSort": 0,
		"lineChartUrl": "http:\/\/192.168.1.222:9083\/exchange\/system\/spot\/kline\/minute15\/doge_usdt",
		"matchRunningNode": "1de46260-1a69-49d0-84bb-ff4ad4beb84d",
		"maxCount": "500000",
		"maxCountDecimalDigit": 8,
		"maxPrice": "5000001",
		"maxPriceDecimalDigit": 8,
		"minCount": "0.0001",
		"minPrice": "0.0001",
		"onlineTime": 1637135919,
		"openTime": "00:00",
		"priceRange": "1",
		"sort": 11,
		"supportDepth": "1",
		"symbol": "doge_usdt",
		"tradeBoard": "00",
		"tradeMarket": "spot"
	}, {
		"areaCode": "usdt",
		"enable": true,
		"exchangeRunningNode": "233e2c62-1b56-45f6-aff0-2bee8f94933c",
		"extendedJson": "",
		"hotSort": 0,
		"indexSort": 0,
		"lineChartUrl": "http:\/\/192.168.1.222:9083\/exchange\/system\/spot\/kline\/minute15\/shib_usdt",
		"matchRunningNode": "1de46260-1a69-49d0-84bb-ff4ad4beb84d",
		"maxCount": "500000",
		"maxCountDecimalDigit": 8,
		"maxPrice": "500000",
		"maxPriceDecimalDigit": 8,
		"minCount": "0.0001",
		"minPrice": "0.0001",
		"onlineTime": 1637139916,
		"openTime": "00:00",
		"priceRange": "1",
		"sort": 12,
		"supportDepth": "1",
		"symbol": "shib_usdt",
		"tradeBoard": "00",
		"tradeMarket": "spot"
	}, {
		"areaCode": "usdt",
		"enable": true,
		"exchangeRunningNode": "233e2c62-1b56-45f6-aff0-2bee8f94933c",
		"extendedJson": "",
		"hotSort": 0,
		"indexSort": 0,
		"lineChartUrl": "http:\/\/192.168.1.222:9083\/exchange\/system\/spot\/kline\/minute15\/avax_usdt",
		"matchRunningNode": "1de46260-1a69-49d0-84bb-ff4ad4beb84d",
		"maxCount": "500000",
		"maxCountDecimalDigit": 8,
		"maxPrice": "500000",
		"maxPriceDecimalDigit": 8,
		"minCount": "0.0001",
		"minPrice": "0.0001",
		"onlineTime": 1637140148,
		"openTime": "00:00",
		"priceRange": "1",
		"sort": 13,
		"supportDepth": "1",
		"symbol": "avax_usdt",
		"tradeBoard": "00",
		"tradeMarket": "spot"
	}, {
		"areaCode": "usdt",
		"enable": true,
		"exchangeRunningNode": "233e2c62-1b56-45f6-aff0-2bee8f94933c",
		"extendedJson": "",
		"hotSort": 0,
		"indexSort": 0,
		"lineChartUrl": "http:\/\/192.168.1.222:9083\/exchange\/system\/spot\/kline\/minute15\/luna_usdt",
		"matchRunningNode": "1de46260-1a69-49d0-84bb-ff4ad4beb84d",
		"maxCount": "5000",
		"maxCountDecimalDigit": 8,
		"maxPrice": "500000000",
		"maxPriceDecimalDigit": 8,
		"minCount": "0.0001",
		"minPrice": "0.0001",
		"onlineTime": 1637140239,
		"openTime": "00:00",
		"priceRange": "1",
		"sort": 14,
		"supportDepth": "1",
		"symbol": "luna_usdt",
		"tradeBoard": "00",
		"tradeMarket": "spot"
	}, {
		"areaCode": "usdt",
		"enable": true,
		"exchangeRunningNode": "233e2c62-1b56-45f6-aff0-2bee8f94933c",
		"extendedJson": "",
		"hotSort": 0,
		"indexSort": 0,
		"lineChartUrl": "http:\/\/192.168.1.222:9083\/exchange\/system\/spot\/kline\/minute15\/ltc_usdt",
		"matchRunningNode": "1de46260-1a69-49d0-84bb-ff4ad4beb84d",
		"maxCount": "1000000",
		"maxCountDecimalDigit": 8,
		"maxPrice": "5000000",
		"maxPriceDecimalDigit": 8,
		"minCount": "0.00000001",
		"minPrice": "0.00001",
		"onlineTime": 1637140492,
		"openTime": "00:00",
		"priceRange": "1",
		"sort": 15,
		"supportDepth": "1",
		"symbol": "ltc_usdt",
		"tradeBoard": "00",
		"tradeMarket": "spot"
	}, {
		"areaCode": "usdt",
		"enable": true,
		"exchangeRunningNode": "233e2c62-1b56-45f6-aff0-2bee8f94933c",
		"extendedJson": "",
		"hotSort": 0,
		"indexSort": 0,
		"lineChartUrl": "http:\/\/192.168.1.222:9083\/exchange\/system\/spot\/kline\/minute15\/uni_usdt",
		"matchRunningNode": "1de46260-1a69-49d0-84bb-ff4ad4beb84d",
		"maxCount": "10000000000",
		"maxCountDecimalDigit": 8,
		"maxPrice": "10000000",
		"maxPriceDecimalDigit": 8,
		"minCount": "0",
		"minPrice": "0.0000001",
		"onlineTime": 1637140546,
		"openTime": "00:00",
		"priceRange": "1",
		"sort": 16,
		"supportDepth": "1",
		"symbol": "uni_usdt",
		"tradeBoard": "00",
		"tradeMarket": "spot"
	}, {
		"areaCode": "usdt",
		"enable": true,
		"exchangeRunningNode": "233e2c62-1b56-45f6-aff0-2bee8f94933c",
		"extendedJson": "",
		"hotSort": 0,
		"indexSort": 0,
		"lineChartUrl": "http:\/\/192.168.1.222:9083\/exchange\/system\/spot\/kline\/minute15\/link_usdt",
		"matchRunningNode": "1de46260-1a69-49d0-84bb-ff4ad4beb84d",
		"maxCount": "10000000000",
		"maxCountDecimalDigit": 8,
		"maxPrice": "100000000",
		"maxPriceDecimalDigit": 8,
		"minCount": "0",
		"minPrice": "0",
		"onlineTime": 1637140599,
		"openTime": "00:00",
		"priceRange": "1",
		"sort": 17,
		"supportDepth": "1",
		"symbol": "link_usdt",
		"tradeBoard": "00",
		"tradeMarket": "spot"
	}]
}]
```
```
GET /v1/exchange/market/spot/areas
```


|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| areaCode  | string | 否 | 交易区编码 |
| areaName  | string | 是 | 交易区名称 |
| tradePairMarkets  | array | 是 | 交易区内交易对列表 |

tradePairMarkets

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| symbol  | string | 否 | 交易对编号 |
| maxCount  | string | 是 | 最大下单数量 |
| maxCountDecimalDigit  | int | 是 | 最大下单数量精度 |
| maxPrice  | string | 是 | 最大下单价格 |
| maxPriceDecimalDigit  | int | 是 | 最大下单价格精度 |
| minCount  | string | 是 | 最小下单数量 |
| minPrice  | string | 是 | 最小下单价格 |
| priceRange  | string | 是 | 价格波动范围 |
| supportDepth  | string | 是 | 支持深度 |


获取交易规则和交易对信息。

**数据源:** 缓存

**权重(IP):10**

### 深度信息  

> **响应**

```json
{
	"buy": [{
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67653.29",
		"total": "0"
	}],
	"sell": [{
		"amount": "9",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67666.53",
		"total": "0"
	}],
	"symbol": "btc_usdt",
	"timestamp": "1636429063333",
	"tradeMarket": "spot"
}
```

`GET /v1/exchange/spot/depth/{symbol}/1`

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| amount | string | 否 |数量 |
| entrustType | string | 否 |委托类型:B（买）、S（卖） |
| price | string | 否 |价格 |
| total | string | 否 |成交额 |


**参数:**

| 名称   | 类型   | 是否必需 | 描述                                                         |
| :----- | :----- | :------- | :----------------------------------------------------------- |
| symbol | STRING | YES      |

**数据源:** 缓存

**权重(IP):10**

### 近期成交列表

> **响应**

```json
[{
	"dealAmount": "2",
	"dealPrice": "67680.49",
	"dealSide": "B",
	"dealTimestamp": "1636427410",
	"symbol": "btc_usdt"
}, {
	"dealAmount": "3",
	"dealPrice": "67685",
	"dealSide": "B",
	"dealTimestamp": "1636427410",
	"symbol": "btc_usdt"
}]
```
```
GET /v1/exchange/spot/trades/{symbol}/{limit}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| dealAmount | string | 否 |成交量 |
| dealPrice | string | 否 |成交价 |
| dealSide | string | 否 |成交方向：B（买）、S（卖） |
| dealTimestamp | string | 否 |成交时间 |
| symbol | string | 否 |交易对 |


获取近期成交

**参数:**

| 名称   | 类型   | 是否必需 | 描述                   |
| :----- | :----- | :------- | :--------------------- |
| symbol | STRING | YES      |                        |
| limit  | INT    | YES       |  最大值 50. |

**数据源:** 缓存

**权重(IP):1**

### K线数据

> **响应**

```json
[
["1636428360", "67569.77", "67576.6", "67569.77", "67576.6", "25", "11.00", 10]
]
```
```
GET /v1/exchange/spot/kline/{symbol}/{klineTime}/{timestamp}/{count}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| arr[0] | string | 否 |时间戳 |
| arr[1] | string | 否 |开盘价 |
| arr[2] | string | 否 |最高成交价 |
| arr[3] | string | 否 |最低成交价 |
| arr[4] | string | 否 |收盘价 |
| arr[5] | string | 否 |成交量 |
| arr[6] | string | 否 |右币成交额 |
| arr[7] | string | 否 |交易成交笔数 |

每根K线代表一个交易对。
每根K线的开盘时间可视为唯一ID

**参数:**

| 名称      | 类型   | 是否必需 | 描述                  |
| :-------- | :----- | :------- | :-------------------- |
| symbol    | STRING | YES      |                       |
| klineTime  | STRING   | YES      | 详见枚举定义：K线间隔 |
| timestamp | LONG   | YES       |   截止时间                    |
| count     | INT    | YES       |  最大 300.  |


**数据源:** 缓存

**权重(IP):10**

## 现货钱包相关


### 获取现货资产 

获取针对用户的所有现货资产。

> **响应**

```json
{
	"accounts": [{
		"btc": "10011",
		"canRecharge": true,
		"canWithdraw": true,
		"cash": "10011",
		"cny": "4097187353.94",
		"contractHost": "",
		"currency": "btc",
		"freeze": "0",
		"locked": "0",
		"mid": 18,
		"productMode": "exchange",
		"submode": "00",
		"usdt": "631986621.42",
		"walletAddress": ""
	}, {
		"btc": "0",
		"canRecharge": true,
		"canWithdraw": true,
		"cash": "10000",
		"cny": "0",
		"contractHost": "eth",
		"currency": "bzz",
		"freeze": "0",
		"locked": "0",
		"mid": 18,
		"productMode": "exchange",
		"submode": "00",
		"usdt": "0",
		"walletAddress": ""
	}, {
		"btc": "0",
		"canRecharge": true,
		"canWithdraw": true,
		"cash": "10000",
		"cny": "0",
		"contractHost": "",
		"currency": "cny",
		"freeze": "0",
		"locked": "0",
		"mid": 18,
		"productMode": "exchange",
		"submode": "00",
		"usdt": "0",
		"walletAddress": ""
	}],
	"totalBTC": "10737.65206",
	"totalCNY": "4394343969.34",
	"totalUSDT": "677792991.292"
}
```

`GET /v1/ma/account/assets/sub/00 (HMAC SHA256)`

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| totalBTC  | string | 否 | BTC总资产 |
| totalCNY  | string | 否 | CNY总资产 |
| totalUSDT  | string | 否 | USDT总资产 |
| accounts  | array | 是 | 所有币种账户 |

accounts

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| btc  | string | 否 | BTC资产 |
| canRecharge  | string | 否 | 是否允许充值 |
| canWithdraw  | string | 否 | 是否允许提现 |
| cash  | string | 否 | 现金户 |
| currency  | string | 否 | 币种 |
| freeze  | string | 否 | 冻结户 |
| locked  | string | 否 | 锁仓户 |

**权重(IP):1**

### 上架资产详情 

> **响应**

```json
[{
	"alias": "以太坊",
	"blockBrowser": "https:\/\/www.yitaifang.com\/txs\/",
	"coinLogo": "www.baidu.com",
	"currency": "eth",
	"enable": true,
	"enableRecharge": true,
	"enableWithdraw": true,
	"minRecharge": "0.05",
	"minWithdraw": "0.05",
	"withdrawFee": "0.05",
	"supportChains": [{
		"currency": "eth",
		"name": "Etherum(ERC 20)"
	}]
}]
```
`GET /v1/ins/coin/config (HMAC SHA256)`

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| alias  | string | 否 | 别名|
| blockBrowser  | string | 否 | 区块链浏览器 |
| coinLogo  | string | 否 | 币logo |
| currency  | string | 否 | 币种 |
| enable  | string | 否 | 是否启用币种 |
| enableRecharge  | string | 否 | 开启冲币 |
| enableWithdraw  | string | 否 | 开启提币 |
| minRecharge  | string | 否 | 最小充值 |
| minWithdraw  | string | 否 | 最小提现 |
| supportChains  | array | 否 | 支持的链 |

supportChains

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| currency  | string | 否 | 支持的链 |
| name  | string | 否 | 支持的链的名称 |

**权重(IP):1**

## 现货交易

### 测试下单

> **响应**

```
{}
POST /v1/exchange/spot/order/test (HMAC SHA256)
```

用于测试订单请求，但不会提交到撮合引擎

**参数:**

同于 `POST /v1/exchange/spot/order`

### 下单

> **Response**
```json
{
		"averagePrice": "31217.8575",
		"createTime": 1635991697,
		"entrustSource":"01",
		"entrustMode": "10",
		"entrustPrice": "62445.64",
		"entrustStrategy": "GTC",
		"entrustType": "B",
		"fee": "0.004",
		"feeCoin": "btc",
		"leftAmount": "4",
		"leftAmountDeal": "4",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-1869967-68728",
		"orderStatus": "01",
		"rightAmount": "249782.56",
		"rightAmountDeal": "249754.29",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "HER21635991697",
		"terminalType": "API",
		"trades": [{
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-1869967-68728",
			"buyOrderPrice": "62445.64",
			"buyerFee": "0.001",
			"buyerFeeCoin": "btc",
			"buyerId": 5712839,
			"buyerMatchType": "T",
			"dealPrice": "62430",
			"dealTimestamp": "1635991697",
			"leftAmount": "1",
			"leftCoin": "btc",
			"matchSerialNo": "376171478089307",
			"rightAmount": "62430",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-1869967-68671",
			"sellOrderPrice": "62430",
			"sellerFee": "62.43",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "M",
			"symbol": "btc_usdt",
			"tradeId": "376182395287228"
		}, {
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-1869967-68728",
			"buyOrderPrice": "62445.64",
			"buyerFee": "0.003",
			"buyerFeeCoin": "btc",
			"buyerId": 5712839,
			"buyerMatchType": "T",
			"dealPrice": "62441.43",
			"dealTimestamp": "1635991697",
			"leftAmount": "3",
			"leftCoin": "btc",
			"matchSerialNo": "376171478089308",
			"rightAmount": "187324.29",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-1869967-68724",
			"sellOrderPrice": "62441.43",
			"sellerFee": "187.32429",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "M",
			"symbol": "btc_usdt",
			"tradeId": "376182395287229"
		}]
	}
```

```
POST /v1/exchange/spot/order (HMAC SHA256)
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| averagePrice  | string | 否 | 平均价 |
| createTime  | int | 否 | 创建时间 |
| entrustMode  | string | 否 | 委单模式：10(限价)、11(止盈止损限价)、20(市价)、21(止盈止损市价) |
| entrustPrice  | string | 否 | 委单单价 |
| entrustSource  | string | 否 | 委托来源：00(普通)、01（闪兑） |
| entrustStrategy  | string | 否 | 委托策略：GTC（普通限价单）、MAKER（只做Maker）、IOC（订单若不能立即成交则未成交的部分立即取消）、FOK（订单若不能全部成交则立即全部取消） |
| entrustType  | string | 否 | 委单类型：B(买)、S(卖) |
| fee  | string | 否 | 手续费 |
| feeCoin  | string | 否 | 手续费币种 |
| leftAmount  | string | 否 | 左币金额 |
| leftAmountDeal  | string | 否 | 左币已成交 |
| leftCoin  | string | 否 | 左币 |
| mid  | int | 否 | 会员编号 |
| orderNo  | string | 否 | 订单号 |
| orderStatus  | string | 否 | 委单状态:("00", "交易中"),("01", "交易完成"),("02", "交易失败"),("09", "推送撮合中"),("10", "撮合中"),("11", "部分撮合"),("20", "撤销中"),("21", "撤销完成") |
| rightAmount  | string | 否 | 右边成交金额 |
| rightAmountDeal  | string | 否 | 右币已处理金额 |
| rightCoin  | string | 否 | 右币 |
| symbol  | string | 否 | 交易对 |
| terminalCode  | string | 否 | 终端编号 |
| terminalType  | string | 否 | 终端类型：Android（安卓）、IOS（ios）、WEB（web 端）、API（api） |
| trades  | list | 否 | 成交列表 |

trades

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| buyOrderFullDeal  | boolean | 否 | 买单是否完全成交 |
| buyOrderNo  | string | 否 | 买单单号 |
| buyOrderPrice  | string | 否 | 买单下单价 |
| buyerFee  | string | 否 | 买方手续费 |
| buyerFeeCoin  | string | 否 | 买方手续费币种 |
| buyerId  | int | 否 | 卖方编号 |
| buyerMatchType  | string | 否 | 买方撮合类型:M（被撮合）、T（撮合） |
| dealPrice  | string | 否 | 成交价 |
| dealTimestamp  | string | 否 | 成交时间 |
| leftAmount  | string | 否 | 左币数量 |
| leftCoin  | string | 否 | 左币 |
| matchSerialNo  | string | 否 | 撮合流水号 |
| rightAmount  | string | 否 | 右币数量 |
| rightCoin  | string | 否 | 右币 |
| sellOrderFullDeal  | boolean | 否 | 卖单是否完全成交 |
| sellOrderNo  | string | 否 | 卖单单号 |
| sellOrderPrice  | string | 否 | 卖单下单价 |
| sellerFee  | string | 否 |卖方手续费 |
| sellerFeeCoin  | string | 否 |卖方手续费币种 |
| sellerId  | int | 否 | 卖方编号 |
| sellerMatchType  | string | 否 | 卖方撮合类型:M（被撮合）、T（撮合） |
| symbol  | string | 否 | 交易对 |
| tradeId  | string | 否 | 订单ID |

发送下单。

**参数:**

```json
{
	"entrustAmount": "1",
	"entrustPrice": "61620.7100",
	"triggerPrice": "61620.7100",
	"entrustType": "B",
	"symbol": "btc_usdt",
	"entrustSource": "01",
	"entrustStrategy": "GTC"
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| entrustAmount  | string | 否 | 委托数量 |
| entrustType  | string | 否 | 委托类型:B（买）、S（卖） |
| entrustSource  | string | 否 | 委托来源：00(普通) |
| symbol  | string | 否 | 交易对 |
| entrustMode  | string | 否 | 委单模式：10(限价)、11(止盈止损限价)、20(市价)、21(止盈止损市价) |
| entrustStrategy  | string | 是 | 委托策略：GTC（普通委单）、MAKER（只做Maker）、IOC（订单若不能立即成交则未成交的部分立即取消）、FOK（订单若不能全部成交则立即全部取消） |
| entrustPrice  | string | 是 | 委托价 |
| triggerPrice  | string | 是 | 触发价 |

基于订单 entrustMode不同，强制要求某些参数:
|  类型   |强制要求的参数|
|  ----   |----  |
| 10    | entrustStrategy,entrustPrice |
| 11    | triggerPrice,entrustPrice |

其他信息:
- 当 entrustMode 为10 时，可设置entrustStrategy,其他类型只支持GTC
- 当 entrustMode 为20 时，下买单时 entrustAmount 为右币冻结的数量，下卖单时entrustAmount为左币冻结的数量，最终获得的数量以市场最终成交为准；
  

**数据源:** 数据库

**权重(IP):1**


### 撤销订单

> **响应**

```json
{
		"averagePrice": "31217.8575",
		"createTime": 1635991697,
		"entrustSource":"01",
		"entrustMode": "10",
		"entrustPrice": "62445.64",
		"entrustStrategy": "GTC",
		"entrustType": "B",
		"fee": "0.004",
		"feeCoin": "btc",
		"leftAmount": "4",
		"leftAmountDeal": "4",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-1869967-68728",
		"orderStatus": "01",
		"rightAmount": "249782.56",
		"rightAmountDeal": "249754.29",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "HER21635991697",
		"terminalType": "API",
		"trades": [{
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-1869967-68728",
			"buyOrderPrice": "62445.64",
			"buyerFee": "0.001",
			"buyerFeeCoin": "btc",
			"buyerId": 5712839,
			"buyerMatchType": "T",
			"dealPrice": "62430",
			"dealTimestamp": "1635991697",
			"leftAmount": "1",
			"leftCoin": "btc",
			"matchSerialNo": "376171478089307",
			"rightAmount": "62430",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-1869967-68671",
			"sellOrderPrice": "62430",
			"sellerFee": "62.43",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "M",
			"symbol": "btc_usdt",
			"tradeId": "376182395287228"
		}, {
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-1869967-68728",
			"buyOrderPrice": "62445.64",
			"buyerFee": "0.003",
			"buyerFeeCoin": "btc",
			"buyerId": 5712839,
			"buyerMatchType": "T",
			"dealPrice": "62441.43",
			"dealTimestamp": "1635991697",
			"leftAmount": "3",
			"leftCoin": "btc",
			"matchSerialNo": "376171478089308",
			"rightAmount": "187324.29",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-1869967-68724",
			"sellOrderPrice": "62441.43",
			"sellerFee": "187.32429",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "M",
			"symbol": "btc_usdt",
			"tradeId": "376182395287229"
		}]
	}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| averagePrice  | string | 否 | 平均价 |
| createTime  | int | 否 | 创建时间 |
| entrustMode  | string | 否 | 委单模式：10(限价)、11(止盈止损限价)、20(市价)、21(止盈止损市价) |
| entrustPrice  | string | 否 | 委单单价 |
| entrustSource  | string | 否 | 委托来源：00(普通)、01（闪兑） |
| entrustStrategy  | string | 否 | 委托策略：GTC（普通限价单）、MAKER（只做Maker）、IOC（订单若不能立即成交则未成交的部分立即取消）、FOK（订单若不能全部成交则立即全部取消） |
| entrustType  | string | 否 | 委单类型：B(买)、S(卖) |
| fee  | string | 否 | 手续费 |
| feeCoin  | string | 否 | 手续费币种 |
| leftAmount  | string | 否 | 左币金额 |
| leftAmountDeal  | string | 否 | 左币已成交 |
| leftCoin  | string | 否 | 左币 |
| mid  | int | 否 | 会员编号 |
| orderNo  | string | 否 | 订单号 |
| orderStatus  | string | 否 | 委单状态:("00", "交易中"),("01", "交易完成"),("02", "交易失败"),("09", "推送撮合中"),("10", "撮合中"),("11", "部分撮合"),("20", "撤销中"),("21", "撤销完成") |
| rightAmount  | string | 否 | 右边成交金额 |
| rightAmountDeal  | string | 否 | 右币已处理金额 |
| rightCoin  | string | 否 | 右币 |
| symbol  | string | 否 | 交易对 |
| terminalCode  | string | 否 | 终端编号 |
| terminalType  | string | 否 | 终端类型：Android（安卓）、IOS（ios）、WEB（web 端）、API（api） |
| trades  | list | 否 | 成交列表 |

trades

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| buyOrderFullDeal  | boolean | 否 | 买单是否完全成交 |
| buyOrderNo  | string | 否 | 买单单号 |
| buyOrderPrice  | string | 否 | 买单下单价 |
| buyerFee  | string | 否 | 买方手续费 |
| buyerFeeCoin  | string | 否 | 买方手续费币种 |
| buyerId  | int | 否 | 卖方编号 |
| buyerMatchType  | string | 否 | 买方撮合类型:M（被撮合）、T（撮合） |
| dealPrice  | string | 否 | 成交价 |
| dealTimestamp  | string | 否 | 成交时间 |
| leftAmount  | string | 否 | 左币数量 |
| leftCoin  | string | 否 | 左币 |
| matchSerialNo  | string | 否 | 撮合流水号 |
| rightAmount  | string | 否 | 右币数量 |
| rightCoin  | string | 否 | 右币 |
| sellOrderFullDeal  | boolean | 否 | 卖单是否完全成交 |
| sellOrderNo  | string | 否 | 卖单单号 |
| sellOrderPrice  | string | 否 | 卖单下单价 |
| sellerFee  | string | 否 |卖方手续费 |
| sellerFeeCoin  | string | 否 |卖方手续费币种 |
| sellerId  | int | 否 | 卖方编号 |
| sellerMatchType  | string | 否 | 卖方撮合类型:M（被撮合）、T（撮合） |
| symbol  | string | 否 | 交易对 |
| tradeId  | string | 否 | 订单ID |

```
DELETE /v1/exchange/spot/entrust/{orderNo} (HMAC SHA256)
```


取消有效订单。

**参数:**

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| orderNo  | string | 否 | 订单编号 |

发起撤单之后，订单会先处于撤单中，等待撮合中心处理后会转为撤单成功。

**权重(IP):1**

### 撤销单一交易对的所有挂单 


```
POST /v1/exchange/spot/cancel/all
```

撤销单一交易对下所有挂单。

**参数:**

BODY
```json
{
	"symbol": "btc_usdt"
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| symbol  | string | 否 | 交易对编号 |

发起撤单之后，订单会先处于撤单中，等待撮合中心处理后会转为撤单成功。

**数据源:** 数据库

**权重(IP):1**

### 查询订单

> **响应**

```json
{
		"averagePrice": "31217.8575",
		"createTime": 1635991697,
		"entrustSource":"01",
		"entrustMode": "10",
		"entrustPrice": "62445.64",
		"entrustStrategy": "GTC",
		"entrustType": "B",
		"fee": "0.004",
		"feeCoin": "btc",
		"leftAmount": "4",
		"leftAmountDeal": "4",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-1869967-68728",
		"orderStatus": "01",
		"rightAmount": "249782.56",
		"rightAmountDeal": "249754.29",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "HER21635991697",
		"terminalType": "API",
		"trades": [{
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-1869967-68728",
			"buyOrderPrice": "62445.64",
			"buyerFee": "0.001",
			"buyerFeeCoin": "btc",
			"buyerId": 5712839,
			"buyerMatchType": "T",
			"dealPrice": "62430",
			"dealTimestamp": "1635991697",
			"leftAmount": "1",
			"leftCoin": "btc",
			"matchSerialNo": "376171478089307",
			"rightAmount": "62430",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-1869967-68671",
			"sellOrderPrice": "62430",
			"sellerFee": "62.43",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "M",
			"symbol": "btc_usdt",
			"tradeId": "376182395287228"
		}, {
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-1869967-68728",
			"buyOrderPrice": "62445.64",
			"buyerFee": "0.003",
			"buyerFeeCoin": "btc",
			"buyerId": 5712839,
			"buyerMatchType": "T",
			"dealPrice": "62441.43",
			"dealTimestamp": "1635991697",
			"leftAmount": "3",
			"leftCoin": "btc",
			"matchSerialNo": "376171478089308",
			"rightAmount": "187324.29",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-1869967-68724",
			"sellOrderPrice": "62441.43",
			"sellerFee": "187.32429",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "M",
			"symbol": "btc_usdt",
			"tradeId": "376182395287229"
		}]
	}
```

```
GET /v1/exchange/spot/entrust/{orderNo} (HMAC SHA256)
```

查询订单状态。

**参数:**

| 名称              | 类型   | 是否必需 | 描述                 |
| :---------------- | :----- | :------- | :------------------- |
| orderNo           | LONG   | YES       |                      |



**数据源:** 数据库

**权重(IP):2**

### 现货当前挂单

> **响应**


```json
{
	"list": [{
		"averagePrice": "10232.36333333",
		"createTime": 1636076701,
		"entrustMode": "10",
		"entrustPrice": "61394.19",
		"entrustStrategy": "GTC",
		"entrustType": "B",
		"fee": "0.006",
		"feeCoin": "btc",
		"leftAmount": "7",
		"leftAmountDeal": "6",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5427115-93051",
		"orderStatus": "10",
		"rightAmount": "429759.33",
		"rightAmountDeal": "368365.08",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "HER21636076701",
		"terminalType": "API",
		"trades": [{
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-5427115-93051",
			"buyOrderPrice": "61394.19",
			"buyerFee": "0.006",
			"buyerFeeCoin": "btc",
			"buyerId": 5712839,
			"buyerMatchType": "T",
			"dealPrice": "61394.18",
			"dealTimestamp": "1636076701",
			"leftAmount": "6",
			"leftCoin": "btc",
			"matchSerialNo": "376535531136761",
			"rightAmount": "368365.08",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-5427115-93050",
			"sellOrderPrice": "61394.18",
			"sellerFee": "368.36508",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "M",
			"symbol": "btc_usdt",
			"tradeId": "376529788559194"
		}]
	}, {
		"averagePrice": "30690.04",
		"createTime": 1636076692,
		"entrustMode": "10",
		"entrustPrice": "61380.08",
		"entrustStrategy": "GTC",
		"entrustType": "B",
		"fee": "0.002",
		"feeCoin": "btc",
		"leftAmount": "4",
		"leftAmountDeal": "2",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5427115-93038",
		"orderStatus": "10",
		"rightAmount": "245520.32",
		"rightAmountDeal": "122760.16",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "HER21636076692",
		"terminalType": "API",
		"trades": [{
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-5427115-93038",
			"buyOrderPrice": "61380.08",
			"buyerFee": "0.002",
			"buyerFeeCoin": "btc",
			"buyerId": 5712839,
			"buyerMatchType": "T",
			"dealPrice": "61380.08",
			"dealTimestamp": "1636076692",
			"leftAmount": "2",
			"leftCoin": "btc",
			"matchSerialNo": "376535531136754",
			"rightAmount": "122760.16",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-5427115-93035",
			"sellOrderPrice": "61380.08",
			"sellerFee": "122.76016",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "M",
			"symbol": "btc_usdt",
			"tradeId": "376529788559187"
		}]
	}, {
		"averagePrice": "0",
		"createTime": 1636076680,
		"entrustMode": "10",
		"entrustPrice": "0",
		"entrustStrategy": "GTC",
		"entrustType": "B",
		"fee": "0",
		"feeCoin": "btc",
		"leftAmount": "1",
		"leftAmountDeal": "0",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5427115-93019",
		"orderStatus": "10",
		"rightAmount": "0",
		"rightAmountDeal": "0",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "HER21636076680",
		"terminalType": "API",
		"trades": []
	}, {
		"averagePrice": "40770.32666667",
		"createTime": 1636076556,
		"entrustMode": "10",
		"entrustPrice": "61160.55",
		"entrustStrategy": "GTC",
		"entrustType": "B",
		"fee": "0.006",
		"feeCoin": "btc",
		"leftAmount": "7",
		"leftAmountDeal": "6",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5427115-92874",
		"orderStatus": "10",
		"rightAmount": "428123.85",
		"rightAmountDeal": "366937.8",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "HER21636076556",
		"terminalType": "API",
		"trades": [{
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-5427115-92874",
			"buyOrderPrice": "61160.55",
			"buyerFee": "0.001",
			"buyerFeeCoin": "btc",
			"buyerId": 5712839,
			"buyerMatchType": "T",
			"dealPrice": "61147.11",
			"dealTimestamp": "1636076556",
			"leftAmount": "1",
			"leftCoin": "btc",
			"matchSerialNo": "376535531136679",
			"rightAmount": "61147.11",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-5427115-92850",
			"sellOrderPrice": "61147.11",
			"sellerFee": "61.14711",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "M",
			"symbol": "btc_usdt",
			"tradeId": "376529788559112"
		}, {
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-5427115-92874",
			"buyOrderPrice": "61160.55",
			"buyerFee": "0.001",
			"buyerFeeCoin": "btc",
			"buyerId": 5712839,
			"buyerMatchType": "T",
			"dealPrice": "61159.01",
			"dealTimestamp": "1636076556",
			"leftAmount": "1",
			"leftCoin": "btc",
			"matchSerialNo": "376535531136680",
			"rightAmount": "61159.01",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-5427115-92770",
			"sellOrderPrice": "61159.01",
			"sellerFee": "61.15901",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "M",
			"symbol": "btc_usdt",
			"tradeId": "376529788559113"
		}, {
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-5427115-92874",
			"buyOrderPrice": "61160.55",
			"buyerFee": "0.002",
			"buyerFeeCoin": "btc",
			"buyerId": 5712839,
			"buyerMatchType": "T",
			"dealPrice": "61160.55",
			"dealTimestamp": "1636076556",
			"leftAmount": "2",
			"leftCoin": "btc",
			"matchSerialNo": "376535531136681",
			"rightAmount": "122321.1",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-5427115-92872",
			"sellOrderPrice": "61160.55",
			"sellerFee": "122.3211",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "M",
			"symbol": "btc_usdt",
			"tradeId": "376529788559114"
		}, {
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-5427115-92874",
			"buyOrderPrice": "61160.55",
			"buyerFee": "0.002",
			"buyerFeeCoin": "btc",
			"buyerId": 5712839,
			"buyerMatchType": "T",
			"dealPrice": "61155.29",
			"dealTimestamp": "1636076556",
			"leftAmount": "2",
			"leftCoin": "btc",
			"matchSerialNo": "376535531136682",
			"rightAmount": "122310.58",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-5354344-74190",
			"sellOrderPrice": "61155.29",
			"sellerFee": "122.31058",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "M",
			"symbol": "btc_usdt",
			"tradeId": "376529788559115"
		}]
	}, {
		"averagePrice": "0",
		"createTime": 1636076300,
		"entrustMode": "10",
		"entrustPrice": "61044.43",
		"entrustStrategy": "GTC",
		"entrustType": "B",
		"fee": "0",
		"feeCoin": "btc",
		"leftAmount": "5",
		"leftAmountDeal": "0",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5427115-92583",
		"orderStatus": "10",
		"rightAmount": "305222.15",
		"rightAmountDeal": "0",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "HER21636076300",
		"terminalType": "API",
		"trades": []
	}, {
		"averagePrice": "0",
		"createTime": 1636076053,
		"entrustMode": "10",
		"entrustPrice": "0",
		"entrustStrategy": "GTC",
		"entrustType": "B",
		"fee": "0",
		"feeCoin": "btc",
		"leftAmount": "1",
		"leftAmountDeal": "0",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5354344-74843",
		"orderStatus": "10",
		"rightAmount": "0",
		"rightAmountDeal": "0",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "HER21636076053",
		"terminalType": "API",
		"trades": []
	}, {
		"averagePrice": "0",
		"createTime": 1636075594,
		"entrustMode": "10",
		"entrustPrice": "0",
		"entrustStrategy": "GTC",
		"entrustType": "B",
		"fee": "0",
		"feeCoin": "btc",
		"leftAmount": "1",
		"leftAmountDeal": "0",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5354344-74301",
		"orderStatus": "10",
		"rightAmount": "0",
		"rightAmountDeal": "0",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "HER21636075594",
		"terminalType": "API",
		"trades": []
	}, {
		"averagePrice": "0",
		"createTime": 1636075233,
		"entrustMode": "10",
		"entrustPrice": "0",
		"entrustStrategy": "GTC",
		"entrustType": "B",
		"fee": "0",
		"feeCoin": "btc",
		"leftAmount": "4",
		"leftAmountDeal": "0",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5354344-73901",
		"orderStatus": "10",
		"rightAmount": "0",
		"rightAmountDeal": "0",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "HER21636075233",
		"terminalType": "API",
		"trades": []
	}, {
		"averagePrice": "0",
		"createTime": 1636074419,
		"entrustMode": "10",
		"entrustPrice": "61029.43",
		"entrustStrategy": "GTC",
		"entrustType": "B",
		"fee": "0",
		"feeCoin": "btc",
		"leftAmount": "9",
		"leftAmountDeal": "0",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5281951-04211",
		"orderStatus": "10",
		"rightAmount": "549264.87",
		"rightAmountDeal": "0",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "HER21636074419",
		"terminalType": "API",
		"trades": []
	}, {
		"averagePrice": "0",
		"createTime": 1636074366,
		"entrustMode": "10",
		"entrustPrice": "61004.42",
		"entrustStrategy": "GTC",
		"entrustType": "B",
		"fee": "0",
		"feeCoin": "btc",
		"leftAmount": "1",
		"leftAmountDeal": "0",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5281951-04153",
		"orderStatus": "10",
		"rightAmount": "61004.42",
		"rightAmountDeal": "0",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "HER21636074366",
		"terminalType": "API",
		"trades": []
	}],
	"total": "27"
}
```
|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| total  | string | 否 | 总数 |
| list  | array | 否 | 记录列表 |

list

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| averagePrice  | string | 否 | 平均价 |
| createTime  | int | 否 | 创建时间 |
| entrustMode  | string | 否 | 委单模式：10(限价)、11(止盈止损限价)、20(市价)、21(止盈止损市价) |
| entrustPrice  | string | 否 | 委单单价 |
| entrustSource  | string | 否 | 委托来源：00(普通)、01（闪兑） |
| entrustStrategy  | string | 否 | 委托策略：GTC（普通限价单）、MAKER（只做Maker）、IOC（订单若不能立即成交则未成交的部分立即取消）、FOK（订单若不能全部成交则立即全部取消） |
| entrustType  | string | 否 | 委单类型：B(买)、S(卖) |
| fee  | string | 否 | 手续费 |
| feeCoin  | string | 否 | 手续费币种 |
| leftAmount  | string | 否 | 左币金额 |
| leftAmountDeal  | string | 否 | 左币已成交 |
| leftCoin  | string | 否 | 左币 |
| mid  | int | 否 | 会员编号 |
| orderNo  | string | 否 | 订单号 |
| orderStatus  | string | 否 | 委单状态:("00", "交易中"),("01", "交易完成"),("02", "交易失败"),("09", "推送撮合中"),("10", "撮合中"),("11", "部分撮合"),("20", "撤销中"),("21", "撤销完成") |
| rightAmount  | string | 否 | 右边成交金额 |
| rightAmountDeal  | string | 否 | 右币已处理金额 |
| rightCoin  | string | 否 | 右币 |
| symbol  | string | 否 | 交易对 |
| terminalCode  | string | 否 | 终端编号 |
| terminalType  | string | 否 | 终端类型：Android（安卓）、IOS（ios）、WEB（web 端）、API（api） |
| trades  | list | 否 | 成交列表 |

trades

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| buyOrderFullDeal  | boolean | 否 | 买单是否完全成交 |
| buyOrderNo  | string | 否 | 买单单号 |
| buyOrderPrice  | string | 否 | 买单下单价 |
| buyerFee  | string | 否 | 买方手续费 |
| buyerFeeCoin  | string | 否 | 买方手续费币种 |
| buyerId  | int | 否 | 卖方编号 |
| buyerMatchType  | string | 否 | 买方撮合类型:M（被撮合）、T（撮合） |
| dealPrice  | string | 否 | 成交价 |
| dealTimestamp  | string | 否 | 成交时间 |
| leftAmount  | string | 否 | 左币数量 |
| leftCoin  | string | 否 | 左币 |
| matchSerialNo  | string | 否 | 撮合流水号 |
| rightAmount  | string | 否 | 右币数量 |
| rightCoin  | string | 否 | 右币 |
| sellOrderFullDeal  | boolean | 否 | 卖单是否完全成交 |
| sellOrderNo  | string | 否 | 卖单单号 |
| sellOrderPrice  | string | 否 | 卖单下单价 |
| sellerFee  | string | 否 |卖方手续费 |
| sellerFeeCoin  | string | 否 |卖方手续费币种 |
| sellerId  | int | 否 | 卖方编号 |
| sellerMatchType  | string | 否 | 卖方撮合类型:M（被撮合）、T（撮合） |
| symbol  | string | 否 | 交易对 |
| tradeId  | string | 否 | 订单ID |

```
POST /v1/exchange/spot/entrust/p/{pageNum}-{pageSize} (HMAC SHA256)
```
**参数:**

| 名称       | 类型   | 是否必需 | 描述                 |
| :--------- | :----- | :------- | :------------------- |
| pageNum     | int | YES       |             页码：1开始          |
| pageSize | int   | YES       | 每页数量：最大100 |


BODY
```json
{
	"symbol": "btc_usdt",
	"startTime":1635955200,
	"endTime":1637510400,
	"entrustType":"B"
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| symbol  | string | 是 | 交易对编号 |
| startTime  | int | 是 | 开始时间 |
| endTime  | int | 是 | 结束时间 |
| entrustType  | string | 是 | 委单类型：B(买)、S(卖) |



- 不带symbol参数，会返回所有交易对的挂单


**权重(IP):** 3 单一交易对;
**40** 交易对参数缺失;

**数据源:** 数据库


### 现货成交历史 

> **响应**


```json
{
	"list": [{
		"buyOrderFullDeal": false,
		"buyOrderNo": "376-5752824-70471",
		"buyOrderPrice": "62063",
		"buyerFee": "0.00310315",
		"buyerFeeCoin": "usdt",
		"buyerId": 5712839,
		"buyerMatchType": "M",
		"contractCode": "btcw20211108",
		"dealPrice": "62063",
		"dealTimestamp": "1636083578",
		"leftAmount": "1",
		"leftCoin": "usdt",
		"matchSerialNo": "376571445669263",
		"rightAmount": "62.063",
		"rightCoin": "usdt",
		"sellOrderFullDeal": false,
		"sellOrderNo": "376-5752824-70480",
		"sellOrderPrice": "62063",
		"sellerFee": "0.00310315",
		"sellerFeeCoin": "usdt",
		"sellerId": 5712839,
		"sellerMatchType": "T",
		"tradeId": "376565506876348"
	}, {
		"buyOrderFullDeal": false,
		"buyOrderNo": "376-5752824-70056",
		"buyerFee": "0.00025",
		"buyerFeeCoin": "usdt",
		"buyerId": 5712839,
		"buyerMatchType": "T",
		"contractCode": "eosusdt",
		"dealPrice": "5",
		"dealTimestamp": "1636083549",
		"leftAmount": "1",
		"leftCoin": "usdt",
		"matchSerialNo": "376571445669227",
		"rightAmount": "5",
		"rightCoin": "usdt",
		"sellOrderFullDeal": false,
		"sellOrderNo": "376-5752824-70136",
		"sellOrderPrice": "5",
		"sellerFee": "0.00025",
		"sellerFeeCoin": "usdt",
		"sellerId": 5712839,
		"sellerMatchType": "M",
		"tradeId": "376565506876312"
	}, {
		"buyOrderFullDeal": false,
		"buyOrderNo": "376-5746029-91171",
		"buyOrderPrice": "5",
		"buyerFee": "0.00025",
		"buyerFeeCoin": "usdt",
		"buyerId": 5712839,
		"buyerMatchType": "M",
		"contractCode": "eosusdt",
		"dealPrice": "5",
		"dealTimestamp": "1636083540",
		"leftAmount": "1",
		"leftCoin": "usdt",
		"matchSerialNo": "376571445669217",
		"rightAmount": "5",
		"rightCoin": "usdt",
		"sellOrderFullDeal": false,
		"sellOrderNo": "376-5752824-70033",
		"sellOrderPrice": "5",
		"sellerFee": "0.00025",
		"sellerFeeCoin": "usdt",
		"sellerId": 5712839,
		"sellerMatchType": "T",
		"tradeId": "376565506876302"
	}],
	"total": "3"
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| total  | string | 否 | 总数 |
| list  | array | 否 | 记录列表 |

list

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| buyOrderFullDeal  | boolean | 否 | 买单是否完全成交 |
| buyOrderNo  | string | 否 | 买单单号 |
| buyOrderPrice  | string | 否 | 买单下单价 |
| buyerFee  | string | 否 | 买方手续费 |
| buyerFeeCoin  | string | 否 | 买方手续费币种 |
| buyerId  | int | 否 | 卖方编号 |
| buyerMatchType  | string | 否 | 买方撮合类型:M（被撮合）、T（撮合） |
| dealPrice  | string | 否 | 成交价 |
| dealTimestamp  | string | 否 | 成交时间 |
| leftAmount  | string | 否 | 左币数量 |
| leftCoin  | string | 否 | 左币 |
| matchSerialNo  | string | 否 | 撮合流水号 |
| rightAmount  | string | 否 | 右币数量 |
| rightCoin  | string | 否 | 右币 |
| sellOrderFullDeal  | boolean | 否 | 卖单是否完全成交 |
| sellOrderNo  | string | 否 | 卖单单号 |
| sellOrderPrice  | string | 否 | 卖单下单价 |
| sellerFee  | string | 否 |卖方手续费 |
| sellerFeeCoin  | string | 否 |卖方手续费币种 |
| sellerId  | int | 否 | 卖方编号 |
| sellerMatchType  | string | 否 | 卖方撮合类型:M（被撮合）、T（撮合） |
| symbol  | string | 否 | 交易对 |
| tradeId  | string | 否 | 订单ID |

```
GET /v1/exchange/spot/entrust/detail/p/{pageNum}-{pageSize} (HMAC SHA256)
```



获取账户指定交易对的成交历史

**参数:**

| 名称       | 类型   | 是否必需 | 描述                 |
| :--------- | :----- | :------- | :------------------- |
| pageNum     | int | YES       |             页码：1开始          |
| pageSize | int   | YES       | 每页数量：最大100 |

BODY
```json
{
	"symbol": "btc_usdt",
	"startTime":1635955200,
	"endTime":1637510400
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| symbol  | string | 是 | 交易对编号 |
| startTime  | int | 是 | 开始时间 |
| endTime  | int | 是 | 结束时间 |
| entrustType  | string | 是 | 委单类型：B(买)、S(卖) |

**数据源:** 数据库

**权重(IP):10**


## 合约行情

### 合约规范

> **响应**

```json
[{
	"baseCurrency": "btc",
	"contractInfos": [{
		"autoDeleveragingEnabled": true,
		"baseCurrency": "btc",
		"callPriceMethod": {
			"contractIndexSymbol": "her_btc_usdt_30m",
			"type": "1"
		},
		"contractCode": "btcusdt",
		"contractName": "btc\/usdt 永续",
		"contractType": "6",
		"createTime": 1635761526,
		"deliveryFee": "0.00005",
		"enable": true,
		"initMargin": "0.01",
		"leverage": "100",
		"maintMargin": "0.005",
		"makerFee": "0.00005",
		"marketMethod": {
			"indexSymbol": "her_btc_usdt",
			"marginEffect": "5000",
			"priceProtectInterval": "0.5",
			"type": "1"
		},
		"multiplier": "0.001",
		"pnlCalculation": "1",
		"quoteCurrency": "usdt",
		"rejectOpenTime": 0,
		"riskLimit": "600000000",
		"riskStep": "300000000",
		"settlementCurrency": "usdt",
		"sort": 0,
		"status": "1",
		"supportDepth": "1,2,3",
		"takerFee": "0.00005",
		"tradePairs": {
			"areaCode": "",
			"enable": false,
			"exchangeRunningNode": "",
			"extendedJson": "",
			"indexSort": 0,
			"matchRunningNode": "",
			"maxCount": "9000000",
			"maxCountDecimalDigit": 0,
			"maxPrice": "150000",
			"maxPriceDecimalDigit": 0,
			"minCount": "1",
			"minPrice": "0.0001",
			"priceRange": "0",
			"supportDepth": "1,2,3",
			"symbol": "btc_usdt",
			"tradeBoard": "00",
			"tradeMarket": "contract"
		}
	}, {
		"autoDeleveragingEnabled": true,
		"baseCurrency": "btc",
		"callPriceMethod": {
			"contractIndexSymbol": "her_btc_usdt_30m",
			"type": "1"
		},
		"contractCode": "btcw20211108",
		"contractName": "btc1108",
		"contractType": "1",
		"createTime": 1635748624,
		"deliveryFee": "0.00005",
		"deliveryTime": 1636362000,
		"enable": true,
		"initMargin": "0.01",
		"leverage": "100",
		"maintMargin": "0.005",
		"makerFee": "0.00005",
		"marketMethod": {
			"indexSymbol": "her_btc_usdt",
			"marginEffect": "5000",
			"priceProtectInterval": "0.5",
			"type": "1"
		},
		"multiplier": "0.001",
		"pnlCalculation": "1",
		"quoteCurrency": "usdt",
		"rejectOpenTime": 0,
		"remainTime": "596013",
		"riskLimit": "600000000",
		"riskStep": "300000000",
		"settlementCurrency": "usdt",
		"sort": 5,
		"status": "1",
		"supportDepth": "1,2,3",
		"takerFee": "0.00005",
		"tradePairs": {
			"areaCode": "",
			"enable": false,
			"exchangeRunningNode": "",
			"extendedJson": "",
			"indexSort": 0,
			"matchRunningNode": "",
			"maxCount": "9000000",
			"maxCountDecimalDigit": 0,
			"maxPrice": "150000",
			"maxPriceDecimalDigit": 0,
			"minCount": "1",
			"minPrice": "0.0001",
			"priceRange": "0",
			"supportDepth": "1,2,3",
			"symbol": "btc_usdt",
			"tradeBoard": "00",
			"tradeMarket": "contract"
		}
	}, {
		"autoDeleveragingEnabled": true,
		"baseCurrency": "btc",
		"callPriceMethod": {
			"contractIndexSymbol": "her_btc_usdt_30m",
			"type": "1"
		},
		"contractCode": "btcm20221226",
		"contractName": "btc1226",
		"contractType": "2",
		"createTime": 1635748564,
		"deliveryFee": "0.00005",
		"deliveryTime": 1640509200,
		"enable": true,
		"initMargin": "0.01",
		"leverage": "100",
		"maintMargin": "0.005",
		"makerFee": "0.00005",
		"marketMethod": {
			"indexSymbol": "her_btc_usdt",
			"marginEffect": "5000",
			"priceProtectInterval": "0.5",
			"type": "1"
		},
		"multiplier": "0.001",
		"pnlCalculation": "1",
		"quoteCurrency": "usdt",
		"rejectOpenTime": 0,
		"remainTime": "4743243",
		"riskLimit": "600000000",
		"riskStep": "300000000",
		"settlementCurrency": "usdt",
		"sort": 5,
		"status": "1",
		"supportDepth": "1,2,3",
		"takerFee": "0.00005",
		"tradePairs": {
			"areaCode": "",
			"enable": false,
			"exchangeRunningNode": "",
			"extendedJson": "",
			"indexSort": 0,
			"matchRunningNode": "",
			"maxCount": "9000000",
			"maxCountDecimalDigit": 0,
			"maxPrice": "150000",
			"maxPriceDecimalDigit": 0,
			"minCount": "1",
			"minPrice": "0.0001",
			"priceRange": "0",
			"supportDepth": "1,2,3",
			"symbol": "btc_usdt",
			"tradeBoard": "00",
			"tradeMarket": "contract"
		}
	}]
}, {
	"baseCurrency": "eth",
	"contractInfos": [{
		"autoDeleveragingEnabled": true,
		"baseCurrency": "eth",
		"callPriceMethod": {
			"contractIndexSymbol": "her_eth_usdt_30m",
			"type": "1"
		},
		"contractCode": "ethusdt",
		"contractName": "eth\/usdt 永续",
		"contractType": "6",
		"createTime": 1635751144,
		"deliveryFee": "0.00005",
		"enable": true,
		"initMargin": "0.01",
		"leverage": "100",
		"maintMargin": "0.005",
		"makerFee": "0.00005",
		"marketMethod": {
			"indexSymbol": "her_eth_usdt",
			"marginEffect": "500",
			"priceProtectInterval": "0.5",
			"type": "1"
		},
		"multiplier": "0.1",
		"pnlCalculation": "1",
		"quoteCurrency": "usdt",
		"rejectOpenTime": 0,
		"riskLimit": "600000000",
		"riskStep": "300000000",
		"settlementCurrency": "usdt",
		"sort": 0,
		"status": "1",
		"supportDepth": "1,2,3",
		"takerFee": "0.00005",
		"tradePairs": {
			"areaCode": "",
			"enable": false,
			"exchangeRunningNode": "",
			"extendedJson": "",
			"indexSort": 5,
			"matchRunningNode": "",
			"maxCount": "9000000",
			"maxCountDecimalDigit": 0,
			"maxPrice": "10000",
			"maxPriceDecimalDigit": 0,
			"minCount": "1",
			"minPrice": "0",
			"priceRange": "0",
			"supportDepth": "1,2,3",
			"symbol": "eth_usdt",
			"tradeBoard": "01",
			"tradeMarket": "contract"
		}
	}]
}, {
	"baseCurrency": "dot",
	"contractInfos": [{
		"autoDeleveragingEnabled": true,
		"baseCurrency": "dot",
		"callPriceMethod": {
			"contractIndexSymbol": "her_dot_usdt",
			"type": "1"
		},
		"contractCode": "dotusdt",
		"contractName": "dot\/usdt 永续",
		"contractType": "6",
		"createTime": 1635748502,
		"deliveryFee": "0.00005",
		"enable": true,
		"initMargin": "0.02",
		"leverage": "50",
		"maintMargin": "0.01",
		"makerFee": "0.00005",
		"marketMethod": {
			"indexSymbol": "her_dot_usdt",
			"marginEffect": "5000",
			"priceProtectInterval": "0.05",
			"type": "1"
		},
		"multiplier": "1",
		"pnlCalculation": "1",
		"quoteCurrency": "usdt",
		"rejectOpenTime": 0,
		"riskLimit": "600000000",
		"riskStep": "300000000",
		"settlementCurrency": "usdt",
		"sort": 0,
		"status": "1",
		"supportDepth": "1,2,3,4,5,6",
		"takerFee": "0.00005",
		"tradePairs": {
			"areaCode": "",
			"enable": false,
			"exchangeRunningNode": "",
			"extendedJson": "",
			"indexSort": 0,
			"matchRunningNode": "",
			"maxCount": "9000000",
			"maxCountDecimalDigit": 0,
			"maxPrice": "150000",
			"maxPriceDecimalDigit": 4,
			"minCount": "1",
			"minPrice": "0.0001",
			"priceRange": "0",
			"supportDepth": "1,2,3,4,5,6",
			"symbol": "dot_usdt",
			"tradeBoard": "00",
			"tradeMarket": "contract"
		}
	}]
}, {
	"baseCurrency": "eos",
	"contractInfos": [{
		"autoDeleveragingEnabled": true,
		"baseCurrency": "eos",
		"callPriceMethod": {
			"contractIndexSymbol": "her_eos_usdt",
			"type": "1"
		},
		"contractCode": "eosusdt",
		"contractName": "eos\/usdt 永续",
		"contractType": "6",
		"createTime": 1635748579,
		"deliveryFee": "0.00005",
		"enable": true,
		"initMargin": "0.02",
		"leverage": "50",
		"maintMargin": "0.01",
		"makerFee": "0.00005",
		"marketMethod": {
			"indexSymbol": "her_eos_usdt",
			"marginEffect": "5000",
			"priceProtectInterval": "0.05",
			"type": "1"
		},
		"multiplier": "1",
		"pnlCalculation": "1",
		"quoteCurrency": "usdt",
		"rejectOpenTime": 0,
		"riskLimit": "600000000",
		"riskStep": "300000000",
		"settlementCurrency": "usdt",
		"sort": 0,
		"status": "1",
		"supportDepth": "1,2,3",
		"takerFee": "0.00005",
		"tradePairs": {
			"areaCode": "",
			"enable": false,
			"exchangeRunningNode": "",
			"extendedJson": "",
			"indexSort": 0,
			"matchRunningNode": "",
			"maxCount": "9000000",
			"maxCountDecimalDigit": 0,
			"maxPrice": "150000",
			"maxPriceDecimalDigit": 0,
			"minCount": "1",
			"minPrice": "0.0001",
			"priceRange": "0",
			"supportDepth": "1,2,3",
			"symbol": "eos_usdt",
			"tradeBoard": "00",
			"tradeMarket": "contract"
		}
	}]
}]
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| baseCurrency  | string | 否 | 基础币种 |
| contractInfos  | array | 是 | 基础币种内合约列表 |

contractInfos

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| contractCode  | string | 否 | 合约编号 |
| contractName  | string | 否 | 合约名称 |
| contractType  | string | 否 | 合约类型:1(周合约)、2（月合约）、3(季度合约)、4(半年合约)、5(双周合约)、6(永续合约) |
| createTime  | string | 否 | 创建时间戳（单位：秒） |
| initMargin  | string | 否 | 起始保证金 |
| leverage  | string | 否 | 最大杠杆 |
| maintMargin  | string | 否 | 维持保证金 |
| makerFee  | string | 否 | maker 交易手续费 |
| marketMethod  | object | 否 | 价格计算方式 |
| multiplier  | string | 否 | 乘数|
| pnlCalculation  | string | 否 |盈亏计算方式:1（正向合约）、2（反向合约）、3（双向合约）|
| quoteCurrency  | string | 否 | 计价币种|
| rejectOpenTime  | string | 是 | 交割前禁止开仓小时|
| riskLimit  | string | 否 | 风险限额|
| riskStep  | string | 否 | 风险限额递增值|
| settlementCurrency  | string | 是 |结算币种|
| sort  | string | 否 | 排序|
| status  | string | 否 |状态：1（开盘中）、2（已收盘）、3（交割中）、4（撤单中）|
| supportDepth  | string | 否 | 支持的深度，逗号隔开|
| takerFee  | string | 否 | taker 手续费|
| autoDeleveragingEnabled  | boolrean | 否 |是否启用自动减仓|
| baseCurrency  | string | 否 | 基础币种|
| callPriceMethod  | object | 否 | 结算价格计算方法 |
| tradePairs  | object | 否 | 合约交易对|
| deliveryTime  | int | 是 | 交割时间戳（单位：秒）|
| deliveryFee  | string | 是 | 交割手续费|
| remainTime  | string | 是 | 合约剩余交割时间（单位:秒）|

tradePairs

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| symbol  | string | 否 | 交易对编号 |
| indexSort  | int | 否 | 交易对排序 |
| maxCount  | string | 是 | 最大下单数量 |
| maxCountDecimalDigit  | int | 是 | 最大下单数量精度 |
| maxPrice  | string | 是 | 最大下单价格 |
| maxPriceDecimalDigit  | int | 是 | 最大下单价格精度 |
| minCount  | string | 是 | 最小下单数量 |
| minPrice  | string | 是 | 最小下单价格 |
| priceRange  | string | 是 | 价格波动范围 |
| supportDepth  | string | 是 | 支持深度 |
| tradeBoard  | string | 是 | 交易板块编号:00(主板)、01(创业版) |
| tradeMarket  | array | 是 | 交易市场编号： spot（现货）、contract(合约)、otc（场外）|

marketMethod

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| type  | string | 否 | 价格计算方法:1(合理价格标记) |
| marginEffect  | string | 否 | 保证金影响额 |
| priceProtectInterval  | string | 否 | 价格保证区间 |
| indexSymbol  | string | 否 | 指数交易对 |

callPriceMethod

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| type  | string | 否 | 交割价格计算方法:1(时间加权平均价格)、2（交易量加权平均价格） |
| contractIndexSymbol  | string | 否 | 指数交易对 |

```
GET /v1/exchange/contract/online
```

获取合约交易规则和交易对信息。

**数据源:** 缓存

**权重(IP):1**

### 深度信息  

> **响应**

```json
{
	"buy": [{
		"amount": "1",
		"entrustType": "B",
		"price": "67653.29",
		"total": "0"
	}],
	"sell": [{
		"amount": "9",
		"entrustType": "S",
		"price": "67666.53",
		"total": "0"
	}],
	"symbol": "btcusdt",
	"timestamp": "1636429063333",
	"tradeMarket": "contract"
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| amount | string | 否 |数量 |
| entrustType | string | 否 |委托类型:B（买）、S（卖） |
| price | string | 否 |价格 |
| total | string | 否 |成交额 |

`GET /v1/exchange/contract/depth/{contractCode}/1`


**参数:**

| 名称   | 类型   | 是否必需 | 描述                                                         |
| :----- | :----- | :------- | :----------------------------------------------------------- |
| contractCode | STRING | YES      |

**数据源:** 缓存

**权重(IP):10**

### 近期成交列表

> **响应**

```json
[{
	"dealAmount": "2",
	"dealPrice": "67680.49",
	"dealSide": "B",
	"dealTimestamp": "1636427410",
	"contractCode": "btcusdt"
}, {
	"dealAmount": "3",
	"dealPrice": "67685",
	"dealSide": "B",
	"dealTimestamp": "1636427410",
	"contractCode": "btcusdt"
}]
```
```
GET /v1/exchange/contract/trades/{contractCode}/{limit}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| dealAmount | string | 否 |成交量 |
| dealPrice | string | 否 |成交价 |
| dealSide | string | 否 |成交方向：B（买）、S（卖） |
| dealTimestamp | string | 否 |成交时间 |
| contractCode | string | 否 |合约编号 |


获取近期成交

**参数:**

| 名称   | 类型   | 是否必需 | 描述                   |
| :----- | :----- | :------- | :--------------------- |
| symbol | STRING | YES      |                        |
| limit  | INT    | YES       |  最大值 50. |

**数据源:** 缓存

**权重(IP):1**

### K线数据

> **响应**

```json
[
["1636428360", "67569.77", "67576.6", "67569.77", "67576.6", "25", "11.00", 10]
]
```
```
GET /v1/exchange/contract/kline/{contractCode}/{klineTime}/{timestamp}/{count}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| arr[0] | string | 否 |时间戳 |
| arr[1] | string | 否 |开盘价 |
| arr[2] | string | 否 |最高成交价 |
| arr[3] | string | 否 |最低成交价 |
| arr[4] | string | 否 |收盘价 |
| arr[5] | string | 否 |成交量 |
| arr[6] | string | 否 |右币成交额 |
| arr[7] | string | 否 |交易成交笔数 |

每根K线代表一个交易对。
每根K线的开盘时间可视为唯一ID

**参数:**

| 名称      | 类型   | 是否必需 | 描述                  |
| :-------- | :----- | :------- | :-------------------- |
| contractCode    | STRING | YES      |                       |
| klineTime  | STRING   | YES      | 详见枚举定义：K线间隔 |
| timestamp | LONG   | YES       |   截止时间                    |
| count     | INT    | YES       |  最大 300.  |


**数据源:** 缓存

**权重(IP):10**


## 合约钱包相关


### 获取现货资产

获取针对用户的所有合约资产。

> **响应**

```json
{
	"accounts": [{
		"btc": "10011",
		"canRecharge": true,
		"canWithdraw": true,
		"cash": "10011",
		"cny": "4097187353.94",
		"contractHost": "",
		"currency": "btc",
		"freeze": "0",
		"locked": "0",
		"mid": 18,
		"productMode": "exchange",
		"submode": "00",
		"usdt": "631986621.42",
		"walletAddress": ""
	}, {
		"btc": "0",
		"canRecharge": true,
		"canWithdraw": true,
		"cash": "10000",
		"cny": "0",
		"contractHost": "eth",
		"currency": "bzz",
		"freeze": "0",
		"locked": "0",
		"mid": 18,
		"productMode": "exchange",
		"submode": "00",
		"usdt": "0",
		"walletAddress": ""
	}, {
		"btc": "0",
		"canRecharge": true,
		"canWithdraw": true,
		"cash": "10000",
		"cny": "0",
		"contractHost": "",
		"currency": "cny",
		"freeze": "0",
		"locked": "0",
		"mid": 18,
		"productMode": "exchange",
		"submode": "00",
		"usdt": "0",
		"walletAddress": ""
	}],
	"totalBTC": "10737.65206",
	"totalCNY": "4394343969.34",
	"totalUSDT": "677792991.292"
}
```

`GET /v1/ma/account/assets/sub/02 (HMAC SHA256)`

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| totalBTC  | string | 否 | BTC总资产 |
| totalCNY  | string | 否 | CNY总资产 |
| totalUSDT  | string | 否 | USDT总资产 |
| accounts  | array | 是 | 所有币种账户 |

accounts

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| btc  | string | 否 | BTC资产 |
| canRecharge  | string | 否 | 是否允许充值 |
| canWithdraw  | string | 否 | 是否允许提现 |
| cash  | string | 否 | 现金户 |
| currency  | string | 否 | 币种 |
| freeze  | string | 否 | 冻结户 |
| locked  | string | 否 | 锁仓户 |

**权重(IP):1**

## 合约交易

### 测试下单

> **响应**

```
{}
POST /v1/exchange/contract/order/test (HMAC SHA256)
```

用于测试订单请求，但不会提交到撮合引擎

**权重:** 1

**参数:**

同于 `POST /v1/exchange/spot/order`

### 下单

> **Response**

```json
{
		"averagePrice": "62063",
		"contractCode": "btcw20211108",
		"createTime": 1636083578,
		"entrustMode": "10",
		"entrustPrice": "62063",
		"entrustSource": "1",
		"entrustType": "S",
		"fee": "0.00310315",
		"feeCoin": "usdt",
		"leftAmount": "2",
		"leftAmountDeal": "1",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5752824-70480",
		"orderStatus": "10",
		"rightAmount": "124.126",
		"rightAmountDeal": "62.063",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "her2b44e8e20-f755-11eb-b8c9-37c73197ff9d",
		"terminalType": "Web",
		"trades": [{
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-5752824-70471",
			"buyOrderPrice": "62063",
			"buyerFee": "0.00310315",
			"buyerFeeCoin": "usdt",
			"buyerId": 5712839,
			"buyerMatchType": "M",
			"contractCode": "btcw20211108",
			"dealPrice": "62063",
			"dealTimestamp": "1636083578",
			"leftAmount": "1",
			"leftCoin": "usdt",
			"matchSerialNo": "376571445669263",
			"rightAmount": "62.063",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-5752824-70480",
			"sellOrderPrice": "62063",
			"sellerFee": "0.00310315",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "T",
			"tradeId": "376565506876348"
		}]
}
```


|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| averagePrice  | string | 否 | 平均价 |
| createTime  | int | 否 | 创建时间 |
| contractCode  | string | 否 | 合约编号 |
| entrustMode  | string | 否 | 委单模式：10(限价)、11(止盈止损限价)、20(市价)、21(止盈止损市价) |
| entrustPrice  | string | 否 | 委单单价 |
| entrustSource  | string | 否 | 委托来源：00(普通)、01（闪兑） |
| entrustStrategy  | string | 否 | 委托策略：GTC（普通限价单）、MAKER（只做Maker）、IOC（订单若不能立即成交则未成交的部分立即取消）、FOK（订单若不能全部成交则立即全部取消） |
| entrustType  | string | 否 | 委单类型：B(买)、S(卖) |
| fee  | string | 否 | 手续费 |
| feeCoin  | string | 否 | 手续费币种 |
| leftAmount  | string | 否 | 左币金额 |
| leftAmountDeal  | string | 否 | 左币已成交 |
| leftCoin  | string | 否 | 左币 |
| mid  | int | 否 | 会员编号 |
| orderNo  | string | 否 | 订单号 |
| orderStatus  | string | 否 |委单状态:("00", "交易中"),("01", "交易完成"),("02", "交易失败"),("09", "推送撮合中"),("10", "撮合中"),("11", "部分撮合"),("20", "撤销中"),("21", "撤销完成") |
| rightAmount  | string | 否 | 右边成交金额 |
| rightAmountDeal  | string | 否 | 右币已处理金额 |
| rightCoin  | string | 否 | 右币 |
| symbol  | string | 否 | 交易对 |
| terminalCode  | string | 否 | 终端编号 |
| terminalType  | string | 否 | 终端类型：Android（安卓）、IOS（ios）、WEB（web 端）、API（api） |
| trades  | list | 否 | 成交列表 |

trades

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| buyOrderFullDeal  | boolean | 否 | 买单是否完全成交 |
| buyOrderNo  | string | 否 | 买单单号 |
| buyOrderPrice  | string | 否 | 买单下单价 |
| buyerFee  | string | 否 | 买方手续费 |
| buyerFeeCoin  | string | 否 | 买方手续费币种 |
| buyerId  | int | 否 | 卖方编号 |
| buyerMatchType  | string | 否 | 买方撮合类型:M（被撮合）、T（撮合） |
| dealPrice  | string | 否 | 成交价 |
| dealTimestamp  | string | 否 | 成交时间 |
| leftAmount  | string | 否 | 左币数量 |
| leftCoin  | string | 否 | 左币 |
| matchSerialNo  | string | 否 | 撮合流水号 |
| rightAmount  | string | 否 | 右币数量 |
| rightCoin  | string | 否 | 右币 |
| sellOrderFullDeal  | boolean | 否 | 卖单是否完全成交 |
| sellOrderNo  | string | 否 | 卖单单号 |
| sellOrderPrice  | string | 否 | 卖单下单价 |
| sellerFee  | string | 否 |卖方手续费 |
| sellerFeeCoin  | string | 否 |卖方手续费币种 |
| sellerId  | int | 否 | 卖方编号 |
| sellerMatchType  | string | 否 | 卖方撮合类型:M（被撮合）、T（撮合） |
| symbol  | string | 否 | 交易对 |
| tradeId  | string | 否 | 订单ID |


`
POST /v1/exchange/contract/order (HMAC SHA256)
`

发送下单。

**参数:**

```json
{
	"entrustCount": 1,
	"entrustPrice": "61620.7100",
	"triggerPrice": "61620.7100",
	"entrustType": "B",
	"entrustMode": "10",
	"contractCode": "btcusdt",
	"entrustStrategy": "GTC",
	"entrustSide": "O"
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| entrustCount  | int | 否 | 委托数量(张) |
| entrustType  | string | 否 | 委托类型:B（买）、S（卖） |
| contractCode  | string | 否 | 合约编号 |
| entrustMode  | string | 否 | 委单模式：10(限价)、11(止盈止损限价)、20(市价)、21(止盈止损市价) |
| entrustStrategy  | string | 是 | 委托策略：GTC（普通委单）、MAKER（只做Maker）、IOC（订单若不能立即成交则未成交的部分立即取消）、FOK（订单若不能全部成交则立即全部取消） |
| entrustPrice  | string | 是 | 委托价 |
| triggerPrice  | string | 是 | 触发价 |
| entrustSide  | string | 是 | 委托模式：O（开仓）、C（平仓） |

基于订单 entrustMode不同，强制要求某些参数:
|  类型   |强制要求的参数|
|  ----   |----  |
| 10    | entrustStrategy,entrustPrice |
| 11    | triggerPrice,entrustPrice |

其他信息:
- 当 entrustMode 为10 时，可设置entrustStrategy,其他类型只支持GTC
  

**数据源:** 数据库

**权重(IP):1**


### 撤销订单 

> **响应**

```json
{
	    "averagePrice": "62063",
		"contractCode": "btcw20211108",
		"createTime": 1636083578,
		"entrustMode": "10",
		"entrustPrice": "62063",
		"entrustSource": "1",
		"entrustType": "S",
		"fee": "0.00310315",
		"feeCoin": "usdt",
		"leftAmount": "2",
		"leftAmountDeal": "1",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5752824-70480",
		"orderStatus": "10",
		"rightAmount": "124.126",
		"rightAmountDeal": "62.063",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "her2b44e8e20-f755-11eb-b8c9-37c73197ff9d",
		"terminalType": "Web",
		"trades": [{
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-5752824-70471",
			"buyOrderPrice": "62063",
			"buyerFee": "0.00310315",
			"buyerFeeCoin": "usdt",
			"buyerId": 5712839,
			"buyerMatchType": "M",
			"contractCode": "btcw20211108",
			"dealPrice": "62063",
			"dealTimestamp": "1636083578",
			"leftAmount": "1",
			"leftCoin": "usdt",
			"matchSerialNo": "376571445669263",
			"rightAmount": "62.063",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-5752824-70480",
			"sellOrderPrice": "62063",
			"sellerFee": "0.00310315",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "T",
			"tradeId": "376565506876348"
		}]
}
```

` DELETE /v1/exchange/contract/entrust/{orderNo} (HMAC SHA256) `



取消有效订单。

**参数:**

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| orderNo  | string | 否 | 订单编号 |

发起撤单之后，订单会先处于撤单中，等待撮合中心处理后会转为撤单成功。

**权重(IP):1**

### 撤销单一合约的所有挂单 


`
POST /v1/exchange/contract/cancel/all
`

撤销单一交易对下所有挂单。

**参数:**

BODY
```json
{
	"symbol": "btcusdt"
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| symbol  | string | 否 | 合约编号 |

发起撤单之后，订单会先处于撤单中，等待撮合中心处理后会转为撤单成功。

**数据源:** 数据库

**权重(IP):1**

### 查询订单

> **响应**

```json
{
	    "averagePrice": "62063",
		"contractCode": "btcw20211108",
		"createTime": 1636083578,
		"entrustMode": "10",
		"entrustPrice": "62063",
		"entrustSource": "1",
		"entrustType": "S",
		"fee": "0.00310315",
		"feeCoin": "usdt",
		"leftAmount": "2",
		"leftAmountDeal": "1",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5752824-70480",
		"orderStatus": "10",
		"rightAmount": "124.126",
		"rightAmountDeal": "62.063",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "her2b44e8e20-f755-11eb-b8c9-37c73197ff9d",
		"terminalType": "Web",
		"trades": [{
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-5752824-70471",
			"buyOrderPrice": "62063",
			"buyerFee": "0.00310315",
			"buyerFeeCoin": "usdt",
			"buyerId": 5712839,
			"buyerMatchType": "M",
			"contractCode": "btcw20211108",
			"dealPrice": "62063",
			"dealTimestamp": "1636083578",
			"leftAmount": "1",
			"leftCoin": "usdt",
			"matchSerialNo": "376571445669263",
			"rightAmount": "62.063",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-5752824-70480",
			"sellOrderPrice": "62063",
			"sellerFee": "0.00310315",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "T",
			"tradeId": "376565506876348"
		}]
}
```

`
GET /v1/exchange/contract/entrust/{orderNo} (HMAC SHA256)
`

查询订单状态。

**参数:**

| 名称              | 类型   | 是否必需 | 描述                 |
| :---------------- | :----- | :------- | :------------------- |
| orderNo           | LONG   | YES       |                      |



**数据源:** 数据库

**权重(IP):2**

### 合约当前挂单

> **响应**


```json
{
	"list": [{
	    "averagePrice": "62063",
		"contractCode": "btcw20211108",
		"createTime": 1636083578,
		"entrustMode": "10",
		"entrustPrice": "62063",
		"entrustSource": "1",
		"entrustType": "S",
		"fee": "0.00310315",
		"feeCoin": "usdt",
		"leftAmount": "2",
		"leftAmountDeal": "1",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5752824-70480",
		"orderStatus": "10",
		"rightAmount": "124.126",
		"rightAmountDeal": "62.063",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "her2b44e8e20-f755-11eb-b8c9-37c73197ff9d",
		"terminalType": "Web",
		"trades": [{
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-5752824-70471",
			"buyOrderPrice": "62063",
			"buyerFee": "0.00310315",
			"buyerFeeCoin": "usdt",
			"buyerId": 5712839,
			"buyerMatchType": "M",
			"contractCode": "btcw20211108",
			"dealPrice": "62063",
			"dealTimestamp": "1636083578",
			"leftAmount": "1",
			"leftCoin": "usdt",
			"matchSerialNo": "376571445669263",
			"rightAmount": "62.063",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-5752824-70480",
			"sellOrderPrice": "62063",
			"sellerFee": "0.00310315",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "T",
			"tradeId": "376565506876348"
		}]
}],
	"total": "27"
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| total  | string | 否 | 总数 |
| list  | array | 否 | 记录列表 |

list

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| averagePrice  | string | 否 | 平均价 |
| createTime  | int | 否 | 创建时间 |
| contractCode  | string | 否 | 合约编号 |
| entrustMode  | string | 否 | 委单模式：10(限价)、11(止盈止损限价)、20(市价)、21(止盈止损市价) |
| entrustPrice  | string | 否 | 委单单价 |
| entrustSource  | string | 否 | 委托来源：00(普通)、01（闪兑） |
| entrustStrategy  | string | 否 | 委托策略：GTC（普通限价单）、MAKER（只做Maker）、IOC（订单若不能立即成交则未成交的部分立即取消）、FOK（订单若不能全部成交则立即全部取消） |
| entrustType  | string | 否 | 委单类型：B(买)、S(卖) |
| fee  | string | 否 | 手续费 |
| feeCoin  | string | 否 | 手续费币种 |
| leftAmount  | string | 否 | 左币金额 |
| leftAmountDeal  | string | 否 | 左币已成交 |
| leftCoin  | string | 否 | 左币 |
| mid  | int | 否 | 会员编号 |
| orderNo  | string | 否 | 订单号 |
| orderStatus  | string | 否 |委单状态:("00", "交易中"),("01", "交易完成"),("02", "交易失败"),("09", "推送撮合中"),("10", "撮合中"),("11", "部分撮合"),("20", "撤销中"),("21", "撤销完成") |
| rightAmount  | string | 否 | 右边成交金额 |
| rightAmountDeal  | string | 否 | 右币已处理金额 |
| rightCoin  | string | 否 | 右币 |
| symbol  | string | 否 | 交易对 |
| terminalCode  | string | 否 | 终端编号 |
| terminalType  | string | 否 | 终端类型：Android（安卓）、IOS（ios）、WEB（web 端）、API（api） |
| trades  | list | 否 | 成交列表 |

trades

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| buyOrderFullDeal  | boolean | 否 | 买单是否完全成交 |
| buyOrderNo  | string | 否 | 买单单号 |
| buyOrderPrice  | string | 否 | 买单下单价 |
| buyerFee  | string | 否 | 买方手续费 |
| buyerFeeCoin  | string | 否 | 买方手续费币种 |
| buyerId  | int | 否 | 卖方编号 |
| buyerMatchType  | string | 否 | 买方撮合类型:M（被撮合）、T（撮合） |
| dealPrice  | string | 否 | 成交价 |
| dealTimestamp  | string | 否 | 成交时间 |
| leftAmount  | string | 否 | 左币数量 |
| leftCoin  | string | 否 | 左币 |
| matchSerialNo  | string | 否 | 撮合流水号 |
| rightAmount  | string | 否 | 右币数量 |
| rightCoin  | string | 否 | 右币 |
| sellOrderFullDeal  | boolean | 否 | 卖单是否完全成交 |
| sellOrderNo  | string | 否 | 卖单单号 |
| sellOrderPrice  | string | 否 | 卖单下单价 |
| sellerFee  | string | 否 |卖方手续费 |
| sellerFeeCoin  | string | 否 |卖方手续费币种 |
| sellerId  | int | 否 | 卖方编号 |
| sellerMatchType  | string | 否 | 卖方撮合类型:M（被撮合）、T（撮合） |
| symbol  | string | 否 | 交易对 |
| tradeId  | string | 否 | 订单ID |

```
POST /v1/exchange/contract/entrust/p/{pageNum}-{pageSize} (HMAC SHA256)
```
**参数:**

| 名称       | 类型   | 是否必需 | 描述                 |
| :--------- | :----- | :------- | :------------------- |
| pageNum     | int | YES       |             页码：1开始          |
| pageSize | int   | YES       | 每页数量：最大100 |


BODY
```json
{
	"contractCode": "btcusdt",
	"startTime":1635955200,
	"endTime":1637510400,
	"entrustType":"B"
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| contractCode  | string | 是 | 合约编号 |
| startTime  | int | 是 | 开始时间 |
| endTime  | int | 是 | 结束时间 |
| entrustType  | string | 是 | 委单类型：B(买)、S(卖) |



- 不带symbol参数，会返回所有交易对的挂单


**权重(IP):** 3 单一交易对;
**40** 交易对参数缺失;

**数据源:** 数据库


### 合约成交历史 

> **响应**


```json
{
	"list": [{
	    "averagePrice": "62063",
		"contractCode": "btcw20211108",
		"createTime": 1636083578,
		"entrustMode": "10",
		"entrustPrice": "62063",
		"entrustSource": "1",
		"entrustType": "S",
		"fee": "0.00310315",
		"feeCoin": "usdt",
		"leftAmount": "2",
		"leftAmountDeal": "1",
		"leftCoin": "btc",
		"mid": 5712839,
		"orderNo": "376-5752824-70480",
		"orderStatus": "10",
		"rightAmount": "124.126",
		"rightAmountDeal": "62.063",
		"rightCoin": "usdt",
		"symbol": "btc_usdt",
		"terminalCode": "her2b44e8e20-f755-11eb-b8c9-37c73197ff9d",
		"terminalType": "Web",
		"trades": [{
			"buyOrderFullDeal": false,
			"buyOrderNo": "376-5752824-70471",
			"buyOrderPrice": "62063",
			"buyerFee": "0.00310315",
			"buyerFeeCoin": "usdt",
			"buyerId": 5712839,
			"buyerMatchType": "M",
			"contractCode": "btcw20211108",
			"dealPrice": "62063",
			"dealTimestamp": "1636083578",
			"leftAmount": "1",
			"leftCoin": "usdt",
			"matchSerialNo": "376571445669263",
			"rightAmount": "62.063",
			"rightCoin": "usdt",
			"sellOrderFullDeal": false,
			"sellOrderNo": "376-5752824-70480",
			"sellOrderPrice": "62063",
			"sellerFee": "0.00310315",
			"sellerFeeCoin": "usdt",
			"sellerId": 5712839,
			"sellerMatchType": "T",
			"tradeId": "376565506876348"
		}]
}],
	"total": "27"
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| total  | string | 否 | 总数 |
| list  | array | 否 | 记录列表 |

list

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| averagePrice  | string | 否 | 平均价 |
| createTime  | int | 否 | 创建时间 |
| contractCode  | string | 否 | 合约编号 |
| entrustMode  | string | 否 | 委单模式：10(限价)、11(止盈止损限价)、20(市价)、21(止盈止损市价) |
| entrustPrice  | string | 否 | 委单单价 |
| entrustSource  | string | 否 | 委托来源：00(普通)、01（闪兑） |
| entrustStrategy  | string | 否 | 委托策略：GTC（普通限价单）、MAKER（只做Maker）、IOC（订单若不能立即成交则未成交的部分立即取消）、FOK（订单若不能全部成交则立即全部取消） |
| entrustType  | string | 否 | 委单类型：B(买)、S(卖) |
| fee  | string | 否 | 手续费 |
| feeCoin  | string | 否 | 手续费币种 |
| leftAmount  | string | 否 | 左币金额 |
| leftAmountDeal  | string | 否 | 左币已成交 |
| leftCoin  | string | 否 | 左币 |
| mid  | int | 否 | 会员编号 |
| orderNo  | string | 否 | 订单号 |
| orderStatus  | string | 否 |委单状态:("00", "交易中"),("01", "交易完成"),("02", "交易失败"),("09", "推送撮合中"),("10", "撮合中"),("11", "部分撮合"),("20", "撤销中"),("21", "撤销完成") |
| rightAmount  | string | 否 | 右边成交金额 |
| rightAmountDeal  | string | 否 | 右币已处理金额 |
| rightCoin  | string | 否 | 右币 |
| symbol  | string | 否 | 交易对 |
| terminalCode  | string | 否 | 终端编号 |
| terminalType  | string | 否 | 终端类型：Android（安卓）、IOS（ios）、WEB（web 端）、API（api） |
| trades  | list | 否 | 成交列表 |

trades

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| buyOrderFullDeal  | boolean | 否 | 买单是否完全成交 |
| buyOrderNo  | string | 否 | 买单单号 |
| buyOrderPrice  | string | 否 | 买单下单价 |
| buyerFee  | string | 否 | 买方手续费 |
| buyerFeeCoin  | string | 否 | 买方手续费币种 |
| buyerId  | int | 否 | 卖方编号 |
| buyerMatchType  | string | 否 | 买方撮合类型:M（被撮合）、T（撮合） |
| dealPrice  | string | 否 | 成交价 |
| dealTimestamp  | string | 否 | 成交时间 |
| leftAmount  | string | 否 | 左币数量 |
| leftCoin  | string | 否 | 左币 |
| matchSerialNo  | string | 否 | 撮合流水号 |
| rightAmount  | string | 否 | 右币数量 |
| rightCoin  | string | 否 | 右币 |
| sellOrderFullDeal  | boolean | 否 | 卖单是否完全成交 |
| sellOrderNo  | string | 否 | 卖单单号 |
| sellOrderPrice  | string | 否 | 卖单下单价 |
| sellerFee  | string | 否 |卖方手续费 |
| sellerFeeCoin  | string | 否 |卖方手续费币种 |
| sellerId  | int | 否 | 卖方编号 |
| sellerMatchType  | string | 否 | 卖方撮合类型:M（被撮合）、T（撮合） |
| symbol  | string | 否 | 交易对 |
| tradeId  | string | 否 | 订单ID |

`
GET /v1/exchange/contract/entrust/detail/p/{pageNum}-{pageSize} (HMAC SHA256)
`

获取指定合约的成交历史

**参数:**

| 名称       | 类型   | 是否必需 | 描述                 |
| :--------- | :----- | :------- | :------------------- |
| pageNum     | int | YES       |             页码：1开始          |
| pageSize | int   | YES       | 每页数量：最大100 |

BODY
```json
{
	"contractCode": "btcusdt",
	"startTime":1635955200,
	"endTime":1637510400
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| contractCode  | string | 是 | 合约编号 |
| startTime  | int | 是 | 开始时间 |
| endTime  | int | 是 | 结束时间 |
| entrustType  | string | 是 | 委单类型：B(买)、S(卖) |

**数据源:** 数据库

**权重(IP):10**


### 合约仓位信息 

> **响应**


```json
{
	"data": {
		"eosusdt:long": {
			"amount": "5",
			"bankruptcyPrice": "0",
			"baseCurrency": "eos",
			"contractCode": "eosusdt",
			"contractName": "eos\/usdt 永续",
			"count": 0,
			"entrustAmount": "5",
			"entryPrice": "0",
			"freeze": 0,
			"initMargin": "0",
			"leverage": "50",
			"liquidationPrice": "0",
			"maintMargin": "0.01",
			"margin": "0",
			"mid": 5712839,
			"positionAmount": "0",
			"positionMode": "0",
			"positionNo": "376-5492668-60751",
			"positionType": "long",
			"priorityRanking": "0",
			"quoteCurrency": "usdt",
			"rewardPercent": "0",
			"riskLimit": "600000000",
			"riskStep": "300000000",
			"settlementCurrency": "usdt",
			"unrealised": "0",
			"userMargin": "0",
			"marginRate": "0.01",
			"initMarginRate":"0.5",
			"maintMarginRate":"0.5"
		},
		"btcw20211108:long": {
			"amount": "60.699",
			"bankruptcyPrice": "0",
			"baseCurrency": "btc",
			"contractCode": "btcw20211108",
			"contractName": "btc1108",
			"count": 0,
			"entrustAmount": "60.699",
			"entryPrice": "0",
			"freeze": 0,
			"initMargin": "0",
			"leverage": "100",
			"liquidationPrice": "0",
			"maintMargin": "0.005",
			"margin": "0",
			"mid": 5712839,
			"positionAmount": "0",
			"positionMode": "0",
			"positionNo": "375-1751956-20048",
			"positionType": "long",
			"priorityRanking": "0",
			"quoteCurrency": "usdt",
			"rewardPercent": "0",
			"riskLimit": "600000000",
			"riskStep": "300000000",
			"settlementCurrency": "usdt",
			"unrealised": "0",
			"userMargin": "0",
			"marginRate": "0.01",
			"initMarginRate":"0.5",
			"maintMarginRate":"0.5"
		},
		"btcm20221226:long": {
			"amount": "60.86",
			"bankruptcyPrice": "0",
			"baseCurrency": "btc",
			"contractCode": "btcm20221226",
			"contractName": "btc1226",
			"count": 0,
			"entrustAmount": "60.86",
			"entryPrice": "0",
			"freeze": 0,
			"initMargin": "0",
			"leverage": "100",
			"liquidationPrice": "0",
			"maintMargin": "0.005",
			"margin": "0",
			"mid": 5712839,
			"positionAmount": "0",
			"positionMode": "0",
			"positionNo": "375-1743399-80032",
			"positionType": "long",
			"priorityRanking": "0",
			"quoteCurrency": "usdt",
			"rewardPercent": "0",
			"riskLimit": "600000000",
			"riskStep": "300000000",
			"settlementCurrency": "usdt",
			"unrealised": "0",
			"userMargin": "0",
			"marginRate": "0.01",
			"initMarginRate":"0.5",
			"maintMarginRate":"0.5"
		}
	}
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| data  | map | 否 |持仓数据 |

data

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| contactCode:positionType  | object | 否 |合约编号：仓位类型（short(空头)、long(多头)） |

contactCode:positionType

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| amount  | String | 否 |持仓总价值（包含委托价值） |
| bankruptcyPrice  | String | 否 |破产价 |
| baseCurrency  | String | 否 |基础币种 |
| contractCode  | String | 否 |合约编号 |
| contractName  | String | 否 |合约名称 |
| count  | int | 否 |持仓数量 |
| entrustAmount  | String | 否 |委托价值 |
| entryPrice  | String | 否 |开仓价 |
| freeze  | int | 否 |持仓冻结数量 |
| initMargin  | String | 否 |起始保证金 |
| leverage  | String | 否 |杠杆 |
| liquidationPrice  | String | 否 |强平价 |
| maintMargin  | String | 否 |维持保证金 |
| margin  | String | 否 |用户追加保证金 |
| mid  | int | 否 |用户编号 |
| positionAmount  | String | 否 |持仓价值 |
| positionMode  | String | 否 |持仓类型:0(全仓)、1(逐仓) |
| positionNo  | String | 否 |仓位编号 |
| positionType  | String | 否 |持仓方向：short(空头)、long(多头) |
| priorityRanking  | String | 否 |自动减仓排序 |
| quoteCurrency  | String | 否 |计价币种 |
| rewardPercent  | String | 否 |盈亏百分比 |
| riskLimit  | String | 否 |风险限额 |
| riskStep  | String | 否 |风险限额递增值 |
| settlementCurrency  | String | 否 |结算币种 |
| unrealised  | String | 否 |未实现盈亏 |
| userMargin  | String | 否 |用户当前仓位总保证金 |
| marginRate  | String | 否 |保证金率 |
| initMarginRate  | String | 否 |基础保证金率 |
| maintMarginRate  | String | 否 |维持保证金率 |

`
GET /v1/exchange/contract/position (HMAC SHA256)
`

获取合约的仓位信息

**数据源:** 数据库

**权重(IP):10**

## 错误代码列表

| 错误码       | 描述                 |
| :--------- | :----- |
| 1031     | 会员账户余额不足      |
