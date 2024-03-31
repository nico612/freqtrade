## 订单类型配置

```
order_types = {
    "entry": "limit",								
    "exit": "limit",								
    "emergency_exit": "market",
    "force_entry": "market",				
    "force_exit": "market",					
    "stoploss": "market",
    "stoploss_price_type": "last",
    "stoploss_on_exchange": False,  
    "stoploss_on_exchange_interval": 60,
    "stoploss_on_exchange_limit_ratio": 0.99,
}
```

limit：限价单，使用固定值，当市场达到该值时买入，当价格变化很快，有可能成交不了

market: 市价单，发出市价单时，会根据交易所最接近的价格成交，有肯能跟设置的价格有差别

- entry：买入方式 limit：限价单，market：市价单
- exit：卖出方式 limit：限价单，market：市价单
- emergency_exit：紧急退出，如果交易所stoploss止损单创建失败，紧急退出将被初始化，默认情况下使用市价单   limit：限价单，market：市价单
- force_entry：强制买入，通过控制器（telegram或者RestApi发送"/forceentry"命令）买入，limit：限价单，market：市价单
- force_exit：强制卖出(telegram或者RestApi发送"/forceexit"命令)：limit：限价单，market：市价单
- stoploss：止损委托类型：limit：限价，market：市价
- stoploss_price_type：止损使用的价格类型：last：最新价（合约价）、mark：标记价格（现货价格）
- stoploss_on_exchange：是否在提交订单时同时在交易所设置止损单，默认fasle
- stoploss_on_exchange_interval：检查交易所止损单间隔时间，必须大于5s，默认60s（1分钟）
- stoploss_on_exchange_limit_ratio：止损边界比例

## 订单价格配置

配置项：https://www.freqtrade.io/en/stable/configuration/

配置详解：https://www.freqtrade.io/en/stable/includes/pricing/#market-order-pricing

这里以买入订单价为例，卖出一样的配置

### Entry price

入场价格配置信息

```json
 "entry_pricing": {
      "price_side": "other",
      "use_order_book": true,
      "order_book_top": 1,
      "price_last_balance": 0.0,
      "check_depth_of_market": {
        "enabled": false,
        "bids_to_ask_delta": 1
      }
    }
```

- price_side：机器人买入时选择订单薄的方向，ask：卖出方向，bid：为买入方向， some：为做多或做空时同方向，other：使用其他方式（比如订单类型的买入和卖出设置为market市价单时，这里就需要设置为other）

  ```
  ...
  103
  102
  101  # ask
  -------------Current spread
  99   # bid
  98
  97
  ...
  ```

  如果设置为bid，机器人将选择99作为买入价，如果设置为ask则为101

  如果合约交易做多做空，推荐使用"same"、"other"

  | direction | Order | setting | price | crosses spread |
  | :-------- | :---- | :------ | :---- | :------------- |
  | long      | buy   | ask     | 101   | yes            |
  | long      | buy   | bid     | 99    | no             |
  | long      | buy   | same    | 99    | no             |
  | long      | buy   | other   | 101   | yes            |
  | short     | sell  | ask     | 101   | no             |
  | short     | sell  | bid     | 99    | yes            |
  | short     | sell  | same    | 101   | no             |
  | short     | sell  | other   | 99    | yes            |

- use_order_book：*Defaults to `True`.*一个布尔值，表示是否使用订单簿来确定进入价格。如果为False，表示不使用订单簿，则会根据设置的entry_pricing.price_side和entry_pricing.price_last_balance参数，从市场深度中选择一个最佳的价格进行买入。

- order_book_top：*Defaults to `1`.*当使用订单簿时，指定要使用订单簿中的第几个价格来确定进入价格。默认为1，表示使用订单簿中最顶部的价格。

- price_last_balance：*Defaults to `0`.* 当不使用订单簿时，用于确定进入价格的系数。默认为0，表示使用与交易方向相同的最佳价格，为1表示使用最近的成交价格，0到1之间的值表示使用两者之间的价格插值。

- check_depth_of_market.enabled：*Defaults to `false`* 一个布尔值，表示是否检查订单簿中买卖盘面的深度。默认为False，表示不进行检查。

- check_depth_of_market.bids_to_ask_delta：*Defaults to `0`.* 如果启用了订单簿深度检查，则此参数定义买盘和卖盘之间的差异比率。值小于1表示卖盘的数量更大，大于1表示买盘的数量更大。默认值为0，表示不进行检查。

## 常用设置

### 使用限价单

```json
 "order_types":{
      "entry": "limit",
      "exit": "limit",
      "stoploss": "market",
      "stoploss_on_exchange": false
    },
    "entry_pricing": {
      "price_side": "same",
      "use_order_book": true,
      "order_book_top": 1,
      "price_last_balance": 0.0,
      "check_depth_of_market": {
        "enabled": false,
        "bids_to_ask_delta": 1
      }
    },
    "exit_pricing": {
      "price_side": "same",
      "use_order_book": true,
      "order_book_top": 1
    },
```

### 使用市价单

```js
"order_types":{
      "entry": "market",
      "exit": "market",
      "stoploss": "market",
      "stoploss_on_exchange": false
    },
    "entry_pricing": {
      "price_side": "other",
      "use_order_book": true,
      "order_book_top": 1,
      "price_last_balance": 0.0,
      "check_depth_of_market": {
        "enabled": false,
        "bids_to_ask_delta": 1
      }
    },
    "exit_pricing": {
      "price_side": "other",
      "use_order_book": true,
      "order_book_top": 1
    },
```

