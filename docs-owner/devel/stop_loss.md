## 止盈

最小回报率minimal_roi，**必须设置**，推荐在策略代码中设置

定水平时自动平仓。这个选项可以帮助您在交易中实现及时止盈，防止回报率在达到峰值后回撤。`minimal_roi` 配置是一个字典，其中键表示交易持续时间（以分钟为单位），值表示在此时间段内应达到的最小回报率。

```json
"minimal_roi": {
    "0": 0.1,
    "10": 1.5,
    "30": 0.05,
    "60": 0.01,
    "120": 0
}
```

或者在策略代码中设置

```
minimal_roi: = {
    "0": 0.1,
    "10": 1.5,
    "30": 0.05,
    "60": 0.01,
    "120": 0
}
```

此配置的含义是：

- 在交易开始后的前 10 分钟内，如果达到 10% 的回报率，将平仓（止盈）。
- 在10分钟至30分钟内，如果达到150%的回报率，将平仓止盈。
- 在 30 分钟至 60 分钟之间，如果达到 5% 的回报率，将平仓（止盈）。
- 在 60 分钟至 120 分钟之间，如果达到 1% 的回报率，将平仓（止盈）。
- 在交易进行了 120 分钟（2 小时）之后，可以在任何回报率水平上平仓

***ROI table: 超过10（1000%）将禁用最小投资回报率***

```
minimal_roi = {
        "0": 100
    }
```

## 止损

### stoploss

https://www.freqtrade.io/en/stable/stoploss/

**Required.** 必须项，推荐在策略代码中设置

用于设置交易策略中的止损值。止损是一种风险管理策略，用于在交易亏损达到预定的百分比时自动平仓，以减少进一步的损失。`stoploss` 的值通常是一个负数，表示允许的最大亏损百分比。

例如，以下是一个 `stoploss` 配置的示例：

```json
stoploss = -0.05
```

 此配置表示，如果某个交易的亏损达到或超过 5%，则将自动平仓。在实际交易中，根据您的策略和风险承受能力来设置合适的止损值非常重要。过低的止损值可能会导致频繁的交易失败，而过高的止损值可能会导致您在不良交易中承受过多的损失

禁用止损

```solidity
stoploss = -1
```

### Stop Loss On-Exchange/Freqtrade

可以在交易所或者freqtrade中设置止损，在订单类型order_types中设置止损使用的价格方式，更多关于order_types参数介绍查看订单类型模块

```json
"order_types" = {
    "entry": "limit",								
    "exit": "limit",								
    "emergency_exit": "market",  //紧急止损
    "force_entry": "market",				
    "force_exit": "market",					
    "stoploss": "market",  //止损委托类型：市价或限价
    "stoploss_price_type": "last", //止损使用的价格类型：last：最新价（合约价）、mark：标记价格（现货价格）
    "stoploss_on_exchange": False,  
    "stoploss_on_exchange_interval": 60,
    "stoploss_on_exchange_limit_ratio": 0.99,
}
```

stoploss_on_exchange: 为true时将提交止损单到交易所，为False则在本地设置止损单

该选项在不同的交易所止损所用的止损价方式不同，这里列举部分交易所的止损类型

| Exchange        | stop-loss type          |
| :-------------- | :---------------------- |
| Binance         | limit                   |
| Binance Futures | market, limit           |
| Huobi           | limit                   |
| kraken          | market, limit           |
| Gate            | limit                   |
| Okx             | limit                   |
| Kucoin          | stop-limit, stop-market |

### 止损方式

1. 静态止损
2. 移动止损
3. 移动止损，自定义正向止损
4. **仅在交易达到一定偏移量才进行移动止损**。推荐使用
5. 自定义止损方法

#### 静态止损

```
stoploss = -0.10
```

如果买入价为100$，止损为-10%，则当价格低于90$的时候将触发止损

#### 移动止损

```
stoploss = -0.10
trailing_stop = True
```

- 如果买入价格为100$，止损为-10%，如果一次价格跌破90$将触发止损
- 如果价格先涨到102$，那么现在的止损价将为：102 - 102 * 10% = 91.8，如果价格跌倒101$，止损价将保持91.8$保持不变，且在该位置保持移动止损。

总结：止损将调整为始终观察到的最高价格的 -10%。

#### 移动止损，自定义正向止损

Trailing stop loss, custom positive loss

当您购买（购买 - 费用）处于亏损状态时，您也会拥有一个默认止损(`stoploss`)，但是一旦您获得了利润，系统将使用新的止损，它可以有不同的值。 例如，您的默认止损为 -10%，但一旦您的利润超过 0%（例如 0.1%），将使用不同的追踪止损。

**注意：**如果你想达到盈亏平衡后才更改止损，请参考下一个止损方法

该止损方法两个必须的值：trailing_stop = True，trailing_stop_positive = 0.02（必须大于0）

```json
    stoploss = -0.10
    trailing_stop = True
    trailing_stop_positive = 0.02
    trailing_stop_positive_offset = 0.0
    trailing_only_offset_is_reached = False  # Default - not necessary for this example
```

- 当买入的价格为100，止损定义为-10%，价位一次跌破 90 将触发止损
- 现在价格涨到102时，现在止损将变为 102 - 102 * 2% = 99.96，此时止损将以99.96不变，并将跟随价格以2%的增量增长，比如此时价格跌到101%，止损依旧为99.96

如果设置了trailing_stop_positive_offset的值

```
stoploss = -0.10
trailing_stop = True
trailing_stop_positive = 0.02
trailing_stop_positive_offset = 0.03
```

- 买入价100，止损定义-10%，价格一次跌破90，触发止损
- 如果价格涨到102，止损将为 102 - 102 * 10% = 91.8
- 如果价格涨到103.5 > 设置达到的追踪止损价位 100 + 100 * 3% = 103，则止损为 103.5 - 103.5 * 2% = 101.43
- 现在价格跌到102时将保持101.43价格不变

也就是只有当价格达到了设定的移动止损位置才出发移动止损

#### 仅在交易达到一定偏移量才进行移动止损。

在达到设定的偏移量价位之前仍然保持静态止损，然后在市场转向时追踪交易以获利。

`trailing_only_offset_is_reached = true`

Configuration (offset is buy_price + 3%):

```solidity
 stoploss = -0.10
 trailing_stop = True
 trailing_stop_positive = 0.02
 trailing_stop_positive_offset = 0.03
 trailing_only_offset_is_reached = True
```

- 比如买入价格为100，止损位-10%，价格一次跌破90将触发止损，直到价格上涨到设定的偏移量 100 + 100 * 3% = 103时才触发移动止损
- 如果价格上涨到103，则当前止损为103 - 103 * 2% = 100.94，如果价格跌倒101仍然保持100.94的止损

**注意：**：确保此值 (trailing_stop_positive_offset) 低于最小 ROI，否则最小 ROI 将首先应用并卖出交易。

#### 自定义止损

https://www.freqtrade.io/en/stable/strategy-callbacks/#custom-stoploss

在已开仓的情况下，每5秒钟执行一次自定义止损，知道订单被关闭

必须在策略中设置`use_custom_stoploss = True`才生效， 止损价格只能向上移动止损，如果`custom_stoploss`返回的止损值低于之前设置的止损价位，那么它将被忽略。传统的止损值作为绝对的底线，在首次为交易调用此方法之前，它将被设定为初始止损，并且仍然是必需的。

所有基于时间的计算都应基于 current_time 完成 - 不鼓励使用 `datetime.now() `或 `datetime.utcnow()`，因为这会破坏回测支持。 建议在使用自定义止损值时禁用 `trailing_stop`。 两者可以协同工作，但您可能会遇到移动止损以将价格推高，而您的自定义函数不希望这样，从而导致行为冲突。

##### 移动止损

其他的止损方式参考官方文档

官方例子：使用初始止损，直到利润超过 4%，然后使用当前利润的 50% 的追踪止损，最低为 2.5%，最高为 5%。请注意，止损只能增加，低于当前止损的值将被忽略。

```python
from datetime import datetime, timedelta
from freqtrade.persistence import Trade

class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    use_custom_stoploss = True

    def custom_stoploss(self, pair: str, trade: 'Trade', current_time: datetime,
                        current_rate: float, current_profit: float, **kwargs) -> float:

        if current_profit < 0.04:
            return self. # return a value bigger than the initial stoploss to keep using the initial stoploss

        # After reaching the desired offset, allow the stoploss to trail by half the profit
        desired_stoploss = current_profit / 2

        # Use a minimum of 2.5% and a maximum of 5%
        return max(min(desired_stoploss, 0.05), 0.025)
```

##### 阶梯式止损

此示例不是持续落后于当前价格，而是根据当前利润设置固定的止损价格水平。

- 当收益达到%20之前使用初始止损self.stoploss
- 当 profit > 20% 设置止损为7%
- 当 profit > 25 % 设置止损为15%
- 当 profit > 40 % 设置止损为 25%

```python
from datetime import datetime
from freqtrade.persistence import Trade
from freqtrade.strategy import stoploss_from_open

class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    use_custom_stoploss = True

    def custom_stoploss(self, pair: str, trade: 'Trade', current_time: datetime,
                        current_rate: float, current_profit: float, **kwargs) -> float:

        # evaluate highest to lowest, so that highest possible stop is used
        if current_profit > 0.40:
            # 当收益达到40%使用高于开盘价的25%止损
            return stoploss_from_open(0.25, current_profit, is_short=trade.is_short, leverage=trade.leverage)
        elif current_profit > 0.25:
            #当收益达到25%使用高于开盘价的15%止损
            return stoploss_from_open(0.15, current_profit, is_short=trade.is_short, leverage=trade.leverage)
        elif current_profit > 0.20:
            return stoploss_from_open(0.07, current_profit, is_short=trade.is_short, leverage=trade.leverage)

        # return maximum stoploss value, keeping current stoploss price unchanged
        return 1
```


`stoploss_from_open()` 基于开盘价的移动止损，主要作用：修正做多做空数据

```python
if current_profit < 0.4:
    return -1
retrun stoploss_from_open(0.2, current_profit, is_short=trade.is_short, leverage=trade.leverage) * trade.leverage

```
- 做多，开盘价为100，当前收益 < 40%, , 不适用止损
- 当收益为40%时，此时止损价 ~= 140 * （1 - abs(0.2 * 25 / 25)）= 112
- 当价回落到112时触发止损

当open_relative_stop值越接近当前收益，止损位越接近当前价格，越容易止损

- 当open_relative_stop = 0.1 当前收益为40%, 此时止损位 =  140 * （1 - abs(0.3 * 25 / 25)）= 98

当价位跌倒98时会触发止损


止损价格计算
```python
if self.is_short:
            new_loss = float(current_price * (1 + abs(stoploss / leverage)))
        else:
            new_loss = float(current_price * (1 - abs(stoploss / leverage)))
```

##### 使用数据框示例中的指标自定义止损



### 止损和杠杆

Stoploss and Leverage

止损应被视为“此交易的风险”——因此 100 美元交易的 10% 止损意味着你愿意在此交易中损失 10 美元（10%）——如果价格移动 10% 至 不足之处。

使用杠杆时，应用相同的原则 - 止损定义交易风险（您愿意损失的金额）。

因此，**10 倍交易的 10% 止损将触发 1% 的价格变动**。 如果您的赌注金额（自有资金）为 100 美元 - 此交易在 10 倍（杠杆后）时为 1000 美元。 如果价格移动 1% - 您已经损失了 10 美元的自有资金 - 因此在这种情况下将触发止损。

确保意识到这一点，并避免使用太紧的止损（在 10 倍杠杆下，10% 的风险可能太小而无法让交易稍微“喘口气”）。
