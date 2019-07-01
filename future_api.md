# 交易API

* [概述](#open-api)

* [频率控制](#open-apilimited)

* [开启API权限](#open-apisecret)

* [代码示例](#open-apicode)

* [状态码](#open-apistatuscode)

* [期货API列表](#open-apifuturelist)
  * [获取交易对列表](#open-apifuturelist-symbollist)
  * [用户余额(持仓)](#open-apifuturelist-accountbalance)
  * [修改保证金](#open-apifuturelist-transfermargin)
  * [下单](#open-apifuturelist-order)
  * [撤单](#open-apifuturelist-remove)  
  * [平仓](#open-apifuturelist-close)
  * [当前委托](#open-apifuturelist-activeorders)
  * [委托历史](#open-apifuturelist-orderhistory)
  * [已成交 ](#open-apifuturelist-orderfills)

-----------

## <span id="open-api">概述 </span>

- 所有交易API请求都使用HTTP POST
- 交易API需要在官网申请API需要的key/secret
- 请求的header里添加version = '2.0'
- 请求的header里添加key/sign，sign=hash('sha256', $post_data.$secret)
- 请求的nonce参数为当前系统时间戳，单位为秒，nonce不早/晚于当前系统时间10秒
- 访问频率最快为100ms间隔
- 访问ixx站点的币对时,需要在请求的header里添加from = 'ixx'

## <span id="open-apisecret">频率控制 </span>  
我们对API的请求频率进行控制，具体频率参数请参考接口详情

对 API 的请求，以下标头将被返回︰
```
"X-ratelimit-limit: 1000"
"X-ratelimit-next: 500"
```
X-ratelimit-limit为当前接口的频率控制间隔,具体因接口不同而参数不同
如果你已经被频率限制，你将收到 403 响应, 以及一个额外的标头X-ratelimit-next, 它意味着你在重试前需要等待的时间
 


## <span id="open-apisecret">开启API权限 </span>

### 申请API Key
用户的API权限在网站的个人中心->我的API内获取。点击申请API即可获得，其中API Key是IX提供给API用户的访问密钥，API Secret是用于对请求参数签名的私钥。
官网地址: www.ix.com
备用地址: www.ixex.io

**_注意： 请勿向任何人泄露这两个参数，这两个参数关乎您账号的安全。_**

### 请求认证
在调用 API 时，需要提供 API Key 作为每个请求的身份识别，并且通过secret对请求数据加签

#### 公共参数

字段名 | 字段释义 | 字段类型 | 是否必填 | 默认值 | 参数类型 | 说明
 :-: | :-: | :-: | :-: | :-: | :-: | :-: 
key | 在平台申请的API_KEY |  string | 是 | 无 | http header | 用于身份识别
sign | 签名信息 |  string | 是 | 无 |  http header |按照一定规则形成的签名信息
version| api版本| string | 是 | 2.0 | http header | 用于区分api版本
nonce | 请求发起时的时间戳,单位:秒 | string | 是 | 无 |  http body | nonce不早/晚于当前系统时间10秒


 #### 如何进行签名
 1. 将对应业务的接口参数和除sign外的公共参数以http GET请求形式拼接, 如下
 
 ``` php
$post_data = 'leverage=100&symbol=BTCUSD&nonce=1542434791';

 ```
 
 2. 对拼接后的字符串进行加签
 ``` php
$secret = 't7T0YlFnYXk0Fx3JswQsDrViLg1Gh3DUU5Mr';
$sign = hash('sha256', $post_data.$secret)
 // sign = 670e3e4aa32b243f2dedf1dafcec2fd17a440e71b05681550416507de591d908
 ```
 
 3.header附加上key和sign参数，发送http请求
 
 ```http
 POST /order/active HTTP/1.1
 Content-Type: application/x-www-form-urlencoded
 key: 43f2dedf1dafcec2fd17a440e71b056815
 sign: 670e3e4aa32b243f2dedf1dafcec2fd17a440e71b05681550416507de591d908
 version: 2.0 
 
leverage=100&symbol=BTCUSD&nonce=1542434791
 
 ```

## <span id="open-apicode">代码示例 </span> 
Python：
``` Python
import requests
import hashlib
import json
import time
from urllib import parse

key = 'your key'
secret = 'your secret'
url = "https://api.ixex.io/order/active"
symbol = 'BTC_USDT'
nonce = int(time.time())
payload = {'nonce': nonce, 'symbol': symbol, 'page': 1, 'size': 10}
payload_str = parse.urlencode(payload)
sign = hashlib.sha256((payload_str+secret).encode("utf-8")).hexdigest()
headers = {'Content-Type': 'application/json', 'version': '2.0', 'key': key, 'sign': sign}
response = requests.post(url, data=json.dumps(payload), headers=headers)
```

PHP：
``` PHP
$key = 'your key'
$secret = 'your secret'
$url = "https://api.ixex.io/order/active";
$symbol = 'BTC_USDT';
$nonce = time();
$payload = ['nonce' => $nonce, 'symbol' => $symbol, 'page' => 1, 'size' => 10];
$payload_str = http_build_query($payload, '', '&');
$sign = hash('sha256', urldecode($payload_str).$secret);
$headers = ['Content-Type' => 'application/json', 'version'=> '2.0', 'key' => $key, 'sign' => $sign];
$response = Requests::post($url, $headers, json_encode($payload));
```

JavaScript：
``` JavaScript
let key = 'your key'
let secret = 'your secret'
let url = "https://api.ixex.io/order/active"
let symbol = 'BTC_USDT'
let nonce = Math.floor(Date.now() / 1000)
let sign = sha256("nonce="+ nonce + "&symbol=" + symbol + "&page=" + 1 + "&size=" + 10 + secret)
$.ajax({
  url: url,
  type: 'post',
  data: {
    nonce: nonce,
    symbol: symbol,
    page: 1,
    size: 10
  },
  headers: {
    version: '2.0',
    key: key,
    sign: sign
  },
  ...
```


## <span id="open-apistatuscode">状态码 </span>

| 错误代码        | 详细描述    |    
| :-----    | :-----   |    
|200	|	正常|    
|400	|	缺少参数|    
|401	|	缺少认证|    
|403	|	请求过快|    
|413	|	请求过大|    
|500	|	非法请求|    
|30001	|	交易对不存在|    
|30002	|	下单数量不合法|    
|30003	|	下单金额不合法|    


## <span id="open-apifuturelist">期货API列表 </span>

### <span id="open-apifuturelist-symbollist">获取交易对列表 POST /contract/symbol/list </span>
- 参数
  - nonce 时间戳
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
  - id 唯一编号
  - name 合约名,BTC的交易对，比如BTC永续
  - currency 交易单位USD结算
  - product 合约BTC
  - amount_scale 数量精度
  - price_scale 价格精度
  - value_scale 价值精度
  - max_amount 最大下单数量
  - max_total 最大下单金额
  - min_amount 最小下单数量
  - min_total 最小下单金额
  - state 1可用 0不可用
  - make_rate maker费率
  - take_rate taker费率
- 限定访问间隔时间
  -	1000毫秒
- 示例
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/contract/symbol/list -d 'nonce=1536826456'
```

### <span id="open-apifuturelist-accountbalance">用户余额(持仓) POST /future/account/balance/list </span>
- 参数
  - nonce 时间戳
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
  - wallet_available 钱包可用余额
  - available 可用余额
  - currency 币种
  - holding 目前仓位数量
  - price 开仓价格
  - leverage 杠杆倍数
  - margin_available 保证金余额
  - position_margin 仓位保证金
  - order_margin 委托保证金
  - unrealized 未实现盈亏
  - realized 已实现盈亏
  - liq_price 强平价格
  - adl ADL值
- 限定访问间隔时间
  -	1000毫秒
- 示例
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/future/account/balance/list -d 'nonce=1536826456'
```

### <span id="open-apifuturelist-transfermargin">修改保证金 POST /future/account/transfer_margin </span>
- 参数
  - nonce 时间戳
  - amount 增加或减少数量
  - currency 币种
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
  ok
- 限定访问间隔时间
  -	1000毫秒
- 示例
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/future/account/transfer_margin -d 'nonce=1536826456&currency=BTCUSD&amount=-0.3302'
```

### <span id="open-apifuturelist-order">下单 POST /contract/order </span>
- 参数
  - nonce 时间戳
  - amount 
  - price 
  - type 下单类型 1 限价 2市价 3限价止损 4市价止损 5限价止盈 6市价止盈
  - side 方向 2卖 1买
  - symbol 交易对
  - leverage 杠杆倍数 -1代表全仓 10 20 100
  - passive 是否被动委托 0否 1是
  - trigger_price 触发价格
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
- 限定访问间隔时间
  -	100毫秒
- 示例
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/contract/order -d 'nonce=1536826456&symbol=FUTURE_BTCUSD&side=1&type=1&price=3500&amount=1&leverage=10'
```

### <span id="open-apifuturelist-remove">撤单 POST /contract/remove</span>
- 参数
  - nonce 时间戳
  - symbol 交易对 
  - order_id 订单id 
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
- 限定访问间隔时间
  -	100毫秒
- 示例
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/contract/remove -d 'nonce=1536826456&symbol=FUTURE_BTCUSD&order_id='
```

### <span id="open-apifuturelist-close">平仓 POST /contract/close</span>
- 参数
  - nonce 时间戳
  - symbol 交易对
  - price 平仓价格
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
- 限定访问间隔时间
  -	1000毫秒
- 示例
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/contract/close -d 'nonce=1536826456&symbol=FUTURE_BTCUSD&price=3600'
```

### <span id="open-apifuturelist-activeorders">当前委托 POST /contract/activeorders</span>
- 参数
  - nonce 时间戳
  - page
  - size
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
    - 订单id
    - symbol 合约类型
    - amount 数量
    - price 委托价格
    - total 已成交额
    - executed 已成交量
    - state 状态 1委托中未成交 2委托中部分成交
    - create_time 下单时间
    - update_time 更新时间
- 限定访问间隔时间
  -	1000毫秒
- 示例
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/contract/activeorders -d 'nonce=1536826456&page=1&size=10'
```

### <span id="open-apifuturelist-orderhistory">委托历史 POST /contract/orderhistory</span>
- 参数
  - nonce 时间戳
  - page
  - size
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
    - 订单id
    - symbol 合约类型
    - amount 数量(张)
    - price 委托价格
    - total 已成交额
    - executed 已成交量(张)
    - state 状态 1:委托中未成交, 2:委托中限价部分成交, 3:完全成交, 4:撤单全部, 5:撤单部分成交, 6:市价部分成交, 7:市价部分成交盘口全被吃空
    - create_time 下单时间
    - update_time 更新时间
- 限定访问间隔时间
  -	1000毫秒
- 示例
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/contract/orderhistory -d 'nonce=1536826456&page=1&size=10'
```


### <span id="open-apifuturelist-orderfills">已成交 POST /future/account/orderfills</span>
- 参数
  - nonce 时间戳
  - page
  - size
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
    - id 成交单ID
    - order_id 订单ID
    - uid 用户ID
    - symbol 交易对
    - product 商品简称
    - currency 货币简称
    - side 1买 2卖
    - type 1限价 2市价 3止盈 4止损
    - price 成交价
    - amount 成交量
    - amount_total 委托数量
    - amount_last 成交量
    - total 成交额
    - fee 手续费
    - create_time 成交时间
- 限定访问间隔时间
  -	1000毫秒
- 示例
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/future/account/orderfills -d 'nonce=1536826456&page=1&size=10'
```