# 交易API

## 概述

- 所有交易API请求都使用HTTP POST
- 交易API需要在官网申请API需要的key/secret
- 请求的header里添加version = '2.0'
- ixx平台请求的header里添加from = 'ixx'
- 请求的header里添加key/sign，sign=hash('sha256', $post_data.$secret)
- 请求的nonce参数为当前系统时间戳，单位为秒，nonce不早/晚于当前系统时间10秒
- 单个接口访问频率最快为100ms间隔

## 开启API权限

用户的API权限在网站的个人中心->我的API内获取。点击申请API即可获得，其中API Key是IX提供给API用户的访问密钥，API Secret是用于对请求参数签名的私钥。

**_注意： 请勿向任何人泄露这两个参数，这两个参数关乎您账号的安全。_**    

## 代码示例
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
$headers = ['Content-Type:application/json','version:2.0', 'key:'.$key, 'sign:' .$sign ];
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


## 状态码

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

## API列表

## 获取所有交易对的详情 POST /symbol/list
- 参数
  - nonce 时间戳
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data(id/name/product/price_scale/amount_scale/min_amount/max_amount/min_total/max_total/make_rate/take_rate/state)
  - message
- 字段说明
  - id ID 
  - name 交易对名称 
  - product 商品 
  - currency 货币 
  - price_scale 报价小数位 
  - amount_scale 报量小数位 
  - min_amount 最小下单量 
  - max_amount 最大下单量 
  - min_total 最小下单额 
  - max_total 最大下单额 
  - make_rate 挂单手续费率 
  - take_rate 吃单手续费率 
  - state 状态 1上线 2下线
- 示例
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/symbol/list -d 'nonce=1536826456'
```

## 获取余额列表 POST /balance/list
- 参数
  - nonce 时间戳
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data(user_id/currency/available/ordering/withdrawing/quota/max_quota/rates)
  - message
- 字段说明
  - user_id 用户ID 
  - currency 货币 
  - available 可用量 
  - ordering 委托锁定量 
  - withdrawing 提币锁定量 
  - quota 当前额度 
  - max_quota 最大额度 
  - rates 币种对各法币汇率 
- 示例
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/balance/list -d 'nonce=1536826456'
```

## 下单 POST /order/create
- 参数
  - nonce 时间戳
  - symbol 交易对名称
  - side 1买 2卖
  - type 1限价 2市价
  - price 限价单时报价，市价填0
  - amount 限价单时报量，市价买单填0
  - locked 市价买时的锁定额，市价买单必填
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data(id/user_id/symbol/side/type/price/amount/locked)
  - message
- 字段说明
  - id 订单ID 
  - user_id 用户ID 
  - symbol 交易对名称 
  - side 1买 2卖 
  - type 1限价 2市价
  - price 限价单时报价
  - amount 限价单时报量
  - locked 市价买时的锁定额
- 示例
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/order/create -d 'nonce=1536826456&symbol=BTC_USDT&side=1&type=1&price=6000&amount=1'
```

## 撤单 POST /order/remove
- 参数
  - nonce 时间戳
  - symbol 交易对名称
  - order_id 订单ID
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 示例
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/order/remove -d 'nonce=1536826456&symbol=BTC_USDT&order_id=123'
```

## 查询委托 POST /order/active
- 参数
  - nonce 时间戳
  - symbol 交易对名称
  - page 页码
  - size 每页数量
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data(id/user_id/symbol/product/currency/side/type/price/amount/locked/executed/total/fee/state/create_time/update_time)
  - message
- 字段说明
  - id 订单ID 
  - user_id 用户ID 
  - symbol 交易对名称 
  - product 商品 
  - currency 货币 
  - side 1买 2卖 
  - type 1限价 2市价
  - price 报价
  - amount 报量
  - locked 锁定量
  - executed 成交量
  - total 成交额
  - fee 手续费
  - state 1未成交 2部分成交
  - create_time 下单时间
  - update_time 最近成交时间
- 示例
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/order/active -d 'nonce=1536826456&symbol=BTC_USDT&page=1&size=10'
```

## 历史订单 POST /order/history
- 参数
  - nonce 时间戳
  - page 页码
  - size 每页数量
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data(id/user_id/symbol/product/currency/side/type/price/amount/locked/executed/total/fee/state/create_time/update_time)
  - message
- 示例
- 字段说明
  - id 订单ID 
  - user_id 用户ID 
  - symbol 交易对名称 
  - product 商品 
  - currency 货币 
  - side 1买 2卖 
  - type 1限价 2市价
  - price 报价
  - amount 报量
  - locked 锁定量
  - executed 成交量
  - total 成交额
  - fee 手续费
  - state 3全部成交 4未成交撤单 5部分成交撤单 6未成交系统取消 7部分成交系统取消
  - create_time 下单时间
  - update_time 最近成交时间
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/order/history -d 'nonce=1536826456&page=1&size=10'
```

### 查询活跃委托订单详情 POST /order/query
- 参数
  - nonce 时间戳
  - symbol 交易对名称
  - order_id 订单ID
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data(id/user_id/symbol/product/currency/side/type/price/amount/locked/executed/total/fee/state/create_time/update_time)
  - message
- 示例
- 字段说明
  - id 订单ID 
  - user_id 用户ID 
  - symbol 交易对名称 
  - product 商品 
  - currency 货币 
  - side 1买 2卖 
  - type 1限价 2市价
  - price 报价
  - amount 报量
  - locked 锁定量
  - executed 成交量
  - total 成交额
  - fee 手续费
  - state 1未成交 2部分成交
  - create_time 下单时间
  - update_time 最近成交时间
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.ixex.io/order/query -d 'nonce=1536826456&symbol=BTC_USDT&order_id=123'
```