#  做市商行情推送文档

**版本**
`1.0.0`

[toc]

* * *

## 行情推送基本信息

- 行情推送依赖于mqtt 协议，接入时需先安装mqtt依赖，各个语言的依赖下载地址
https://github.com/mqtt/mqtt.org/wiki/libraries
https://docs.emqx.cn/enterprise/v4.3/development/client.html#mqtt-%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BA%93

- 连接地址
  TCP 地址 mqtt.hotx.pro:1883
  Websocket 地址 ws://mqtt.hotx.pro:8083/mqtt
  
- 连接参数
clientId:her5_{随机字符串：32位以上。相同clientId重复发起连接时，会拒绝当前请求}
username: 用户api key_时间戳(秒单位)
password: 用户签名(hmacSha256(clientId+timestamp,API-Secret))


- 订阅主题名称中所有交易对均为 **小写**


### SIGNED  Endpoint security

- 签名使用`HMAC SHA256`算法. API-KEY所对应的API-Secret作为 `HMAC SHA256` 的密钥，其他所有参数作为`HMAC SHA256`的操作对象，得到的输出即为签名。
- `签名` **大小写不敏感**.

#### 示例

以下是在linux bash环境下使用 echo openssl  实现的参数初始化的示例 apikey、secret仅供示范

| Key       | Value                                                        |
| :-------- | :----------------------------------------------------------- |
| apiKey    | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh12 |
| secretKey | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj1j |


> **Example **
>
> **HMAC SHA256 signature:**

```
    $ echo -n "her5_b44e8e20-f755-11eb-b8c9-37c73197ff9d1499827319" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj1j"
    (stdin)= 7adb0dd77ba4e13da2d04c48d0976cf0c95c1425e7cbecc8e562821fc2caeafb
```

- **conect options:**

```json
{
	"clientId": "her5_b44e8e20-f755-11eb-b8c9-37c73197ff9d",
	"username": "vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh12_1499827319",
	"password": "7adb0dd77ba4e13da2d04c48d0976cf0c95c1425e7cbecc8e562821fc2caeafb"
}
```

### 访问限制

#### MQTT 连接限制

- 每个用户只能连接一个MQTT 连接，其他连接将会被拒绝。

## 公共Topic

- `客户端主动订阅以下的主题，服务端向该主题推送消息`

### 现货行情

topic : `exchange.public.spot.ticker`

```json
[{
		"changeRange": "0",
		"close": "0",
		"count": 0,
		"exchange": "HER",
		"high": "0",
		"last": "0",
		"low": "0",
		"open": "0",
		"rightVolume": "0",
		"symbol": "eos_usdt",
		"timestamp": "0",
		"volume": "0"
	}, {
		"changeRange": "0.00626221",
		"close": "4794.93",
		"count": 18916,
		"high": "4821.21",
		"last": "4794.93",
		"low": "4731.9", 
		"open": "4765.09",
		"rightVolume": "480647428.709",
		"symbol": "",
		"timestamp": "0",
		"volume": "1005982"
	}, {
		"changeRange": "-0.00925699",
		"close": "52.657",
		"contractCode": "dotusdt",
		"contractType": "6",
		"count": 15739,
		"high": "53.809",
		"last": "52.657",
		"low": "52.643",
		"open": "53.149",
		"rightVolume": "44240479.453",
		"symbol": "",
		"timestamp": "0",
		"volume": "830411"
	}, {
		"changeRange": "0",
		"close": "0",
		"count": 0,
		"exchange": "HER",
		"high": "0",
		"last": "0",
		"low": "0",
		"open": "0",
		"rightVolume": "0",
		"symbol": "btc_usdt",
		"timestamp": "0",
		"volume": "0"
	}, {
		"changeRange": "0.01664824",
		"close": "67477.78",
		"count": 15959,
		"high": "67742.63",
		"last": "67477.78",
		"low": "65589.55",
		"open": "66372.79",
		"rightVolume": "55569181.823608",
		"symbol": "",
		"timestamp": "0",
		"volume": "836650"
	}, {
		"changeRange": "0",
		"close": "0",
		"count": 0,
		"exchange": "HER",
		"high": "0",
		"last": "0",
		"low": "0",
		"open": "0",
		"rightVolume": "0",
		"symbol": "btc_usdt",
		"timestamp": "0",
		"volume": "0"
	}
]
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| changeRange | string | 否 |涨跌幅 |
| close | string | 否 |收盘价 |
| count | int | 否 |成交交易数 |
| high | string | 否 |最高成交价 |
| last | string | 否 |最新价 |
| low | string | 否 |最低成交价 |
| open | string | 否 |开盘价 |
| rightVolume | string | 否 |成交额 |
| symbol | string | 否 |交易对 |
| timestamp | string | 否 |数据时间戳 |
| volume | string | 否 |成交量 |


### 合约行情

topic : `exchange.public.contract.ticker`

```json
[{
		"changeRange": "0",
		"close": "0",
		"contractCode": "eosusdt",
		"contractType": "6",
		"count": 0,
		"exchange": "HER",
		"high": "0",
		"last": "0",
		"low": "0",
		"open": "0",
		"rightVolume": "0",
		"symbol": "eos_usdt",
		"timestamp": "0",
		"volume": "0"
	}, {
		"changeRange": "0.00626221",
		"close": "4794.93",
		"contractCode": "ethusdt",
		"contractType": "6",
		"count": 18916,
		"high": "4821.21",
		"last": "4794.93",
		"low": "4731.9",
		"open": "4765.09",
		"rightVolume": "480647428.709",
		"symbol": "",
		"timestamp": "0",
		"volume": "1005982"
	}, {
		"changeRange": "-0.00925699",
		"close": "52.657",
		"contractCode": "dotusdt",
		"contractType": "6",
		"count": 15739,
		"high": "53.809",
		"last": "52.657",
		"low": "52.643",
		"open": "53.149",
		"rightVolume": "44240479.453",
		"symbol": "",
		"timestamp": "0",
		"volume": "830411"
	}, {
		"changeRange": "0",
		"close": "60799.57",
		"contractCode": "btcw20211108",
		"contractType": "1",
		"count": 0,
		"exchange": "HER",
		"high": "60799.57",
		"last": "60799.57",
		"low": "60799.57",
		"open": "60799.57",
		"rightVolume": "0",
		"symbol": "btc_usdt",
		"timestamp": "0",
		"volume": "0"
	}, {
		"changeRange": "0",
		"close": "0",
		"contractCode": "btcw20211115",
		"contractType": "1",
		"count": 0,
		"exchange": "HER",
		"high": "0",
		"last": "0",
		"low": "0",
		"open": "0",
		"rightVolume": "0",
		"symbol": "btc_usdt",
		"timestamp": "0",
		"volume": "0"
	}, {
		"changeRange": "0.01664824",
		"close": "67477.78",
		"contractCode": "btcusdt",
		"contractType": "6",
		"count": 15959,
		"high": "67742.63",
		"last": "67477.78",
		"low": "65589.55",
		"open": "66372.79",
		"rightVolume": "55569181.823608",
		"symbol": "",
		"timestamp": "0",
		"volume": "836650"
	}, {
		"changeRange": "0",
		"close": "0",
		"contractCode": "btcm20221226",
		"contractType": "2",
		"count": 0,
		"exchange": "HER",
		"high": "0",
		"last": "0",
		"low": "0",
		"open": "0",
		"rightVolume": "0",
		"symbol": "btc_usdt",
		"timestamp": "0",
		"volume": "0"
	}
]
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| changeRange | string | 否 |涨跌幅 |
| close | string | 否 |收盘价 |
| count | int | 否 |成交交易数 |
| high | string | 否 |最高成交价 |
| last | string | 否 |最新价 |
| low | string | 否 |最低成交价 |
| open | string | 否 |开盘价 |
| rightVolume | string | 否 |成交额 |
| contractType | string | 否 |合约类型:1(周合约)、2（月合约）、3(季度合约)、4(半年合约)、5(双周合约)、6(永续合约) |
| contractCode | string | 否 |合约编号 |
| timestamp | string | 否 |数据时间戳 |
| volume | string | 否 |成交量 |


### 合约最新仓位信息

topic : `exchange.public.contract.position`

```json
{
	"dotusdt": "149322",
	"btcusdt": "3634",
	"ethusdt": "1271"
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| {contractCode} | string | 否 |合约的仓位总和 |

### 现货最新成交

topic : `exchange.public.spot.trade.{symbol}.newest`

symbol :`交易对编号`

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

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| dealAmount | string | 否 |成交量 |
| dealPrice | string | 否 |成交价 |
| dealSide | string | 否 |成交方向：B（买）、S（卖） |
| dealTimestamp | string | 否 |成交时间 |
| symbol | string | 否 |交易对 |

### 合约最新成交

topic : `exchange.public.spot.trade.{contractCode}.newest`

contractCode :`合约编号`

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

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| dealAmount | string | 否 |成交量 |
| dealPrice | string | 否 |成交价 |
| dealSide | string | 否 |成交方向：B（买）、S（卖） |
| dealTimestamp | string | 否 |成交时间 |
| contractCode | string | 否 |合约编号 |


### 现货最新K线

topic : `exchange.public.spot.kline.{symbol}.{klineType}{val}`

symbol :`交易对编号`

klineType :`k线类型:minute(分钟)、hour(小时)、day(日)、week(周)、month(月)`

val :`k线单位`

```json
["1636428360", "67569.77", "67576.6", "67569.77", "67576.6", "25", "11.00", 10]
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


### 合约最新K线

topic : `exchange.public.contract.kline.{contractCode}.{klineType}{val}`

contractCode :`合约编号`

klineType :`k线类型:minute(分钟)、hour(小时)、day(日)、week(周)、month(月)`

val :`k线单位`

```json
["1636428360", "67569.77", "67576.6", "67569.77", "67576.6", "25", "11.00", 10]
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

### 现货最新深度

topic : `exchange.public.spot.depth.{symbol}.{depth}`

symbol :`交易对编号`

depth :`深度`

```json
{
	"buy": [{
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67653.29",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67629.75",
		"total": "0"
	}, {
		"amount": "8",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67624.8",
		"total": "0"
	}, {
		"amount": "6",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67608.11",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67578.56",
		"total": "0"
	}, {
		"amount": "5",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67544.37",
		"total": "0"
	}, {
		"amount": "3",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67541.78",
		"total": "0"
	}, {
		"amount": "6",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67511.57",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67465.21",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67433.27",
		"total": "0"
	}, {
		"amount": "4",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67430.5",
		"total": "0"
	}, {
		"amount": "2",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67424.2",
		"total": "0"
	}, {
		"amount": "7",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67418.84",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67408.23",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67399.34",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67384.97",
		"total": "0"
	}, {
		"amount": "2",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67286.73",
		"total": "0"
	}, {
		"amount": "7",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67282.58",
		"total": "0"
	}],
	"exchange": "HER",
	"sell": [{
		"amount": "9",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67666.53",
		"total": "0"
	}, {
		"amount": "5",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67673.3",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67673.31",
		"total": "0"
	}, {
		"amount": "4",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67691.86",
		"total": "0"
	}, {
		"amount": "4",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67701.65",
		"total": "0"
	}, {
		"amount": "5",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67715.51",
		"total": "0"
	}, {
		"amount": "3",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67786.92",
		"total": "0"
	}, {
		"amount": "4",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67787.97",
		"total": "0"
	}, {
		"amount": "7",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67796.27",
		"total": "0"
	}],
	"symbol": "btc_usdt",
	"timestamp": "1636429063333",
	"tradeMarket": "spot"
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| symbol | string | 否 |交易对 |
| timestamp | string | 否 |时间戳 |
| buy | array | 否 |买盘 |
| sell | array | 否 |卖盘 |

buy||sell

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| amount | string | 否 |数量 |
| entrustType | string | 否 |委托类型:B（买）、S（卖） |
| price | string | 否 |价格 |
| total | string | 否 |成交额 |

### 合约最新深度

topic : `exchange.public.contract.depth.{contractCode}.{depth}`

contractCode :`合约编号`

depth :`深度`

```json
{
	"buy": [{
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67653.29",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67629.75",
		"total": "0"
	}, {
		"amount": "8",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67624.8",
		"total": "0"
	}, {
		"amount": "6",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67608.11",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67578.56",
		"total": "0"
	}, {
		"amount": "5",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67544.37",
		"total": "0"
	}, {
		"amount": "3",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67541.78",
		"total": "0"
	}, {
		"amount": "6",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67511.57",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67465.21",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67433.27",
		"total": "0"
	}, {
		"amount": "4",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67430.5",
		"total": "0"
	}, {
		"amount": "2",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67424.2",
		"total": "0"
	}, {
		"amount": "7",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67418.84",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67408.23",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67399.34",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67384.97",
		"total": "0"
	}, {
		"amount": "2",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67286.73",
		"total": "0"
	}, {
		"amount": "7",
		"entrustType": "B",
		"legalPrice": "0",
		"price": "67282.58",
		"total": "0"
	}],
	"exchange": "HER",
	"sell": [{
		"amount": "9",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67666.53",
		"total": "0"
	}, {
		"amount": "5",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67673.3",
		"total": "0"
	}, {
		"amount": "1",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67673.31",
		"total": "0"
	}, {
		"amount": "4",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67691.86",
		"total": "0"
	}, {
		"amount": "4",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67701.65",
		"total": "0"
	}, {
		"amount": "5",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67715.51",
		"total": "0"
	}, {
		"amount": "3",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67786.92",
		"total": "0"
	}, {
		"amount": "4",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67787.97",
		"total": "0"
	}, {
		"amount": "7",
		"entrustType": "S",
		"legalPrice": "0",
		"price": "67796.27",
		"total": "0"
	}],
	"contractCode": "btcusdt",
	"timestamp": "1636429063333"
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| contractCode | string | 否 |合约编号 |
| timestamp | string | 否 |时间戳 |
| buy | array | 否 |买盘 |
| sell | array | 否 |卖盘 |

buy||sell
|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| amount | string | 否 |数量 |
| entrustType | string | 否 |委托类型:B（买）、S（卖） |
| price | string | 否 |价格 |
| total | string | 否 |成交额 |


### 合约合理价格标记

topic : `exchange.public.fair.price`

```json
{
	"eosusdt": {
		"last": "5.1851"
	},
	"ethusdt": {
		"last": "4805"
	},
	"dotusdt": {
		"last": "53.1257"
	},
	"btcw20211108": {
		"last": "66220.88"
	},
	"btcw20211115": {
		"last": "68197.185"
	},
	"btcusdt": {
		"last": "68191.749"
	},
	"btcm20221226": {
		"last": "68197.185"
	}
}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| {key} | object | 否 |合约编号 |

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| last | string | 否 |最新合理价格标记 |

### 合约指数价格

topic : `exchange.public.index.price`

```json
{
	"her_btc_bon_2h": {
		"createTime": 1636423200,
		"exchange": "HER",
		"id": "377999727654452",
		"indexSymbol": "her_btc_bon_2h",
		"price": "0.0003",
		"symbol": "btc",
		"tickerTime": 1636423200,
		"timestamp": "1636423200000"
	},
	"her_btc_usdt_fr_8h": {
		"createTime": 1636401600,
		"exchange": "HER",
		"id": "377909407295704",
		"indexSymbol": "her_btc_usdt_fr_8h",
		"price": "-0.00008185",
		"symbol": "btc_usdt",
		"tickerTime": 1636401600,
		"timestamp": "1636401600000",
		"weight": "0"
	},
	"her_eos_usdt_pi": {
		"exchange": "HER",
		"indexSymbol": "HER_eos_usdt_pi",
		"price": "-0.03658395",
		"symbol": "eos_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000"
	},
	"her_btc_usdt_pi_2h": {
		"createTime": 1636423200,
		"exchange": "HER",
		"id": "377999727654461",
		"indexSymbol": "her_btc_usdt_pi_2h",
		"price": "-0.00006915",
		"symbol": "btc_usdt",
		"tickerTime": 1636423200,
		"timestamp": "1636423200000",
		"weight": "0"
	},
	"her_eth_usdt_fr": {
		"exchange": "HER",
		"indexSymbol": "HER_eth_usdt_fr",
		"price": "-0.0001",
		"symbol": "eth_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000"
	},
	"huobi_dot_usdt": {
		"createTime": 1636429763,
		"exchange": "HuoBi",
		"id": "378027452069647",
		"indexSymbol": "HuoBi_dot_usdt",
		"price": "53.1846",
		"symbol": "dot_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000",
		"weight": "0.5"
	},
	"her_eth_usdt_pi": {
		"exchange": "HER",
		"indexSymbol": "HER_eth_usdt_pi",
		"price": "0.00022939",
		"symbol": "eth_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000"
	},
	"her_eos_usdt_30m": {
		"createTime": 1636428600,
		"exchange": "HER",
		"id": "378022343395730",
		"indexSymbol": "her_eos_usdt_30m",
		"price": "5.0941",
		"symbol": "eos_usdt",
		"tickerTime": 1636428600,
		"timestamp": "1636428600000"
	},
	"her_eos_usdt_fr": {
		"exchange": "HER",
		"indexSymbol": "HER_eos_usdt_fr",
		"price": "0",
		"symbol": "eos_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000"
	},
	"huobi_eth_usdt": {
		"createTime": 1636429763,
		"exchange": "HuoBi",
		"id": "378027452069649",
		"indexSymbol": "HuoBi_eth_usdt",
		"price": "4805.02",
		"symbol": "eth_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000",
		"weight": "0.5"
	},
	"her_eth_usdt_30m": {
		"createTime": 1636428600,
		"exchange": "HER",
		"id": "378022343395726",
		"indexSymbol": "her_eth_usdt_30m",
		"price": "4791.55",
		"symbol": "eth_usdt",
		"tickerTime": 1636428600,
		"timestamp": "1636428600000"
	},
	"her_btc_usdt_pi": {
		"exchange": "HER",
		"indexSymbol": "HER_btc_usdt_pi",
		"price": "-0.00077896",
		"symbol": "btc_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000"
	},
	"her_eos_bon_8h": {
		"createTime": 1636401600,
		"exchange": "HER",
		"id": "377909407295692",
		"indexSymbol": "her_eos_bon_8h",
		"price": "0.0003",
		"symbol": "eos",
		"tickerTime": 1636401600,
		"timestamp": "1636401600000"
	},
	"her_btc_bon": {
		"createTime": 1636429740,
		"exchange": "HER",
		"id": "378027452069397",
		"indexSymbol": "her_btc_bon",
		"price": "0.0003",
		"symbol": "btc",
		"tickerTime": 1636429740,
		"timestamp": "1636429740000"
	},
	"her_eth_usdt_1s": {
		"exchange": "HER",
		"indexSymbol": "her_eth_usdt_1s",
		"price": "4806.565",
		"symbol": "eth_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000"
	},
	"zb_btc_usdt": {
		"createTime": 1636429764,
		"exchange": "ZB",
		"id": "378027452069650",
		"indexSymbol": "zb_btc_usdt",
		"price": "68249.9",
		"symbol": "btc_usdt",
		"tickerTime": 1636429764,
		"timestamp": "1636429764000",
		"weight": "0.5"
	},
	"zb_eos_usdt": {
		"createTime": 1636429763,
		"exchange": "ZB",
		"id": "378027452069648",
		"indexSymbol": "zb_eos_usdt",
		"price": "5.1494",
		"symbol": "eos_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000",
		"weight": "0.5"
	},
	"her_dot_usdt": {
		"createTime": 1636429740,
		"exchange": "HER",
		"id": "378027452069427",
		"indexSymbol": "her_dot_usdt",
		"price": "53.1783",
		"symbol": "dot_usdt",
		"tickerTime": 1636429740,
		"timestamp": "1636429740000"
	},
	"her_btc_usdt_pi_8h": {
		"createTime": 1636401600,
		"exchange": "HER",
		"id": "377909407295694",
		"indexSymbol": "her_btc_usdt_pi_8h",
		"price": "0.00000049",
		"symbol": "btc_usdt",
		"tickerTime": 1636401600,
		"timestamp": "1636401600000"
	},
	"her_dot_usdt_30m": {
		"createTime": 1636428600,
		"exchange": "HER",
		"id": "378022343395728",
		"indexSymbol": "her_dot_usdt_30m",
		"price": "52.93477333",
		"symbol": "dot_usdt",
		"tickerTime": 1636428600,
		"timestamp": "1636428600000"
	},
	"zb_dot_usdt": {
		"createTime": 1636429764,
		"exchange": "ZB",
		"id": "378027452069653",
		"indexSymbol": "zb_dot_usdt",
		"price": "0",
		"symbol": "dot_usdt",
		"tickerTime": 1636429764,
		"timestamp": "1636429764000",
		"weight": "0"
	},
	"her_dot_usdt_fr_8h": {
		"createTime": 1636401600,
		"exchange": "HER",
		"id": "377909407295702",
		"indexSymbol": "her_dot_usdt_fr_8h",
		"price": "0.00001256",
		"symbol": "dot_usdt",
		"tickerTime": 1636401600,
		"timestamp": "1636401600000"
	},
	"her_btc_usdt_1s": {
		"exchange": "HER",
		"indexSymbol": "her_btc_usdt_1s",
		"price": "68257.835",
		"symbol": "btc_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000"
	},
	"her_eos_usdt_pi_2h": {
		"createTime": 1636423200,
		"exchange": "HER",
		"id": "377999727654450",
		"indexSymbol": "her_eos_usdt_pi_2h",
		"price": "0.0164091",
		"symbol": "eos_usdt",
		"tickerTime": 1636423200,
		"timestamp": "1636423200000"
	},
	"her_eth_bon_8h": {
		"createTime": 1636401600,
		"exchange": "HER",
		"id": "377909407295698",
		"indexSymbol": "her_eth_bon_8h",
		"price": "0.0003",
		"symbol": "eth",
		"tickerTime": 1636401600,
		"timestamp": "1636401600000"
	},
	"her_eth_usdt": {
		"createTime": 1636429740,
		"exchange": "HER",
		"id": "378027452069430",
		"indexSymbol": "her_eth_usdt",
		"price": "4808.99",
		"symbol": "eth_usdt",
		"tickerTime": 1636429740,
		"timestamp": "1636429740000"
	},
	"her_eth_usdt_pi_2h": {
		"createTime": 1636423200,
		"exchange": "HER",
		"id": "377999727654457",
		"indexSymbol": "her_eth_usdt_pi_2h",
		"price": "-0.00025374",
		"symbol": "eth_usdt",
		"tickerTime": 1636423200,
		"timestamp": "1636423200000",
		"weight": "0"
	},
	"her_dot_usdt_pi_8h": {
		"createTime": 1636401600,
		"exchange": "HER",
		"id": "377909407295700",
		"indexSymbol": "her_dot_usdt_pi_8h",
		"price": "0.00009565",
		"symbol": "dot_usdt",
		"tickerTime": 1636401600,
		"timestamp": "1636401600000"
	},
	"her_dot_usdt_fr": {
		"exchange": "HER",
		"indexSymbol": "HER_dot_usdt_fr",
		"price": "-0.00100429",
		"symbol": "dot_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000"
	},
	"her_usdt_bon": {
		"createTime": 1636429740,
		"exchange": "HER",
		"id": "378027452069398",
		"indexSymbol": "her_usdt_bon",
		"price": "0.0006",
		"symbol": "usdt",
		"tickerTime": 1636429740,
		"timestamp": "1636429740000"
	},
	"her_eth_usdt_pi_8h": {
		"createTime": 1636401600,
		"exchange": "HER",
		"id": "377909407295699",
		"indexSymbol": "her_eth_usdt_pi_8h",
		"price": "-0.00019956",
		"symbol": "eth_usdt",
		"tickerTime": 1636401600,
		"timestamp": "1636401600000",
		"weight": "0"
	},
	"huobi_btc_usdt": {
		"createTime": 1636429763,
		"exchange": "HuoBi",
		"id": "378027452069646",
		"indexSymbol": "HuoBi_btc_usdt",
		"price": "68265.77",
		"symbol": "btc_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000",
		"weight": "0.5"
	},
	"huobi_eos_usdt": {
		"createTime": 1636429764,
		"exchange": "HuoBi",
		"id": "378027452069651",
		"indexSymbol": "HuoBi_eos_usdt",
		"price": "5.1502",
		"symbol": "eos_usdt",
		"tickerTime": 1636429764,
		"timestamp": "1636429764000",
		"weight": "0.5"
	},
	"her_eos_usdt_1s": {
		"exchange": "HER",
		"indexSymbol": "her_eos_usdt_1s",
		"price": "5.1498",
		"symbol": "eos_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000"
	},
	"her_eos_usdt_fr_8h": {
		"createTime": 1636401600,
		"exchange": "HER",
		"id": "377909407295703",
		"indexSymbol": "her_eos_usdt_fr_8h",
		"price": "0.0075",
		"symbol": "eos_usdt",
		"tickerTime": 1636401600,
		"timestamp": "1636401600000"
	},
	"her_dot_usdt_pi_2h": {
		"createTime": 1636423200,
		"exchange": "HER",
		"id": "377999727654455",
		"indexSymbol": "her_dot_usdt_pi_2h",
		"price": "0.00022078",
		"symbol": "dot_usdt",
		"tickerTime": 1636423200,
		"timestamp": "1636423200000"
	},
	"her_btc_usdt_fr": {
		"exchange": "HER",
		"indexSymbol": "HER_btc_usdt_fr",
		"price": "-0.00027896",
		"symbol": "btc_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000"
	},
	"her_eth_usdt_fr_8h": {
		"createTime": 1636401600,
		"exchange": "HER",
		"id": "377909407295695",
		"indexSymbol": "her_eth_usdt_fr_8h",
		"price": "-0.00014666",
		"symbol": "eth_usdt",
		"tickerTime": 1636401600,
		"timestamp": "1636401600000",
		"weight": "0"
	},
	"her_dot_bon": {
		"createTime": 1636429740,
		"exchange": "HER",
		"id": "378027452069401",
		"indexSymbol": "her_dot_bon",
		"price": "0.0003",
		"symbol": "dot",
		"tickerTime": 1636429740,
		"timestamp": "1636429740000"
	},
	"her_dot_usdt_1s": {
		"exchange": "HER",
		"indexSymbol": "her_dot_usdt_1s",
		"price": "53.1846",
		"symbol": "dot_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000"
	},
	"her_btc_usdt_30m": {
		"createTime": 1636428600,
		"exchange": "HER",
		"id": "378022343395729",
		"indexSymbol": "her_btc_usdt_30m",
		"price": "67613.75633333",
		"symbol": "btc_usdt",
		"tickerTime": 1636428600,
		"timestamp": "1636428600000"
	},
	"her_eth_bon": {
		"createTime": 1636429740,
		"exchange": "HER",
		"id": "378027452069399",
		"indexSymbol": "her_eth_bon",
		"price": "0.0003",
		"symbol": "eth",
		"tickerTime": 1636429740,
		"timestamp": "1636429740000"
	},
	"her_eth_bon_2h": {
		"createTime": 1636423200,
		"exchange": "HER",
		"id": "377999727654454",
		"indexSymbol": "her_eth_bon_2h",
		"price": "0.0003",
		"symbol": "eth",
		"tickerTime": 1636423200,
		"timestamp": "1636423200000"
	},
	"her_dot_bon_8h": {
		"createTime": 1636401600,
		"exchange": "HER",
		"id": "377909407295697",
		"indexSymbol": "her_dot_bon_8h",
		"price": "0.0003",
		"symbol": "dot",
		"tickerTime": 1636401600,
		"timestamp": "1636401600000"
	},
	"her_btc_usdt": {
		"createTime": 1636429740,
		"exchange": "HER",
		"id": "378027452069432",
		"indexSymbol": "her_btc_usdt",
		"price": "68303.81",
		"symbol": "btc_usdt",
		"tickerTime": 1636429740,
		"timestamp": "1636429740000"
	},
	"her_btc_bon_8h": {
		"createTime": 1636401600,
		"exchange": "HER",
		"id": "377909407295696",
		"indexSymbol": "her_btc_bon_8h",
		"price": "0.0003",
		"symbol": "btc",
		"tickerTime": 1636401600,
		"timestamp": "1636401600000"
	},
	"zb_eth_usdt": {
		"createTime": 1636429764,
		"exchange": "ZB",
		"id": "378027452069652",
		"indexSymbol": "zb_eth_usdt",
		"price": "4808.11",
		"symbol": "eth_usdt",
		"tickerTime": 1636429764,
		"timestamp": "1636429764000",
		"weight": "0.5"
	},
	"her_eos_usdt": {
		"createTime": 1636429740,
		"exchange": "HER",
		"id": "378027452069433",
		"indexSymbol": "her_eos_usdt",
		"price": "5.1559",
		"symbol": "eos_usdt",
		"tickerTime": 1636429740,
		"timestamp": "1636429740000"
	},
	"her_eos_usdt_pi_8h": {
		"createTime": 1636401600,
		"exchange": "HER",
		"id": "377909407295701",
		"indexSymbol": "her_eos_usdt_pi_8h",
		"price": "0.05906243",
		"symbol": "eos_usdt",
		"tickerTime": 1636401600,
		"timestamp": "1636401600000"
	},
	"her_eos_bon_2h": {
		"createTime": 1636423200,
		"exchange": "HER",
		"id": "377999727654451",
		"indexSymbol": "her_eos_bon_2h",
		"price": "0.0003",
		"symbol": "eos",
		"tickerTime": 1636423200,
		"timestamp": "1636423200000"
	},
	"her_usdt_bon_8h": {
		"createTime": 1636401600,
		"exchange": "HER",
		"id": "377909407295693",
		"indexSymbol": "her_usdt_bon_8h",
		"price": "0.0006",
		"symbol": "usdt",
		"tickerTime": 1636401600,
		"timestamp": "1636401600000"
	},
	"her_dot_usdt_pi": {
		"exchange": "HER",
		"indexSymbol": "HER_dot_usdt_pi",
		"price": "-0.00150429",
		"symbol": "dot_usdt",
		"tickerTime": 1636429763,
		"timestamp": "1636429763000"
	},
	"her_eos_bon": {
		"createTime": 1636429740,
		"exchange": "HER",
		"id": "378027452069402",
		"indexSymbol": "her_eos_bon",
		"price": "0.0003",
		"symbol": "eos",
		"tickerTime": 1636429740,
		"timestamp": "1636429740000"
	},
	"her_dot_bon_2h": {
		"createTime": 1636423200,
		"exchange": "HER",
		"id": "377999727654453",
		"indexSymbol": "her_dot_bon_2h",
		"price": "0.0003",
		"symbol": "dot",
		"tickerTime": 1636423200,
		"timestamp": "1636423200000"
	}
}
```


|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| {key} | object | 否 |指数交易对 |

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| createTime | int | 否 |创建时间 |
| indexSymbol | string | 否 |指数交易对编号 |
| price | string | 否 |指数最新价格 |
| symbol | string | 否 |交易对 |
| tickerTime | int | 否 |指数时间戳 |
| timestamp | string | 否 |时间戳 |

### 最新合约变动
交割合约交割时，会触发此topic
topic : `exchange.public.contract `

```json
{
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
		"status": "2",
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
	}
```

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

## 服务监听Topic

- `服务端监听Topic，客户端向服务端推送[携带不同的操作码和不同的参数][global.public.platform]Topic消息，服务端向[$client/clientId]Topic推送消息 `

| Topic 名称             | 描述          | 举例 |
| ---------------------- | ------------- | ---- |
| global.public.platform | 服务监听Topic | -    |

### 基础参数

```json
{
	"action": 100,
	"from": "her5_qBxKO6AluUCOjqhSakZBAlkaDvuCxbsS",
	"timestamp": 1544614091,
	"extra": "",
	"data": {}
}
```

| 参数名    | 参数类型    | 是否可空 | 描述               |
| --------- | ----------- | -------- | ------------------ |
| action    | int         | 不可空   | -                  |
| from      | String(128) | 不可空   | clientId           |
| timestamp | int(10)     | 不可空   | 时间戳             |
| extra     | String(256) | 可空     | 额外参数，原样传回 |
| data      | JSON        | 不可空   | 具体action 参数    |

 * 用户需订阅特定topic：$client/clientId


### 获取全量K线

* 请求参数

``` json
{
	"action": 100,
	"from": "her5_qBxKO6AluUCOjqhSakZBAlkaDvuCxbsS",
	"timestamp": 1544614091,
	"extra": "",
	"data": {
		"symbol":"btc_usdt",
		"klineTime":"minute1",
		"timestamp":1544614091,
		"count":100,
	}
}
```

| 参数名         | 参数类型    | 是否可空 | 描述                             |
| -------------- | ----------- | -------- | -------------------------------- |
| action         | int         | 不可空   | -                                |
| from           | String(20)  | 不可空   | clientid                         |
| timestamp      | int(10)     | 不可空   | 请求时间戳                       |
| extra          | String(256) | 可空     | 额外参数，原样传回               |
| data           | JSON        | 不可空   | -                                |
| data.symbol    | String(16)  | 不可空   | 交易对                           |
| data.klineTime | string        | 不可空   | minute(分钟)、hour(小时)、day(日)、week(周)、month(月) |
| data.timestamp | int        | 不可空   | K线时间戳                        |
| data.count     | int        | 不可空   | K线时间戳往前多少条数据          |

* 返回参数

``` json
{
	"action": 100,
	"from": "her5_qBxKO6AluUCOjqhSakZBAlkaDvuCxbsS",
	"timestamp": 1544614091,
	"extra": "",
	"data":[["1636428360", "67569.77", "67576.6", "67569.77", "67576.6", "25", "11.00", 10]]
}
```
| 参数名    | 参数类型    | 是否可空 | 描述               |
| --------- | ----------- | -------- | ------------------ |
| action    | int         | 不可空   | -                  |
| from      | String(20)  | 不可空   | clientid           |
| timestamp | int(10)     | 不可空   | 时间戳             |
| extra     | String(256) | 可空     | 额外参数，原样传回 |
| data      | array        | 不可空   | k线数组           |


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

### 获取全量最新成交

* 请求参数

``` json
{
	"action": 101,
	"from": "her5_qBxKO6AluUCOjqhSakZBAlkaDvuCxbsS",
	"timestamp": 1544614091,
	"extra": "",
	"data": {
		"symbol": "btc_usdt",
		"count":"100"
	}
}
```

| 参数名      | 参数类型    | 是否可空 | 描述                    |
| ----------- | ----------- | -------- | ----------------------- |
| action      | int         | 不可空   | -                       |
| from        | String(20)  | 不可空   | clientid                |
| timestamp   | int(10)     | 不可空   | 时间戳                  |
| extra       | String(256) | 可空     | 额外参数，原样传回      |
| data        | JSON        | 不可空   | -                       |
| data.symbol | String(16)  | 不可空   | 交易对                  |
| data.count  | JSON        | 不可空   | 条数 |
  
* 返回参数

``` json
{
	"action": 101,
	"from": "her5_qBxKO6AluUCOjqhSakZBAlkaDvuCxbsS",
	"timestamp": 1544614091,
	"extra": "",
	"data": [{
		"dealAmount": "2",
		"dealPrice": "1.565375",
		"dealSide": "B",
		"dealTimestamp": "1638004108",
		"legalPrice": "0",
		"symbol": "ada_usdt"
	}]
}
```

| 参数名    | 参数类型    | 是否可空 | 描述               |
| --------- | ----------- | -------- | ------------------ |
| action    | int         | 不可空   | -                  |
| from      | String(20)  | 不可空   | clientid           |
| timestamp | int(10)     | 不可空   | 时间戳             |
| extra     | String(256) | 可空     | 额外参数，原样传回 |
| data      | array        | 不可空   | 最新成交列表           |

data

| 参数名    | 参数类型    | 是否可空 | 描述               |
| --------- | ----------- | -------- | ------------------ |
| dealAmount    | String         | 不可空   | 成交数量            |
| dealPrice      | String(20)  | 不可空   | 成交价           |
| dealSide      | String(20)  | 不可空   | 成交类型：B(买)，S(卖)          |
| dealTimestamp      | String(20)  | 不可空   | 成交时间           |
| symbol      | String(20)  | 不可空   | 交易对           |

### 获取合约全量K线

* 请求参数

``` json
{
	"action": 104,
	"from": "her5_qBxKO6AluUCOjqhSakZBAlkaDvuCxbsS",
	"timestamp": 1544614091,
	"extra": "",
	"data": {
		"symbol":"btc_usdt",
		"klineTime":"minute1",
		"timestamp":1544614091,
		"count":100,
	}
}
```

| 参数名         | 参数类型    | 是否可空 | 描述                             |
| -------------- | ----------- | -------- | -------------------------------- |
| action         | int         | 不可空   | -                                |
| from           | String(20)  | 不可空   | clientid                         |
| timestamp      | int(10)     | 不可空   | 请求时间戳                       |
| extra          | String(256) | 可空     | 额外参数，原样传回               |
| data           | JSON        | 不可空   | -                                |
| data.symbol    | String(16)  | 不可空   | 交易对                           |
| data.klineTime | string        | 不可空   | minute(分钟)、hour(小时)、day(日)、week(周)、month(月) |
| data.timestamp | int        | 不可空   | K线时间戳                        |
| data.count     | int        | 不可空   | K线时间戳往前多少条数据          |

* 返回参数

``` json
{
	"action": 104,
	"from": "her5_qBxKO6AluUCOjqhSakZBAlkaDvuCxbsS",
	"timestamp": 1544614091,
	"extra": "",
	"data":[["1636428360", "67569.77", "67576.6", "67569.77", "67576.6", "25", "11.00", 10]]
}
```

| 参数名    | 参数类型    | 是否可空 | 描述               |
| --------- | ----------- | -------- | ------------------ |
| action    | int         | 不可空   | -                  |
| from      | String(20)  | 不可空   | clientid           |
| timestamp | int(10)     | 不可空   | 时间戳             |
| extra     | String(256) | 可空     | 额外参数，原样传回 |
| data      | array        | 不可空   | k线数组           |


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


### 获取合约全量最新成交

* 请求参数

``` json
{
	"action": 105,
	"from": "her5_qBxKO6AluUCOjqhSakZBAlkaDvuCxbsS",
	"timestamp": 1544614091,
	"extra": "",
	"data": {
		"symbol": "btc_usdt",
		"count":"100"
	}
}
```

| 参数名      | 参数类型    | 是否可空 | 描述                    |
| ----------- | ----------- | -------- | ----------------------- |
| action      | int         | 不可空   | -                       |
| from        | String(20)  | 不可空   | clientid                |
| timestamp   | int(10)     | 不可空   | 时间戳                  |
| extra       | String(256) | 可空     | 额外参数，原样传回      |
| data        | JSON        | 不可空   | -                       |
| data.symbol | String(16)  | 不可空   | 合约编号                  |
| data.count  | JSON        | 不可空   | 数量 |
  
* 返回参数

``` json
{
	"action": 105,
	"from": "her5_qBxKO6AluUCOjqhSakZBAlkaDvuCxbsS",
	"timestamp": 1544614091,
	"extra": "",
	"data":[{
		"dealAmount": "2",
		"dealPrice": "1.565375",
		"dealSide": "B",
		"dealTimestamp": "1638004108",
		"legalPrice": "0",
		"symbol": "adausdt"
	}]
}
```

| 参数名    | 参数类型    | 是否可空 | 描述               |
| --------- | ----------- | -------- | ------------------ |
| action    | int         | 不可空   | -                  |
| from      | String(20)  | 不可空   | clientid           |
| timestamp | int(10)     | 不可空   | 时间戳             |
| extra     | String(256) | 可空     | 额外参数，原样传回 |
| data      | array        | 不可空   | 最新成交列表           |

data

| 参数名    | 参数类型    | 是否可空 | 描述               |
| --------- | ----------- | -------- | ------------------ |
| dealAmount    | String         | 不可空   | 成交数量            |
| dealPrice      | String(20)  | 不可空   | 成交价           |
| dealSide      | String(20)  | 不可空   | 成交类型：B(买)，S(卖)          |
| dealTimestamp      | String(20)  | 不可空   | 成交时间           |
| symbol      | String(20)  | 不可空   | 交易对           |





## 服务推送消息

- `服务端主动推送以下消息，除非客户端取消订阅[$client/clientId]主题，不同的操作码推送的消息不同`

### 基础信息

```json
{
	"action": 200,
	"from": "",
	"timestamp": 1544614091,
	"extra": "",
	"data": {}
}
```

| 参数名    | 参数类型    | 是否可空 | 描述               |
| --------- | ----------- | -------- | ------------------ |
| action    | int         | 不可空   | 操作码                  |
| from      | String(128) | 可空   | 空           |
| timestamp | int(10)     | 不可空   | 时间戳             |
| extra     | String(256) | 可空     | 额外参数，原样传回 |
| data      | JSON        | 不可空   | 具体action 参数    |

 * 用户需订阅特定topic：$client/clientId



### 用户最新余额
如果服务端发现用户余额变动后会主动推送以下信息。

```json
{
	"action": 200,
	"timestamp": 1544614091,
	"extra": "",
	"data": [{
			"btc": "1",
			"canRecharge": true,
			"canWithdraw": true,
			"cash": "1",
			"cny": "404229.53",
			"contractHost": "",
			"currency": "btc",
			"freeze": "0",
			"locked": "0",
			"mid": 5712839,
			"productMode": "exchange",
			"submode": "03",
			"usdt": "62240.01",
			"walletAddress": ""
		}]
}
```
data

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| btc  | string | 否 | BTC资产 |
| canRecharge  | string | 否 | 是否允许充值 |
| canWithdraw  | string | 否 | 是否允许提现 |
| cash  | string | 否 | 现金户 |
| cny  | string | 否 | CNY资产 |
| contractHost  | string | 否 | 合约所属 |
| currency  | string | 否 | 币种 |
| freeze  | string | 否 | 冻结户 |
| locked  | string | 否 | 锁仓户 |
| usdt  | string | 否 | USDT资产 |
| walletAddress  | string | 否 | 钱包地址 |
| submode  | string | 否 | 子账户:00(现货账户)、02（合约账户）、03（法币账户） |


### 用户持仓信息
如果服务端发现用户仓位变动后会主动推送以下信息。

* 请求参数

``` json
{
	"action": 201,
	"timestamp": 1544614091,
	"extra": "",
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
			"userMargin": "0"
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
			"userMargin": "0"
		}
	}
}
```

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

### 现货委托变化
如果服务端发现用户现货变动后会主动推送以下信息。

* 请求参数

``` json
{
	"action": 202,
	"timestamp": 1544614091,
	"extra": "",
	"data": [{
		"action": "UPDATE",
		"data": {
			"dealPrice": "4541.38",
			"fee": "4.54138",
			"leftAmountDeal": "1",
			"orderNo": "376-6237791-23266",
			"rightAmountDeal": "4541.38",
			"timestamp": "1636096701"
		},
		"orderNo": "376-6237791-23266",
		"symbol": "eth_usdt"
	}, {
		"action": "ADD",
		"data": {
			"averagePrice": "0",
			"createTime": 1636096701,
			"entrustMode": "10",
			"entrustPrice": "4539.73",
			"entrustStrategy": "GTC",
			"entrustType": "S",
			"fee": "0",
			"feeCoin": "usdt",
			"leftAmount": "4",
			"leftAmountDeal": "0",
			"leftCoin": "eth",
			"mid": 5712839,
			"orderNo": "376-6237791-23266",
			"orderStatus": "00",
			"rightAmount": "18158.92",
			"rightAmountDeal": "0",
			"rightCoin": "usdt",
			"symbol": "eth_usdt",
			"terminalCode": "HER21636096701",
			"terminalType": "API",
			"trades": []
		},
		"orderNo": "376-6237791-23266",
		"symbol": "eth_usdt"
	}, {
		"action": "DELETE",
		"data": {
			"averagePrice": "20790.26333333",
			"createTime": 1636097853,
			"entrustMode": "10",
			"entrustPrice": "62370.79",
			"entrustStrategy": "GTC",
			"entrustType": "B",
			"fee": "0.003",
			"feeCoin": "btc",
			"leftAmount": "7",
			"leftAmountDeal": "3",
			"leftCoin": "btc",
			"mid": 5712839,
			"orderNo": "376-6312827-34381",
			"orderStatus": "20",
			"rightAmount": "436595.53",
			"rightAmountDeal": "187112.37",
			"rightCoin": "usdt",
			"symbol": "btc_usdt",
			"terminalCode": "HER21636097853",
			"terminalType": "API",
			"trades": []
		},
		"orderNo": "376-6312827-34381",
		"symbol": "btc_usdt"
	}]

}
```

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| action | string | 否 |操作类型:ADD(新增委托)、UPDATE(修改委托)、DELETE(删除委托) |
| data | object | 否 |对应数据 |
| orderNo | string | 否 |订单编号 |
| symbol | string | 否 |交易对 |

data
当action为ADD

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| averagePrice  | string | 否 | 平均价 |
| createTime  | int | 否 | 创建时间 |
| entrustMode  | string | 否 | 委单模式：10(限价)、20(市价)、30(止盈止损) |
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
| orderStatus  | string | 否 |委单状态:00(交易中)、10(撮合中)、11(部分成交)、01（交易完成）、20（撤销中）、21（已撤单） |
| rightAmount  | string | 否 | 右边成交金额 |
| rightAmountDeal  | string | 否 | 右币已处理金额 |
| rightCoin  | string | 否 | 右币 |
| symbol  | string | 否 | 交易对 |
| terminalCode  | string | 否 | 终端编号 |
| terminalType  | string | 否 | 终端类型：Android（安卓）、IOS（ios）、WEB（web 端）、API（api） |
| trades  | list | 否 | 金额 |

当action为UPDATE

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| dealPrice | string | 否 |成交价 |
| fee | string | 否 |手续费 |
| leftAmountDeal | string | 否 |左币已成交 |
| orderNo | string | 否 |订单编号 |
| rightAmountDeal | string | 否 |右币已成交 |
| timestamp | string | 否 |成交时间戳 |

当actiong为DELETE

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| averagePrice  | string | 否 | 平均价 |
| createTime  | int | 否 | 创建时间 |
| entrustMode  | string | 否 | 委单模式：10(限价)、20(市价)、30(止盈止损) |
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
| orderStatus  | string | 否 | 委单状态:00(交易中)、10(撮合中)、11(部分成交)、01（交易完成）、20（撤销中）、21（已撤单） |
| rightAmount  | string | 否 | 右边成交金额 |
| rightAmountDeal  | string | 否 | 右币已处理金额 |
| rightCoin  | string | 否 | 右币 |
| symbol  | string | 否 | 交易对 |
| terminalCode  | string | 否 | 终端编号 |
| terminalType  | string | 否 | 终端类型：Android（安卓）、IOS（ios）、WEB（web 端）、API（api） |
| trades  | list | 否 | 金额 |

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


### 合约委托变化
如果服务端发现用户合约变动后会主动推送以下信息。

* 请求参数

``` json
{
	"action": 203,
	"timestamp": 1544614091,
	"extra": "",
	"data": [{
			"data": [{
				"action": "ADD",
				"data": {
					"averagePrice": "0",
					"contractCode": "btcw20211108",
					"createTime": 1636099273,
					"entrustMode": "10",
					"entrustPrice": "62145",
					"entrustSource": "1",
					"entrustType": "S",
					"fee": "0",
					"feeCoin": "usdt",
					"leftAmount": "2",
					"leftAmountDeal": "0",
					"leftCoin": "btc",
					"mid": 5712839,
					"orderNo": "376-6416239-65469",
					"orderStatus": "00",
					"rightAmount": "124.29",
					"rightAmountDeal": "0",
					"rightCoin": "usdt",
					"symbol": "btc_usdt",
					"terminalCode": "her2b44e8e20-f755-11eb-b8c9-37c73197ff9d",
					"terminalType": "Web",
					"trades": [

					]
				},
				"orderNo": "376-6416239-65469",
				"symbol": "btcw20211108"
			}],
			"success": true,
			"action": "create-order"
		},
		{
			"data": [{
				"action": "DELETE",
				"data": {
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
					"orderStatus": "20",
					"rightAmount": "124.126",
					"rightAmountDeal": "62.063",
					"rightCoin": "usdt",
					"symbol": "btc_usdt",
					"terminalCode": "her2b44e8e20-f755-11eb-b8c9-37c73197ff9d",
					"terminalType": "Web",
					"trades": [

					]
				},
				"orderNo": "376-5752824-70480",
				"symbol": "btc_usdt"
			}],
			"success": true,
			"action": "finish-cancel"
		},
		{
			"data": [{
					"action": "UPDATE",
					"data": {
						"dealPrice": "62145",
						"fee": "0.00310725",
						"leftAmountDeal": "1",
						"orderNo": "376-6409570-70548",
						"rightAmountDeal": "62.145",
						"timestamp": "1636099273"
					},
					"orderNo": "376-6409570-70548",
					"symbol": "btcw20211108"
				},
				{
					"action": "UPDATE",
					"data": {
						"dealPrice": "62145",
						"fee": "0.00310725",
						"leftAmountDeal": "1",
						"orderNo": "376-6416239-65469",
						"rightAmountDeal": "62.145",
						"timestamp": "1636099273"
					},
					"orderNo": "376-6416239-65469",
					"symbol": "btcw20211108"
				}
			],
			"success": true,
			"action": "finish-match"
		},
		{
			"data": "1",
			"positionType": "0",
			"success": true,
			"action": "change-mode"
		},
		{
			"data": "3",
			"positionType": "long",
			"success": true,
			"action": "change-leverage"
		},
		{
			"data": "1",
			"positionType": "short",
			"success": true,
			"action": "change-margin"
		}
	]

}
```

当action 为 change-margin、change-leverage、change-mode 时

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| action | string | 否 |操作类型：change-mode(切换仓位)、change-leverage(切换杠杆)、change-margin(修改保证金) |
| data | object | 是 |修改的数据 |
| positionType | string | 是 |仓位类型（short(空头)、long(多头)） 
| success | string | 否 |是否操作成功 |
| code | string | 是 |操作失败时，返回错误码 |
| msg | string | 是 |操作失败时，返回错误信息 |


当action 为 finish-match、finish-cancel、create-order 时

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| action | string | 否 |操作类型:ADD(新增委托)、UPDATE(修改委托)、DELETE(删除委托) |
| data | object | 否 |对应数据 |
| orderNo | string | 否 |订单编号 |
| symbol | string | 否 |交易对 |

data
当data中的action为ADD

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| averagePrice  | string | 否 | 平均价 |
| createTime  | int | 否 | 创建时间 |
| entrustMode  | string | 否 | 委单模式：10(限价)、20(市价)、30(止盈止损) |
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
| orderStatus  | string | 否 | 委单状态:00(交易中)、10(撮合中)、11(部分成交)、01（交易完成）、20（撤销中）、21（已撤单） |
| rightAmount  | string | 否 | 右边成交金额 |
| rightAmountDeal  | string | 否 | 右币已处理金额 |
| rightCoin  | string | 否 | 右币 |
| symbol  | string | 否 | 交易对 |
| terminalCode  | string | 否 | 终端编号 |
| terminalType  | string | 否 | 终端类型：Android（安卓）、IOS（ios）、WEB（web 端）、API（api） |
| trades  | list | 否 | 金额 |

当data中的action为UPDATE

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| dealPrice | string | 否 |成交价 |
| fee | string | 否 |手续费 |
| leftAmountDeal | string | 否 |左币已成交 |
| orderNo | string | 否 |订单编号 |
| rightAmountDeal | string | 否 |右币已成交 |
| timestamp | string | 否 |成交时间戳 |

当data中的action为DELETE

|  参数名   | 参数类型  |可空|介绍|
|  ----  | ----  |----  |----  |
| averagePrice  | string | 否 | 平均价 |
| createTime  | int | 否 | 创建时间 |
| entrustMode  | string | 否 | 委单模式：10(限价)、20(市价)、30(止盈止损) |
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
| orderStatus  | string | 否 | 委单状态:00(交易中)、10(撮合中)、11(部分成交)、01（交易完成）、20（撤销中）、21（已撤单） |
| rightAmount  | string | 否 | 右边成交金额 |
| rightAmountDeal  | string | 否 | 右币已处理金额 |
| rightCoin  | string | 否 | 右币 |
| symbol  | string | 否 | 交易对 |
| terminalCode  | string | 否 | 终端编号 |
| terminalType  | string | 否 | 终端类型：Android（安卓）、IOS（ios）、WEB（web 端）、API（api） |
| trades  | list | 否 | 金额 |

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


