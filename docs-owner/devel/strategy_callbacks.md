
# Strategy Callbacks

在量化交易策略中，主要的策略函数（如 populate_indicators()、populate_entry_trend()、populate_exit_trend()）应该以矢量化的方式使用，并且在回测过程中只会被调用一次。而回调函数（callbacks）则是“在需要时”被调用的。

因此，在回调函数中应避免进行大量的计算，以避免在操作过程中出现延迟。这些回调函数可能会在进入/退出交易时被调用，或者在交易持续期间不断被调用。

目前可用的回调函数有：

- bot_start()
- bot_loop_start()
- custom_stake_amount()
- custom_exit()
- custom_stoploss()
- custom_entry_price() 和 custom_exit_price()
- check_entry_timeout() 和 check_exit_timeout()
- confirm_trade_entry()
- confirm_trade_exit()
- adjust_trade_position()
- adjust_entry_price()
- leverage()

这些回调函数提供了灵活性，可以在特定的交易阶段或情况下执行特定的操作。然而，要注意在这些回调函数中避免进行耗时的计算，以确保在交易过程中不会出现不必要的延迟或性能问题。

可以在bot-basics中找到回调调用顺序


## 自定义退出  Custom exit signal
custom_exit()允许定义自定义的退出信号，以指示应该卖出指定的头寸。这对于需要为每个单独的交易定制退出条件或者需要交易数据来做出退出决策非常有用。

以下是一个示例，展示如何基于当前利润使用不同的指标，并且在交易持续超过一天时退出交易：

```python
from datetime import datetime
from freqtrade.persistence import Trade

class AwesomeStrategy(IStrategy):
    def custom_exit(self, pair: str, trade: 'Trade', current_time: 'datetime', current_rate: float,
                    current_profit: float, **kwargs):
        dataframe, _ = self.dp.get_analyzed_dataframe(pair, self.timeframe)
        last_candle = dataframe.iloc[-1].squeeze()

        # Above 20% profit, sell when rsi < 80
        if current_profit > 0.2:
            if last_candle['rsi'] < 80:
                return 'rsi_below_80'

        # Between 2% and 10%, sell if EMA-long above EMA-short
        if 0.02 < current_profit < 0.1:
            if last_candle['emalong'] > last_candle['emashort']:
                return 'ema_long_below_80'

        # Sell any positions at a loss if they are held for more than one day.
        if current_profit < 0.0 and (current_time - trade.open_date_utc).days >= 1:
            return 'unclog'
```
这个示例展示了根据不同的利润水平和交易持续时间设置不同的退出条件。当利润达到一定水平时，可以基于RSI或者移动平均线等指标触发退出信号。同时，如果交易持续时间超过一天并且处于亏损状态，也会触发退出信号。

## 自定义止损 Custom Stoploss

`custom_stoploss()` 方法会在每次迭代中（大约每5秒）针对开放的交易被调用，直到交易被关闭。

要使用 `custom_stoploss()` 方法，必须在策略对象上设置 `use_custom_stoploss=True`。

这个方法用于设定一个自定义的止损价格，但是要注意，这个止损价格只能上调（不能低于之前设置的止损价格）。传统的止损值作为一个绝对的下限，它将作为初始的止损（在该方法首次为某个交易调用之前），并且仍然是必须设置的。

该方法必须返回一个止损值（浮点数 / 数字），表示相对于当前价格的百分比。比如，如果当前价格是200美元，返回0.02将会将止损价格设定为2%低于当前价格，即196美元。在回测过程中，current_rate（以及current_profit）是根据蜡烛图的最高价（或者做空交易时的最低价）提供的，而最终的止损则是针对蜡烛图的最低价（或者做空交易时的最高价）进行评估的。

返回值的绝对值被使用（符号被忽略），因此返回0.05或-0.05会产生相同的效果，即比当前价格低5%的止损。如果你不想修改止损，可以返回None，这是唯一安全的返回方式。

交易所上的止损与跟踪止损类似，并且交易所上的止损会根据 `stoploss_on_exchange_interval` 配置进行更新。

### Adjust stoploss after position adjustments

在进行头寸调整后，可能需要根据你的策略需要，在两个方向上调整止损。为此，Freqtrade会在订单成交后使用 after_fill=True 进行额外的调用，允许策略在任何方向上移动止损（也可以扩大止损和当前价格之间的差距，这在其他情况下是被禁止的）。

兼容性说明：

只有在 after_fill 参数作为你的 custom_stoploss 函数定义的一部分时，才会进行这个额外的调用。因此，这不会影响（也不会出现意外）已存在的运行中的策略。

### Custom stoploss examples

这是一个使用自定义止损函数来模拟普通的动态止损（Trailing Stop Loss）的例子。这个例子是设定一个4%的动态止损，跟踪最高价格的4%之后触发止损。这个例子使用了简单的方法来实现：

#### 普通的动态止损 Trailing stop via custom stoploss

```python
# 需要额外导入的模块
from datetime import datetime
from freqtrade.persistence import Trade

class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    use_custom_stoploss = True

    def custom_stoploss(self, pair: str, trade: 'Trade', current_time: datetime,
                        current_rate: float, current_profit: float, after_fill: bool, 
                        **kwargs) -> Optional[float]:
        """
        Custom stoploss logic, returning the new distance relative to current_rate (as ratio).
        e.g. returning -0.05 would create a stoploss 5% below current_rate.
        The custom stoploss can never be below self.stoploss, which serves as a hard maximum loss.

        For full documentation please go to https://www.freqtrade.io/en/latest/strategy-advanced/

        When not implemented by a strategy, returns the initial stoploss value.
        Only called when use_custom_stoploss is set to True.

        :param pair: Pair that's currently analyzed
        :param trade: trade object.
        :param current_time: datetime object, containing the current datetime
        :param current_rate: Rate, calculated based on pricing settings in exit_pricing.
        :param current_profit: Current profit (as ratio), calculated based on current_rate.
        :param after_fill: True if the stoploss is called after the order was filled.
        :param **kwargs: Ensure to keep this here so updates to this won't break your strategy.
        :return float: New stoploss value, relative to the current_rate
        """
        return -0.04

```
这个自定义止损函数（custom_stoploss()）会跟踪最高价格的4%。当价格达到最高点的4%以下时，会触发止损。请根据自己的策略和需求调整这个百分比，以符合你的交易策略。

#### 基于时间的动态止损 Time based trailing stop
这个例子展示了一个基于时间的动态止损设置。它在最初的60分钟内使用初始止损，之后变为10%的动态止损，在2个小时（120分钟）后变为5%的动态止损。
```python
from datetime import datetime, timedelta
from freqtrade.persistence import Trade

class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    use_custom_stoploss = True

    def custom_stoploss(self, pair: str, trade: 'Trade', current_time: datetime,
                        current_rate: float, current_profit: float, after_fill: bool, 
                        **kwargs) -> Optional[float]:

        # Make sure you have the longest interval first - these conditions are evaluated from top to bottom.
        if current_time - timedelta(minutes=120) > trade.open_date_utc:
            return -0.05
        elif current_time - timedelta(minutes=60) > trade.open_date_utc:
            return -0.10
        return None


```
在这个例子中，custom_stoploss() 方法根据当前时间和交易开始的时间来选择适当的动态止损。在60分钟后，它切换到10%的动态止损，在120分钟后，又切换到5%的动态止损。这个例子展示了如何根据时间动态调整止损，根据你的交易策略，你可以根据不同的时间段设置不同的止损水平。

#### 基于时间的动态止损 Time based trailing stop with after-fill adjustments
这个例子展示了一个基于时间的动态止损设置，同时考虑了在附加订单成交后进行调整的情况。在最初的60分钟内使用初始止损，之后变为10%的动态止损，在2个小时（120分钟）后变为5%的动态止损。如果附加订单成交，那么将止损设定为新的开仓价格下方10%。
```python
from datetime import datetime, timedelta
from freqtrade.persistence import Trade

class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    use_custom_stoploss = True

    def custom_stoploss(self, pair: str, trade: 'Trade', current_time: datetime,
                        current_rate: float, current_profit: float, after_fill: bool, 
                        **kwargs) -> Optional[float]:

        if after_fill: 
            # After an additional order, start with a stoploss of 10% below the new open rate
            return stoploss_from_open(0.10, current_profit, is_short=trade.is_short, leverage=trade.leverage)
        # Make sure you have the longest interval first - these conditions are evaluated from top to bottom.
        if current_time - timedelta(minutes=120) > trade.open_date_utc:
            return -0.05
        elif current_time - timedelta(minutes=60) > trade.open_date_utc:
            return -0.10
        return None

```

这个示例中，custom_stoploss() 方法根据当前时间和交易开始的时间来选择适当的动态止损。如果 after_fill 为真（即附加订单成交后），它会将止损设置为新的开仓价格下方10%。在60分钟后，它切换到10%的动态止损，在120分钟后，又切换到5%的动态止损。这个例子同时考虑了附加订单成交后如何进行止损调整，以适应新的市场情况。


#### 根据交易对设置不同的止损 Different stoploss per pair
这个例子展示了如何根据交易对设置不同的止损策略。在这个示例中，对于'ETH/BTC'和'XRP/BTC'交易对，使用10%的动态止损；对于'LTC/BTC'交易对，使用5%的动态止损；对于其他所有交易对，使用15%的动态止损。

```python
from datetime import datetime
from freqtrade.persistence import Trade

class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    use_custom_stoploss = True

    def custom_stoploss(self, pair: str, trade: 'Trade', current_time: datetime,
                        current_rate: float, current_profit: float, after_fill: bool,
                        **kwargs) -> Optional[float]:

        if pair in ('ETH/BTC', 'XRP/BTC'):
            return -0.10
        elif pair in ('LTC/BTC'):
            return -0.05
        return -0.15

```
在这个例子中，custom_stoploss() 方法根据交易对的不同返回不同的止损百分比。你可以根据自己的交易对和策略需求，设置不同的止损百分比，以适应不同的市场情况和交易对特性。


#### 动态跟踪止损 Trailing stoploss with positive offset
这个例子展示了如何设置一个动态跟踪止损，当利润达到4%以上时，使用当前利润的50%作为动态止损，但动态止损的最小值为2.5%，最大值为5%。

```python
from datetime import datetime, timedelta
from freqtrade.persistence import Trade

class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    use_custom_stoploss = True

    def custom_stoploss(self, pair: str, trade: 'Trade', current_time: datetime,
                        current_rate: float, current_profit: float, after_fill: bool,
                        **kwargs) -> Optional[float]:

        if current_profit < 0.04:
            return -1 # return a value bigger than the initial stoploss to keep using the initial stoploss

        # After reaching the desired offset, allow the stoploss to trail by half the profit
        desired_stoploss = current_profit / 2

        # Use a minimum of 2.5% and a maximum of 5%
        return max(min(desired_stoploss, 0.05), 0.025)

```

这个自定义止损函数（custom_stoploss()）在利润低于4%时返回一个值，确保继续使用初始止损。当利润达到4%以上时，它会计算出当前利润的50%作为动态跟踪止损，并限制在2.5%到5%之间。你可以根据自己的策略需求，调整这些百分比来适应不同的交易策略。

#### 根据当前利润设置固定的止损价位 Stepped stoploss

这个例子展示了如何根据当前利润设置固定的止损价位，而不是持续跟随当前价格。

```python
from datetime import datetime
from freqtrade.persistence import Trade
from freqtrade.strategy import stoploss_from_open

class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    use_custom_stoploss = True

    def custom_stoploss(self, pair: str, trade: 'Trade', current_time: datetime,
                        current_rate: float, current_profit: float, after_fill: bool,
                        **kwargs) -> Optional[float]:

        # evaluate highest to lowest, so that highest possible stop is used
        if current_profit > 0.40:
            return stoploss_from_open(0.25, current_profit, is_short=trade.is_short, leverage=trade.leverage)
        elif current_profit > 0.25:
            return stoploss_from_open(0.15, current_profit, is_short=trade.is_short, leverage=trade.leverage)
        elif current_profit > 0.20:
            return stoploss_from_open(0.07, current_profit, is_short=trade.is_short, leverage=trade.leverage)

        # return maximum stoploss value, keeping current stoploss price unchanged
        return None

```
这个示例中，custom_stoploss() 方法根据当前利润选择适当的固定止损价位。一旦利润达到一定阈值，止损价位会被设置为固定的百分比，随着利润增加，止损价位也会不断调整。你可以根据需要添加更多的止损价位和利润阈值，以适应不同的交易策略和市场情况。

#### 使用数据帧中的指标来设置自定义止损 Custom stoploss using an indicator from dataframe example

这个例子展示了如何使用数据帧中的指标来设置自定义止损。在这个例子中，使用了数据帧中的抛物线 SAR 作为止损。

```python
class AwesomeStrategy(IStrategy):

    def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        # <...>
        dataframe['sar'] = ta.SAR(dataframe)

    use_custom_stoploss = True

    def custom_stoploss(self, pair: str, trade: 'Trade', current_time: datetime,
                        current_rate: float, current_profit: float, after_fill: bool,
                        **kwargs) -> Optional[float]:

        dataframe, _ = self.dp.get_analyzed_dataframe(pair, self.timeframe)
        last_candle = dataframe.iloc[-1].squeeze()

        # Use parabolic sar as absolute stoploss price
        stoploss_price = last_candle['sar']

        # Convert absolute price to percentage relative to current_rate
        if stoploss_price < current_rate:
            return stoploss_from_absolute(stoploss_price, current_rate, is_short=trade.is_short)

        # return maximum stoploss value, keeping current stoploss price unchanged
        return None

```

在这个示例中，custom_stoploss() 方法通过读取数据帧中的 SAR 指标来设置止损价位。然后将绝对价格转换为相对于当前价格的百分比，并将其作为自定义止损。你可以根据策略需要，使用数据帧中的不同指标来定义自己的自定义止损策略。

### 常用的止损计算函数 Common helpers for stoploss calculations

#### 相对于开盘价格的止损值 Stoploss relative to open price¶
stoploss_from_open()是一个辅助函数，用于计算相对于开盘价格的止损值。它可以帮助你在custom_stoploss()中返回相对于入场价格的止损值，以达到预期的交易利润。

这个函数可以被用于在自定义止损函数中返回相对于开盘价格的止损值。例如：

```python
from datetime import datetime
from freqtrade.persistence import Trade
from freqtrade.strategy import IStrategy, stoploss_from_open

class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    use_custom_stoploss = True

    def custom_stoploss(self, pair: str, trade: 'Trade', current_time: datetime,
                        current_rate: float, current_profit: float, after_fill: bool,
                        **kwargs) -> Optional[float]:

        # once the profit has risen above 10%, keep the stoploss at 7% above the open price
        if current_profit > 0.10:
            return stoploss_from_open(0.07, current_profit, is_short=trade.is_short, leverage=trade.leverage)

        return 1


```

这个例子展示了如何在自定义止损函数中使用stoploss_from_open()函数，根据交易的利润或指定的相对价格设置止损。

**注意：**

在custom_stoploss()中使用stoploss_from_open()时，提供了无效的输入参数。这种情况可能出现在current_profit参数低于指定的open_relative_stop时，导致无法产生有效的止损值。

为了解决这个问题，有几种方法可以尝试：

- 避免阻止止损售出： 在confirm_trade_exit()方法中检查exit_reason，确保不阻止止损售出。这可以确保在current_profit < open_relative_stop时，止损售出不会被阻止。

- 使用返回值 stoploss_from_open(...) 或 1： 当current_profit < open_relative_stop时，使用返回值 stoploss_from_open(...) 或 1 的惯用法，请求保持止损不变。


#### 将绝对价格转换为相对于当前价格的百分比 Stoploss percentage from absolute price¶
stoploss_from_absolute()是一个辅助函数，用于将绝对价格转换为相对于当前价格的百分比，从而可以在custom_stoploss()中返回相对于当前价格的止损值。

这个函数可以用于在自定义止损函数中，将绝对价格转换为相对于当前价格的百分比。例如：

```python
from datetime import datetime
from freqtrade.persistence import Trade
from freqtrade.strategy import IStrategy, stoploss_from_absolute, timeframe_to_prev_date

class AwesomeStrategy(IStrategy):

    use_custom_stoploss = True

    def populate_indicators_1h(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
        dataframe['atr'] = ta.ATR(dataframe, timeperiod=14)
        return dataframe

    def custom_stoploss(self, pair: str, trade: 'Trade', current_time: datetime,
                        current_rate: float, current_profit: float, after_fill: bool,
                        **kwargs) -> Optional[float]:
        dataframe, _ = self.dp.get_analyzed_dataframe(pair, self.timeframe)
        trade_date = timeframe_to_prev_date(self.timeframe, trade.open_date_utc)
        candle = dataframe.iloc[-1].squeeze()
        sign = 1 if trade.is_short else -1
        return stoploss_from_absolute(current_rate + (side * candle['atr'] * 2), 
                                      current_rate, is_short=trade.is_short,
                                      leverage=trade.leverage)


```

这段代码演示了如何使用stoploss_from_absolute()函数，在自定义止损函数custom_stoploss()中返回相对于绝对价格的止损值。这个例子中，我们希望以当前价格的2倍ATR（平均真实范围）以下的价格设置止损。ATR是一种用于衡量价格波动性的技术指标。

在这个例子中，我们首先计算了1小时时间框架上的ATR指标，并在populate_indicators_1h()方法中添加到数据帧中。在custom_stoploss()方法中，我们获取了最后一个数据点的ATR值，并使用stoploss_from_absolute()函数计算了相对于当前价格的止损值。

这样的策略允许根据市场波动性调整止损，以便更好地适应不同的市场条件。




## 自定义订单价格规则 Custom order price rules

默认情况下，Freqtrade使用订单簿自动设置订单价格。但是您也可以根据您的策略创建自定义订单价格。

### 自定义入场和出场价 Custom order entry and exit price example

自定义订单价格提供了根据特定策略条件定义交易的入场和离场价格的灵活性。您可以通过在您的策略文件中创建 custom_entry_price() 和 custom_exit_price() 函数来使用此功能。

以下是演示如何实现自定义入场和离场价格的示例：

```python
from datetime import datetime, timedelta, timezone
from freqtrade.persistence import Trade

class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    def custom_entry_price(self, pair: str, trade: Optional['Trade'], current_time: datetime, proposed_rate: float,
                           entry_tag: Optional[str], side: str, **kwargs) -> float:

        dataframe, last_updated = self.dp.get_analyzed_dataframe(pair=pair, timeframe=self.timeframe)
        new_entryprice = dataframe['bollinger_10_lowerband'].iat[-1]

        return new_entryprice

    def custom_exit_price(self, pair: str, trade: Trade,
                          current_time: datetime, proposed_rate: float,
                          current_profit: float, exit_tag: Optional[str], **kwargs) -> float:

        dataframe, last_updated = self.dp.get_analyzed_dataframe(pair=pair, timeframe=self.timeframe)
        new_exitprice = dataframe['bollinger_10_upperband'].iat[-1]

        return new_exitprice

```

这些函数允许您基于策略内的某些指标或条件设置入场和离场价格。custom_entry_price() 函数在放置入场订单之前调用，而 custom_exit_price() 在放置离场订单之前使用。

**注意：** 修改入场和离场价格会影响限价订单，并可能导致未成交的订单。默认情况下，当前价格和自定义价格之间有一个最大允许的距离，可以在配置中进行调整（custom_price_max_distance_ratio）。此外，请记住自定义价格在回测中受支持，订单将在蜡烛的低/高范围内成交。

在设置自定义入场和离场价格时，请仔细考虑价格范围和策略条件，以避免订单执行可能出现的问题。



## 自定义的订单超时规则 Custom order timeout rules

自定义的订单超时规则可以通过策略或在配置文件的 unfilledtimeout 部分进行简单的基于时间的设置。

除此之外，Freqtrade还提供了用于两种订单类型的自定义回调，允许您根据自定义条件判断订单是否超时。

**注意：** 在回测中，如果订单价格落在蜡烛的最高/最低范围内，订单将被认为已填充。下面的回调函数将针对未立即填充的订单（使用自定义定价）每个（细节）蜡烛调用一次。


### Custom order timeout example

以下是一个简单的示例，根据资产价格的不同应用不同的未填充超时时间。对于更高价格的资产，它应用较短的超时时间，而对于便宜的资产，则允许更长时间来填充。

该函数必须返回 True（取消订单）或 False（保持订单有效）。

```python
from datetime import datetime, timedelta
from freqtrade.persistence import Trade, Order

class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    # 设置 unfilledtimeout 为 25 小时，因为下面的最大超时时间是 24 小时。
    unfilledtimeout = {
        'entry': 60 * 25,
        'exit': 60 * 25
    }

    def check_entry_timeout(self, pair: str, trade: 'Trade', order: 'Order',
                            current_time: datetime, **kwargs) -> bool:
        if trade.open_rate > 100 and trade.open_date_utc < current_time - timedelta(minutes=5):
            return True
        elif trade.open_rate > 10 and trade.open_date_utc < current_time - timedelta(minutes=3):
            return True
        elif trade.open_rate < 1 and trade.open_date_utc < current_time - timedelta(hours=24):
           return True
        return False


    def check_exit_timeout(self, pair: str, trade: Trade, order: 'Order',
                           current_time: datetime, **kwargs) -> bool:
        if trade.open_rate > 100 and trade.open_date_utc < current_time - timedelta(minutes=5):
            return True
        elif trade.open_rate > 10 and trade.open_date_utc < current_time - timedelta(minutes=3):
            return True
        elif trade.open_rate < 1 and trade.open_date_utc < current_time - timedelta(hours=24):
           return True
        return False



```
注意：对于上述示例，unfilledtimeout 必须设置为大于 24 小时的值，否则该类型的超时时间将首先应用。

### Custom order timeout example (using additional data)

以下是**使用额外数据的自定义订单超时**示例。这个例子中，check_entry_timeout 方法用于交易进入时的订单超时，check_exit_timeout 方法用于交易退出时的订单超时。

```python
from datetime import datetime
from freqtrade.persistence import Trade, Order

class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    # Set unfilledtimeout to 25 hours, since the maximum timeout from below is 24 hours.
    unfilledtimeout = {
        'entry': 60 * 25,
        'exit': 60 * 25
    }

    def check_entry_timeout(self, pair: str, trade: 'Trade', order: 'Order',
                            current_time: datetime, **kwargs) -> bool:
        ob = self.dp.orderbook(pair, 1)
        current_price = ob['bids'][0][0]
        # 如果价格比订单高出 2%，取消买单。
        if current_price > order.price * 1.02:
            return True
        return False


    def check_exit_timeout(self, pair: str, trade: 'Trade', order: 'Order',
                           current_time: datetime, **kwargs) -> bool:
        ob = self.dp.orderbook(pair, 1)
        current_price = ob['asks'][0][0]
        # 如果价格比订单低出 2%，取消卖单。
        if current_price < order.price * 0.98:
            return True
        return False

```
这个示例中，如果买单价格比当前价格高出 2%，则会取消买单。如果卖单价格比当前价格低出 2%，则会取消卖单。

## 订单确认 Bot order confirmation

确认交易入场/出场。 这是下订单之前调用的最后一个方法。

### 确认 入场/买单 Trade entry (buy order) confirmation

这是一个confirm_trade_entry()方法的示例。在这个方法中，您可以根据自定义条件来决定是否确认交易的进入（买入）。
```python
class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    def confirm_trade_entry(self, pair: str, order_type: str, amount: float, rate: float,
                            time_in_force: str, current_time: datetime, entry_tag: Optional[str],
                            side: str, **kwargs) -> bool:
        """
        Called right before placing a entry order.
        Timing for this function is critical, so avoid doing heavy computations or
        network requests in this method.

        For full documentation please go to https://www.freqtrade.io/en/latest/strategy-advanced/

        When not implemented by a strategy, returns True (always confirming).

        :param pair: Pair that's about to be bought/shorted.
        :param order_type: Order type (as configured in order_types). usually limit or market.
        :param amount: Amount in target (base) currency that's going to be traded.
        :param rate: Rate that's going to be used when using limit orders 
                     or current rate for market orders.
        :param time_in_force: Time in force. Defaults to GTC (Good-til-cancelled).
        :param current_time: datetime object, containing the current datetime
        :param entry_tag: Optional entry_tag (buy_tag) if provided with the buy signal.
        :param side: 'long' or 'short' - indicating the direction of the proposed trade
        :param **kwargs: Ensure to keep this here so updates to this won't break your strategy.
        :return bool: When True is returned, then the buy-order is placed on the exchange.
            False aborts the process
        """
        # 在此处添加您的自定义逻辑以确认或取消交易进入
        # 返回 True 以确认交易，返回 False 以取消交易
        return True

```

在confirm_trade_entry()中，您可以使用pair（交易对）、amount（数量）、rate（价格）、current_time（当前时间）等参数来制定确认交易进入的逻辑。当返回True时，买单将被放置在交易所上进行交易，而返回False则会取消这个买单。

### 确认 出场/卖出 Trade exit (sell order) confirmation

confirm_trade_exit()方法允许您在最后一刻中止交易退出（卖出），可能是因为价格不符合预期。如果不同的退出原因适用，confirm_trade_exit()可能在同一次迭代中被多次调用。这些退出原因按照以下顺序适用：

- 退出信号（exit_signal）/ 自定义退出（custom_exit）
- 止损（stop_loss）
- 投资回报率（ROI）
- 移动止损（trailing_stop_loss）

您可以利用confirm_trade_exit()在每个退出原因被考虑之前进行适当的控制和逻辑处理。


在下面这段代码中，confirm_trade_exit()方法被用于决定是否执行交易退出。在这个例子中，当退出原因是force_exit（强制退出）且交易计算的利润比率小于0时，返回False以取消这次交易退出。这样的设计可以让您根据自定义条件来控制是否执行强制退出。
```python
from freqtrade.persistence import Trade


class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    def confirm_trade_exit(self, pair: str, trade: Trade, order_type: str, amount: float,
                           rate: float, time_in_force: str, exit_reason: str,
                           current_time: datetime, **kwargs) -> bool:
        """
        Called right before placing a regular exit order.
        Timing for this function is critical, so avoid doing heavy computations or
        network requests in this method.

        For full documentation please go to https://www.freqtrade.io/en/latest/strategy-advanced/

        When not implemented by a strategy, returns True (always confirming).

        :param pair: Pair for trade that's about to be exited.
        :param trade: trade object.
        :param order_type: Order type (as configured in order_types). usually limit or market.
        :param amount: Amount in base currency.
        :param rate: Rate that's going to be used when using limit orders
                     or current rate for market orders.
        :param time_in_force: Time in force. Defaults to GTC (Good-til-cancelled).
        :param exit_reason: Exit reason.
            Can be any of ['roi', 'stop_loss', 'stoploss_on_exchange', 'trailing_stop_loss',
                           'exit_signal', 'force_exit', 'emergency_exit']
        :param current_time: datetime object, containing the current datetime
        :param **kwargs: Ensure to keep this here so updates to this won't break your strategy.
        :return bool: When True, then the exit-order is placed on the exchange.
            False aborts the process
        """
        if exit_reason == 'force_exit' and trade.calc_profit_ratio(rate) < 0:
            # Reject force-sells with negative profit
            # This is just a sample, please adjust to your needs
            # (this does not necessarily make sense, assuming you know when you're force-selling)
            return False
        return True

```


## Adjust trade position

adjust_trade_position() 函数是一种强大的工具，通过设置 position_adjustment_enable 策略属性启用。它允许您管理和控制交易仓位，根据特定条件执行额外的订单。该函数用于各种目的，例如分仓（Dollar Cost Averaging，DCA）或调整仓位大小。

以下是关于 adjust_trade_position() 的一些关键要点：

- 目的： 用于在交易过程中执行额外的订单，主要是为了管理风险或调整仓位。
- 回调激活： 仅在没有等待执行的挂单（买入或卖出）时调用。
- 执行频率： 在交易的整个过程中频繁调用，要求实现高效。
- 仓位调整： 始终以交易方向执行。正值增加仓位；负值减少仓位。
- 下单金额： 返回的下单金额（以交易货币表示）应在指定范围内（min_stake 到 max_stake）。
- 最大入场调整： 由 max_entry_position_adjustment 控制，限制每笔交易的额外买入次数。
- 止损考虑： 止损是根据初始开仓价格而不是平均价格计算的。
- 钱包资金： 在进行分仓时要注意钱包资金。使用 DCA 订单的“无限”下单金额需要实现 custom_stake_amount() 以有效管理资金。
- 回测注意事项： 在回测中，此函数对每根K线都进行调用，可能影响运行时性能，并且由于调整的频率不同，可能导致与实时交易不同的结果。

在调用频繁的情况下，务必制定高效且准确的 adjust_trade_position() 实现。


```python
from freqtrade.persistence import Trade


class DigDeeperStrategy(IStrategy):

    position_adjustment_enable = True

    # Attempts to handle large drops with DCA. High stoploss is required.
    stoploss = -0.30

    # ... populate_* methods

    # Example specific variables
    max_entry_position_adjustment = 3
    # This number is explained a bit further down
    max_dca_multiplier = 5.5

    # This is called when placing the initial order (opening trade)
    def custom_stake_amount(self, pair: str, current_time: datetime, current_rate: float,
                            proposed_stake: float, min_stake: Optional[float], max_stake: float,
                            leverage: float, entry_tag: Optional[str], side: str,
                            **kwargs) -> float:

        # We need to leave most of the funds for possible further DCA orders
        # This also applies to fixed stakes
        return proposed_stake / self.max_dca_multiplier

    def adjust_trade_position(self, trade: Trade, current_time: datetime,
                              current_rate: float, current_profit: float,
                              min_stake: Optional[float], max_stake: float,
                              current_entry_rate: float, current_exit_rate: float,
                              current_entry_profit: float, current_exit_profit: float,
                              **kwargs) -> Optional[float]:
        """
        Custom trade adjustment logic, returning the stake amount that a trade should be
        increased or decreased.
        This means extra entry or exit orders with additional fees.
        Only called when `position_adjustment_enable` is set to True.

        For full documentation please go to https://www.freqtrade.io/en/latest/strategy-advanced/

        When not implemented by a strategy, returns None

        :param trade: trade object.
        :param current_time: datetime object, containing the current datetime
        :param current_rate: Current entry rate (same as current_entry_profit)
        :param current_profit: Current profit (as ratio), calculated based on current_rate 
                               (same as current_entry_profit).
        :param min_stake: Minimal stake size allowed by exchange (for both entries and exits)
        :param max_stake: Maximum stake allowed (either through balance, or by exchange limits).
        :param current_entry_rate: Current rate using entry pricing.
        :param current_exit_rate: Current rate using exit pricing.
        :param current_entry_profit: Current profit using entry pricing.
        :param current_exit_profit: Current profit using exit pricing.
        :param **kwargs: Ensure to keep this here so updates to this won't break your strategy.
        :return float: Stake amount to adjust your trade,
                       Positive values to increase position, Negative values to decrease position.
                       Return None for no action.
        """

        if current_profit > 0.05 and trade.nr_of_successful_exits == 0:
            # Take half of the profit at +5%
            return -(trade.stake_amount / 2)

        if current_profit > -0.05:
            return None

        # Obtain pair dataframe (just to show how to access it)
        dataframe, _ = self.dp.get_analyzed_dataframe(trade.pair, self.timeframe)
        # Only buy when not actively falling price.
        last_candle = dataframe.iloc[-1].squeeze()
        previous_candle = dataframe.iloc[-2].squeeze()
        if last_candle['close'] < previous_candle['close']:
            return None

        filled_entries = trade.select_filled_orders(trade.entry_side)
        count_of_entries = trade.nr_of_successful_entries
        # Allow up to 3 additional increasingly larger buys (4 in total)
        # Initial buy is 1x
        # If that falls to -5% profit, we buy 1.25x more, average profit should increase to roughly -2.2%
        # If that falls down to -5% again, we buy 1.5x more
        # If that falls once again down to -5%, we buy 1.75x more
        # Total stake for this trade would be 1 + 1.25 + 1.5 + 1.75 = 5.5x of the initial allowed stake.
        # That is why max_dca_multiplier is 5.5
        # Hope you have a deep wallet!
        try:
            # This returns first order stake size
            stake_amount = filled_entries[0].stake_amount
            # This then calculates current safety order size
            stake_amount = stake_amount * (1 + (count_of_entries * 0.25))
            return stake_amount
        except Exception as exception:
            return None

        return None

```

这段代码演示了如何使用 `adjust_trade_position()` 方法来在交易过程中管理仓位大小。让我一步步解释：

1. `position_adjustment_enable = True`：这个设置启用了仓位调整功能。

2. `stoploss = -0.30`：设置了止损。当交易的利润下降到 -30% 时，可能会触发额外的买入来尝试处理这种大幅下跌的情况。

3. `custom_stake_amount()` 函数：这个函数确定了初始订单的下单金额（开仓交易）。这个例子中，它返回了 `proposed_stake / self.max_dca_multiplier`，意味着它会留出足够的资金来执行可能的分仓交易。

4. `adjust_trade_position()` 函数：这个函数根据当前的交易情况尝试调整仓位大小。主要的逻辑是：

   - 首先检查当前的利润情况：
     - 如果利润超过 5%，尝试卖出一半的头寸。
     - 如果利润在 -5% 到 5% 之间，则尝试不进行任何操作。
     - 如果利润低于 -5%，则尝试根据过去成功买入订单的数量增加下一个订单的金额。
   
   - 通过获取填充的买入订单，计算当前的安全订单（DCA）大小。下一个订单的大小会逐渐增加，从而在利润下降时增加头寸。

这样的策略尝试根据不同的利润情况调整仓位大小，以便在交易遇到不同利润状况时，通过额外的买入或卖出来管理交易风险。

## Position adjust calculations


1. **入场价格的计算**：入场价格是使用加权平均值计算的。也就是说，如果你进行了多次买入操作，每次买入都是以不同的价格完成的，最后的入场价格将是这些价格的加权平均值。

2. **出场不影响入场价格**：卖出操作不会影响入场价格的计算，无论你何时卖出。

3. **部分卖出的相对利润**：部分卖出的相对利润是相对于此时的平均入场价格来计算的。也就是说，如果你部分卖出头寸，盈亏将与该部分头寸的平均购买价格相关。

4. **最终出场的相对利润**：最终出场的相对利润是基于你的总投资资本计算的。这个相对利润是根据你所有买入操作的总投资资本计算得出的，而不仅仅是最后一次卖出所得到的利润。

下面这个例子很好地演示了仓位调整的计算方式。让我一步步解释：


1. 假设你持有一个虚拟币的多头头寸。

2. 交易流程如下：
   - 买入 100 枚虚拟币，价格为 $8；
   - 买入 100 枚虚拟币，价格为 $9。这时的平均购入价为 $8.5；
   - 卖出 100 枚虚拟币，价格为 $10。这时的平均购入价仍为 $8.5。实现的利润为 $150，利润率为 17.65%；
   - 买入 150 枚虚拟币，价格为 $11。这时的平均购入价为 $10。实现的利润为 $150，利润率为 17.65%；
   - 卖出 100 枚虚拟币，价格为 $12。这时的平均购入价仍为 $10。总实现利润为 $350，利润率为 20%；
   - 最后，卖出 150 枚虚拟币，价格为 $14。这时的平均购入价仍为 $10。总实现利润为 $950，利润率为 40%。这将是最后一个 "Exit" 消息。

3. 整个交易的总利润为 $950，而你的总投资为 $3350（分别购入 100 枚虚拟币、100 枚虚拟币、150 枚虚拟币），因此最终的相对利润率为 28.35%（950 / 3350）。这个数值表示你相对于初始投资获得的利润百分比。

这个示例展示了如何计算多个买入和卖出操作的利润，以及如何计算最终的相对利润率，这个计算方法能更好地反映整个交易的盈利情况。



## Adjust Entry Price

`adjust_entry_price()`回调函数允许策略开发者在新的K线出现时刷新或替换限价订单。需要注意的是，`custom_entry_price()`决定了触发入场时的初始入场限价订单价格目标。

此回调允许通过返回`None`取消订单。返回`current_order_rate`会保持订单在交易所中原样存在。如果返回其他价格，原有订单将被取消，并用更新后的价格放置新订单。

要注意交易的开启日期（`trade.open_date_utc`）将保持为第一个订单下单时的时间。请确保意识到这一点，并在其他回调中调整逻辑，以便使用第一个成交订单的日期。

如果原始订单的取消失败，则订单不会被替换，但很可能已在交易所取消。对于初始入场订单，这将导致订单被删除，而对于位置调整订单，则保持交易规模不变。

请注意，入场未成交订单的常规超时（以及`check_entry_timeout()`）优先于此回调。通过上述方法取消的入场订单不会触发此回调。确保超时值符合您的预期。

```python
from freqtrade.persistence import Trade
from datetime import timedelta

class AwesomeStrategy(IStrategy):

    # ... populate_* methods

    def adjust_entry_price(self, trade: Trade, order: Optional[Order], pair: str,
                           current_time: datetime, proposed_rate: float, current_order_rate: float,
                           entry_tag: Optional[str], side: str, **kwargs) -> float:
        """
        Entry price re-adjustment logic, returning the user desired limit price.
        This only executes when a order was already placed, still open (unfilled fully or partially)
        and not timed out on subsequent candles after entry trigger.

        When not implemented by a strategy, returns current_order_rate as default.
        If current_order_rate is returned then the existing order is maintained.
        If None is returned then order gets canceled but not replaced by a new one.

        :param pair: Pair that's currently analyzed
        :param trade: Trade object.
        :param order: Order object
        :param current_time: datetime object, containing the current datetime
        :param proposed_rate: Rate, calculated based on pricing settings in entry_pricing.
        :param current_order_rate: Rate of the existing order in place.
        :param entry_tag: Optional entry_tag (buy_tag) if provided with the buy signal.
        :param side: 'long' or 'short' - indicating the direction of the proposed trade
        :param **kwargs: Ensure to keep this here so updates to this won't break your strategy.
        :return float: New entry price value if provided

        """
        # Limit orders to use and follow SMA200 as price target for the first 10 minutes since entry trigger for BTC/USDT pair.
        if pair == 'BTC/USDT' and entry_tag == 'long_sma200' and side == 'long' and (current_time - timedelta(minutes=10)) > trade.open_date_utc:
            # just cancel the order if it has been filled more than half of the amount
            if order.filled > order.remaining:
                return None
            else:
                dataframe, _ = self.dp.get_analyzed_dataframe(pair=pair, timeframe=self.timeframe)
                current_candle = dataframe.iloc[-1].squeeze()
                # desired price
                return current_candle['sma_200']
        # default: maintain existing order
        return current_order_rate

```

## Leverage Callback

这个回调函数用于确定交易中所需的杠杆倍数，默认为1（无杠杆）。

假设资本为500USDT，使用杠杆倍数为3的交易将导致持仓价值为500 x 3 = 1500 USDT。

如果设置的杠杆倍数超过了最大杠杆倍数（max_leverage），则会调整为最大杠杆倍数。对于不支持杠杆的市场或交易所，此方法将被忽略。

```python
class AwesomeStrategy(IStrategy):
    def leverage(self, pair: str, current_time: datetime, current_rate: float,
                 proposed_leverage: float, max_leverage: float, entry_tag: Optional[str], side: str,
                 **kwargs) -> float:
        """
        Customize leverage for each new trade. This method is only called in futures mode.

        :param pair: Pair that's currently analyzed
        :param current_time: datetime object, containing the current datetime
        :param current_rate: Rate, calculated based on pricing settings in exit_pricing.
        :param proposed_leverage: A leverage proposed by the bot.
        :param max_leverage: Max leverage allowed on this pair
        :param entry_tag: Optional entry_tag (buy_tag) if provided with the buy signal.
        :param side: 'long' or 'short' - indicating the direction of the proposed trade
        :return: A leverage amount, which is between 1.0 and max_leverage.
        """
        return 1.0

```

**所有利润计算都包括杠杆。止损/投资回报率（ROI）的计算也包括杠杆。以10倍杠杆定义的10%止损会在价格下跌1%时触发止损。**












































