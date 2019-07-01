# 行情和盘口WebSocket API 参考 v1

## 连接频道

以**Beta**环境为例，使用WebSocket连接 **/v1/ticker/EOS_ETH** 频道，例如：

``` javascript
var socket = new WebSocket("wss://ws.ixex.io/v1/ticker/EOS_ETH");
```

客户端请求为：

``` text
GET wss://ws.ixex.io/v1/ticker/EOS_ETH HTTP/1.1
Host: ws.ixex.io
Connection: Upgrade
Pragma: no-cache
Cache-Control: no-cache
Upgrade: websocket
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: zVu/qw6mod9ivrbSex2GBw==
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
```

服务端响应为：

``` text
HTTP/1.1 101 Switching Protocols
Date: Thu, 14 Jun 2018 11:49:21 GMT
Connection: upgrade
Expires: Thu, 01 Jan 1970 00:00:01 GMT
Cache-Control: no-cache, no-store, must-revalidate
Upgrade: WebSocket
Sec-WebSocket-Accept: ip3WBDpEnyzPMRPngEgDZgM+6lU=
```

如果连接不存在的频道，服务器会响应HTTP 404状态码，例如：

``` javascript
var socket = new WebSocket("wss://ws.ixex.io/v1/unknown");
```

客户端请求为：
``` text
GET wss://ws.ixex.io/v1/unknown HTTP/1.1
Host: ws.ixex.io
Connection: Upgrade
Pragma: no-cache
Cache-Control: no-cache
Upgrade: websocket
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: BP2dOFyaJFIIb2+E77/7fA==
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
```

服务器响应为：

``` text
HTTP/1.1 404 Not Found
Date: Thu, 14 Jun 2018 11:51:47 GMT
Content-Type: text/html
Transfer-Encoding: chunked
Connection: keep-alive
Server: nginx
Content-Encoding: gzip
```

如果连接的频道存在，但参数错误，例如交易对不存在，服务器响应后会主动断开连接，例如：
``` javascript
var socket = new WebSocket("wss://ws.ixex.io/v1/ticker/EOS");
```

客户端请求为：
``` text
GET wss://ws.ixex.io/v1/ticker/EOS HTTP/1.1
Host: ws.ixex.io
Connection: Upgrade
Pragma: no-cache
Cache-Control: no-cache
Upgrade: websocket
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: /ulaHe5O/DCNxbF30evMWQ==
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
```

服务端响应为：
``` text
HTTP/1.1 101 Switching Protocols
Date: Thu, 14 Jun 2018 11:54:49 GMT
Connection: upgrade
Expires: Thu, 01 Jan 1970 00:00:01 GMT
Cache-Control: no-cache, no-store, must-revalidate
Upgrade: WebSocket
Sec-WebSocket-Accept: 4RdyPxpw7+pNAG6bpU5l/gwSIEY=
```

## 推送格式
所有推送消息均为JSON，包含:
+ 预留状态码(**code**);
+ 数据(**data**)

示例如下：
``` javascript
{
    "code":0,   // 预留状态码，当前版本尚未使用
    "data":{
        ...     // 推送数据
    }
}
```


## 公共频道

### 大盘周期频道
URI：**/v1/market/histories/{period}**  

| 参数名         | 类型        | 说明 | 是否必需 |
| ------------- |:------------- | :-----| :-----:|
| period | 字符串 | 行情周期 | 是 |

#### 周期参数(**period**)

|  参数  | 定义  |  参数  | 定义  |  参数  | 定义  |
| :---: | :---: | :---: | :---: | :---: | :---: |
|  1m   |  一分钟  |  1h   |  一小时  |  1d   |    日    |  
|  5m   |  五分钟  |  2h   |  两小时  |  1w   |    周    |
|  15m  | 十五分钟 |  4h   |  四小时  |  1M   |    月    |
|  30m  | 三十分钟 |  6h   |  六小时  |
|       |         |  12h  | 十二小时 |

连接示例：
``` javascript
var socket = new WebSocket("wss://ws.ixex.io/v1/market/histories/1d");
```

推送示例：
``` javascript
{
    "code": 0,
    "data": [
        {
            "time": 1529798400000,  // 时间，单位毫秒  
            "pair": "ETH_BTC",
            "values": [
                "0.076973",         // 起始价
                "0.076838",         // 结束价
                "0.07606",          // 最低价
                "0.077378",         // 最高价
                "27.3411",          // 成交量
                "2.1000977805",     // 成交额
                "0.076973",         // 上一周期结束价
                "121",              // 成交笔数
                "0.205"             // 现量
            ]
        },
        {
            "time": 1529798400000,
            "pair": "EOS_ETH",
            "values": [
                "0.017866",
                "0.017433",
                "0.017178",
                "0.018546",
                "2013.0353",
                "35.5771107184",
                "0.017866",
                "123",
                "18.4377"
            ]
        },
        {
            "time": 1529798400000,
            "pair": "EOS_ETH",
            "values": [
                "0.001372",
                "0.00135",
                "0.001346",
                "0.001386",
                "2446.2574",
                "3.3256018155",
                "0.001372",
                "128",
                "20.1341"
            ]
        },
        {
            "time": 1529798400000,
            "pair": "EOS_USDT",
            "values": [
                "8.4371",
                "8.2357",
                "8.1938",
                "8.5026",
                "2681.5773",
                "22255.92854128",
                "8.4371",
                "126",
                "18.2521"
            ]
        },
        {
            "time": 1529798400000,
            "pair": "BTC_USDT",
            "values": [
                "6136.14",
                "6072.41",
                "5937.46",
                "6210.94",
                "3.7433",
                "22847.165608",
                "6136.14",
                "134",
                "0.0267"
            ]
        },
        {
            "time": 1529798400000,
            "pair": "ETH_USDT",
            "values": [
                "473.25",
                "465.78",
                "463.37",
                "475.6",
                "55.9782",
                "26252.350092",
                "473.25",
                "128",
                "0.4091"
            ]
        }
    ]
}
```

### 大盘报价频道
URI：**/v1/market/tickers**  

连接示例：
``` javascript
var socket = new WebSocket("wss://ws.ixex.io/v1/market/tickers");
```

推送示例：
``` javascript
{
    "code": 0,
    "data": [
        {
            "time": 1530862375429,                  // 时间，单位毫秒
            "pair": "ETH_BTC",                      // 交易对名称
            "current": "0.070269",                  // 现价
            "volume_current": "0.2399",             // 现量
            "increment_24h": "-0.001124",           // 24小时价格变化量
            "highest_24h": "0.072012",              // 24小时最高价
            "lowest_24h": "0.070082",               // 24小时最低价
            "volume_24h": "222.45380000000003",     // 24小时成交量
            "highest_bid": "0.069891",              // 买一价
            "highest_bid_amount": "0.1",            // 买一量
            "lowest_ask": "0.070642",               // 卖一价
            "lowest_ask_amount": "0.1",             // 卖一量
            "change_24h": "-1.5744"                 // 24小时涨跌幅，百分比
        },
        {
            "time": 1530862374942,
            "pair": "EOS_ETH",
            "current": "0.018232",
            "volume_current": "29.6759",
            "increment_24h": "-0.000795",
            "highest_24h": "0.019975",
            "lowest_24h": "0.017696",
            "volume_24h": "27341.4681",
            "highest_bid": "0.017182",
            "highest_bid_amount": "9.33",
            "lowest_ask": "0.018661",
            "lowest_ask_amount": "30.15",
            "change_24h": "-4.1783"
        },
        {
            "time": 1530862375218,
            "pair": "EOS_BTC",
            "current": "0.001289",
            "volume_current": "17.0712",
            "increment_24h": "-0.000074",
            "highest_24h": "0.001379",
            "lowest_24h": "0.001284",
            "volume_24h": "20040.206399999995",
            "highest_bid": "0.001277",
            "highest_bid_amount": "4.85",
            "lowest_ask": "0.001305",
            "lowest_ask_amount": "4.89",
            "change_24h": "-5.4292"
        },
        {
            "time": 1530862375103,
            "pair": "EOS_USDT",
            "current": "8.3349",
            "volume_current": "43.1394",
            "increment_24h": "-0.633",
            "highest_24h": "9.1656",
            "lowest_24h": "8.3123",
            "volume_24h": "46383.8888",
            "highest_bid": "8.2472",
            "highest_bid_amount": "7.66",
            "lowest_ask": "8.4231",
            "lowest_ask_amount": "7.76",
            "change_24h": "-7.0585"
        },
        {
            "time": 1530862375228,
            "pair": "BTC_USDT",
            "current": "6460.67",
            "volume_current": "0.06",
            "increment_24h": "-176.13",
            "highest_24h": "6812.95",
            "lowest_24h": "5922.07",
            "volume_24h": "59.355399999999996",
            "highest_bid": "6425.71",
            "highest_bid_amount": "0.0985",
            "lowest_ask": "6492.45",
            "lowest_ask_amount": "0.0971",
            "change_24h": "-2.6538"
        },
        {
            "time": 1530862374902,
            "pair": "ETH_USDT",
            "current": "453.87",
            "volume_current": "0.9173",
            "increment_24h": "-16.49",
            "highest_24h": "478.88",
            "lowest_24h": "452",
            "volume_24h": "494.0506",
            "highest_bid": "451.62",
            "highest_bid_amount": "0.098",
            "lowest_ask": "456.42",
            "lowest_ask_amount": "0.097",
            "change_24h": "-3.5058"
        }
    ]
}
```

### 报价频道

URI：**/v1/ticker/{pair}**   
参数：

| 参数名 | 类型   | 说明       | 是否必需 |
| ------ | :----- | :--------- | :------: |
| pair   | 字符串 | 交易对名称 |    是    |

连接示例：
``` javascript
var socket = new WebSocket("wss://ws.ixex.io/v1/ticker/EOS_ETH");
```

推送示例：
``` javascript
{
    "code": 0,
    "data": {
        "time": 1530862854941,          // 时间，单位毫秒
        "pair": "EOS_ETH",              // 交易对名称
        "current": "0.018334",          // 现价
        "volume_current": "0",          // 现量
        "increment_24h": "-0.000658",   // 24小时价格变化量
        "highest_24h": "0.019975",      // 24小时最高价
        "lowest_24h": "0.017568",       // 24小时最低价
        "volume_24h": "27285.2885",     // 24小时成交量
        "highest_bid": "0.018149",      // 买一价
        "highest_bid_amount": "7.62",   // 买一量
        "lowest_ask": "0.018516",       // 卖一价
        "lowest_ask_amount": "7.7",     // 卖一量
        "change_24h": "-3.4646"         // 24小时涨跌幅，百分比
    }
}
```





### 周期频道

URI：**/v1/history/{pair}/{period}**  
参数：

| 参数名         | 类型        | 说明 | 是否必需 |
| ------------- |:------------- | :-----| :-----:|
| pair | 字符串 | 交易对名称 | 是 |
| period | 字符串 | 行情周期 | 是 |

#### 周期参数(**period**)

|  参数  | 定义  |  参数  | 定义  |  参数  | 定义  |
| :---: | :---: | :---: | :---: | :---: | :---: |
|  1m   |  一分钟  |  1h   |  一小时  |  1d   |    日    |  
|  5m   |  五分钟  |  2h   |  两小时  |  1w   |    周    |
|  15m  | 十五分钟 |  4h   |  四小时  |  1M   |    月    |
|  30m  | 三十分钟 |  6h   |  六小时  |
|       |         |  12h  | 十二小时 |

连接示例：
``` javascript
var socket = new WebSocket("wss://ws.ixex.io/v1/history/EOS_ETH/1m");
```

推送示例：  

``` javascript
{
    "code": 0,
    "data": [
        {
            "time": 1529808300000,  // 时间，单位毫秒
            "pair": "EOS_ETH",      // 交易对名称
            "values": [
                "0.017436",         // 起始价
                "0.017436",         // 结束价
                "0.017436",         // 最低价
                "0.017436",         // 最高价
                "0",                // 成交量
                "0",                // 成交额
                "0.017436",         // 上一周期结束价
                "0",                // 成交笔数
                "17.5732"           // 现量
            ]
        }
    ]
}
```

### 成交频道

URI：**/v1/deal/{pair}**  
参数：

| 参数名         | 类型  | 说明          | 是否必需 |
| ------------- | :-----|:------------- | :-----:|
| pair | 字符串 | 交易对名称 | 是 |

连接示例：
``` javascript
var socket = new WebSocket("wss://ws.ixex.io/v1/deal/EOS_ETH");
```

推送示例：  

``` javascript
{
    "code": 0,
    "data": [        
        {
            "time": 1529811082258,  // 时间，单位毫秒
            "side": "buy",          // 成交方向
            "values": [
                "0.017262",         // 成交价
                "16.1633"           // 成交量
            ]
        },
        {
            "time": 1529811012254,
            "side": "sell",
            "values": [
                "0.017277",
                "15.1537"
            ]
        }
    ]
}
```



### 订单频道

URI：**/v1/orderbook/{pair}/{offset}/{accuracy}/{size}**  
参数：

| 参数名         | 类型        | 说明 | 是否必需 |
| ------------- |:------------- | :-----| :-----:|
| pair | 字符串 | 交易对名称 | 是 |
| size | 整型 | 档位数，可选：5、10、20 | 是 |
| offset | 整型 | 浮点位偏移量 | 是 |
| accuracy | 整型 | 统计精度 | 是 |

#### 统计精度(**accuracy**)
用于多空双方查询、订单推送接口，进位模式half_even

| 参数 |     定义      |
| :--: | :-----------: |
|  1   |  精度保留为1  |
|  2   | 精度保留为2.5 |
|  5   |  保留精度为5  |



#### 浮点位偏移量(**offset**)

用于涉及买卖双方的API，进位模式half_even，可选范围：**0～10**；
偏移量最大为10，当交易品种浮点位精度小于10时，最大偏移量为该交易品种浮点位精度

推送格式：

| 参数名         | 类型  | 说明          |
| :------------ | :-----|:------------- |
| asks | 数组 | 卖方订单档位集合 |
| bids | 数组 | 买方订单档位集合 |

连接示例
``` javascript
var socket = new WebSocket("wss://ws.ixex.io/v1/orderbook/EOS_ETH/0/1/5");
```

示例：
``` javascript
{
    "code": 0,
    "data": {
        "asks": [
            {
                "values": [
                    "0.017409", // 卖价
                    "7.96"      // 卖量
                ]
            },
            {
                "values": [
                    "0.017481",
                    "40"
                ]
            },
            {
                "values": [
                    "0.017535",
                    "23.39"
                ]
            },
            {
                "values": [
                    "0.017562",
                    "9.84"
                ]
            },
            {
                "values": [
                    "0.019102",
                    "17.74"
                ]
            }
        ],
        "bids": [
            {
                "values": [
                    "0.017021", // 买价
                    "6.81"      // 买量
                ]
            },
            {
                "values": [
                    "0.0169",
                    "2.01"
                ]
            },
            {
                "values": [
                    "0.016827",
                    "71.92"
                ]
            },
            {
                "values": [
                    "0.01631",
                    "108.5"
                ]
            },
            {
                "values": [
                    "0.015604",
                    "62.15"
                ]
            }
        ]
    }
}
```
