
freqtrade 有一些可用的策略：https://github.com/freqtrade/freqtrade-strategies

通过模版创建一个新的策略
```shell
freqtrade new-strategy --strategy AwesomeStrategy
```

### 策略解析
一个策略文件应该包含所有需要构建的策略信息
- 指标
- 进入规则
- 退出规则
- 最低投资回报率 `minimal ROI`
- 止损 `stoploss`

**注意：** 避免使用 `df.iloc[-1]` 应该使用 `df.shift()` 去获取上一根蜡烛

### Dataframe
Freqtrade 使用 pandas 来储存和提供 蜡烛 candlestic(OHLCV) 数据，每一行代表一个蜡烛的图表数据，最新的蜡烛始终是数据框中的最后一个（按日期排序）。

```shell
> dataframe.head()
                       date      open      high       low     close     volume
0 2021-11-09 23:25:00+00:00  67279.67  67321.84  67255.01  67300.97   44.62253
1 2021-11-09 23:30:00+00:00  67300.97  67301.34  67183.03  67187.01   61.38076
2 2021-11-09 23:35:00+00:00  67187.02  67187.02  67031.93  67123.81  113.42728
3 2021-11-09 23:40:00+00:00  67123.80  67222.40  67080.33  67160.48   78.96008
4 2021-11-09 23:45:00+00:00  67160.48  67160.48  66901.26  66943.37  111.39292

```

pandas 提供了快速的方法来计算指标，建议不使用循环，使用矢量方法来计算

dataframe 作为一个表，必须以pandas兼容的方式来写
```shell
dataframe.loc[
  (dataframe['rsi'] > 30), 'enter_long' = 1
]
```
上面这种方式会生成新的一列数据，当 RSI 值大于30 时，该列的值为1

### 自定义指标
买卖信号需要指标。通过扩展策略文件中 `populate_indicators()` 方法中包含的列表来添加更多指标。

您可以通过扩展策略文件中 `populate_indicators()` 方法中包含的列表来添加更多指标。

您应该只添加在 `populate_entry_trend()、populate_exit_trend()` 中使用的指标，或填充另一个指标，否则性能可能会受到影响。

始终只返回数据帧而不删除/修改列“open”、“high”、“low”、“close”、“volume”，否则这些字段将包含意外的内容。

Sample:

```python
def populate_indicators(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
    """
    Adds several different TA indicators to the given DataFrame

    Performance Note: For the best performance be frugal on the number of indicators
    you are using. Let uncomment only the indicator you are using in your strategies
    or your hyperopt configuration, otherwise you will waste your memory and CPU usage.
    :param dataframe: Dataframe with data from the exchange
    :param metadata: Additional information, like the currently traded pair
    :return: a Dataframe with all mandatory indicators for the strategies
    """
    dataframe['sar'] = ta.SAR(dataframe)
    dataframe['adx'] = ta.ADX(dataframe)
    stoch = ta.STOCHF(dataframe)
    dataframe['fastd'] = stoch['fastd']
    dataframe['fastk'] = stoch['fastk']
    dataframe['blower'] = ta.BBANDS(dataframe, nbdevup=2, nbdevdn=2)['lowerband']
    dataframe['sma'] = ta.SMA(dataframe, timeperiod=40)
    dataframe['tema'] = ta.TEMA(dataframe, timeperiod=9)
    dataframe['mfi'] = ta.MFI(dataframe)
    dataframe['rsi'] = ta.RSI(dataframe)
    dataframe['ema5'] = ta.EMA(dataframe, timeperiod=5)
    dataframe['ema10'] = ta.EMA(dataframe, timeperiod=10)
    dataframe['ema50'] = ta.EMA(dataframe, timeperiod=50)
    dataframe['ema100'] = ta.EMA(dataframe, timeperiod=100)
    dataframe['ao'] = awesome_oscillator(dataframe)
    macd = ta.MACD(dataframe)
    dataframe['macd'] = macd['macd']
    dataframe['macdsignal'] = macd['macdsignal']
    dataframe['macdhist'] = macd['macdhist']
    hilbert = ta.HT_SINE(dataframe)
    dataframe['htsine'] = hilbert['sine']
    dataframe['htleadsine'] = hilbert['leadsine']
    dataframe['plus_dm'] = ta.PLUS_DM(dataframe)
    dataframe['plus_di'] = ta.PLUS_DI(dataframe)
    dataframe['minus_dm'] = ta.MINUS_DM(dataframe)
    dataframe['minus_di'] = ta.MINUS_DI(dataframe)
    return dataframe

```

#### 指标库
- ta-lib
- pandas-ta
- technical

#### 策略启动期 （startup period）
在量化交易中，许多指标在启动期间可能不稳定，在这段时间内可能会出现不可用（NaN）或者计算不正确的情况。这可能导致交易不一致性，因为Freqtrade无法确定这个不稳定期应该有多长时间。为了解决这个问题，策略可以分配一个叫做 `startup_candle_count` 的属性。

`startup_candle_count ` 表示策略需要的最大蜡烛数量（或者说数据历史周期），用来计算稳定的指标。假设用户使用了更高时间周期的信息性交易对，startup_candle_count 并不一定会改变。这个数值表示了任何一个信息性时间周期需要计算稳定指标所需的最大周期（以蜡烛数量计算）。

在这个例子中，假设有一个策略，需要设置 startup_candle_count 为 400（startup_candle_count = 400）。这是因为对于计算 ema100 指标，确保值正确所需的最小历史数据为 400 蜡烛周期。
```python
dataframe['ema100'] = ta.EMA(dataframe, timeperiod=400) 
```
通过指定 startup_candle_count，让交易机器人知道需要多少历史数据，可以确保在回测和超参数优化（hyperopt）时，交易开始的时间范围符合指定的计算要求。这样有助于避免在启动期间由于指标不稳定而产生的不准确的交易信号或结果。

如果收到这样的警告：`WARNING - Using 3 calls to get OHLCV. This can result in slower operations for the bot. Please check if you really need 1500 candles for your strategy `
它在提醒你可能过多地获取了历史数据，这可能会导致交易机器人的操作变慢。通常，这个警告会提示你思考是否真的需要如此多的历史数据来支撑你的交易策略。

这个警告的背后意思是，如果你设置了较大的历史数据窗口，Freqtrade可能会进行多次调用来获取同一交易对的数据，这会导致网络请求频繁，进而使得操作速度变慢。Freqtrade在刷新K线数据时可能会花费更长的时间，尤其是在大量调用情况下。

为了避免这种情况，Freqtrade在一定程度上限制了对同一个交易对的数据获取次数。通常这个限制是5次，以避免对交易所造成过多请求，同时也是为了避免Freqtrade变得过于缓慢。

建议在设置历史数据窗口时，仔细考虑实际需要的数据量。如果可能的话，**尝试减少历史数据窗口的大小，以降低获取数据的频率，提高Freqtrade的运行效率。这样可以避免因为过多历史数据而导致的性能下降。**

**注意：** startup_candle_count 是策略需要的历史数据蜡烛数量，而 ohlcv_candle_limit 是限制每次获取历史K线数据（OHLCV数据）的数量，通常为500根蜡烛。
在设置 startup_candle_count 时，需要确保它不超过 ohlcv_candle_limit 的五倍。因为在Dry-Run（模拟交易）或者Live Trade（实际交易）操作中，交易机器人只能使用这个数量的蜡烛数据。

举例来说，如果 ohlcv_candle_limit 是 500，那么根据文档建议，最好将 startup_candle_count 控制在 500 * 5 = 2500根蜡烛以下。这样可以确保你的策略在实际操作中能够利用到足够的历史数据进行交易，而不会超出Freqtrade在实际操作时的限制。

在设置这两个参数时，要注意平衡需要足够的历史数据来支撑策略的准确性，同时也要考虑交易机器人实际运行时的限制，以确保策略能够在实际交易中正常执行。

#### Example
回测一个月（January 2019） 5m 的蜡烛 使用 example 策略 EMA100指标
```shell
freqtrade backtesting --timerange 20190101-20190201 --timeframe 5m
```
在这个例子中，假设 startup_candle_count 被设置为 400。这意味着在回测时，Freqtrade知道需要400根蜡烛数据来生成有效的买入信号。它会加载从2019年1月1日开始往前推400根5分钟K线数据，大约会加载到2018年12月30日11点40分左右的数据。如果这些数据可用，指标将在这个扩展的时间范围内进行计算。

然而，在开始实际回测之前，由于指标可能在起始时段（例如2019年1月1日00:00:00之前）不稳定，Freqtrade会移除这个不稳定的起始期间。因此，在进行回测时，**实际使用的数据将会是从稳定的起始点（大约2018年12月30日11点40分）到回测结束时间（2019年2月1日）之间的数据。**


### 买入信号规则
下面示例展示了如何在自定义策略中编写 populate_entry_trend() 方法来定义买入信号的规则。

这个方法用于根据技术指标生成买入信号，并在数据框中添加一个名为 "enter_long" 的列（对于做空策略来说是 "enter_short" 列），这个列的值应为 1 表示买入信号，0 表示无操作。

Sample from user_data/strategies/sample_strategy.py:
```python
def populate_entry_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
    """
    Based on TA indicators, populates the buy signal for the given dataframe
    :param dataframe: DataFrame populated with indicators
    :param metadata: Additional information, like the currently traded pair
    :return: DataFrame with buy column
    """
    dataframe.loc[
        (
            (qtpylib.crossed_above(dataframe['rsi'], 30)) &  # Signal: RSI crosses above 30
            (dataframe['tema'] <= dataframe['bb_middleband']) &  # Guard
            (dataframe['tema'] > dataframe['tema'].shift(1)) &  # Guard
            (dataframe['volume'] > 0)  # Make sure Volume is not 0
        ),
        ['enter_long', 'enter_tag']] = (1, 'rsi_cross')

    return dataframe
```

在这个示例中，买入信号的规则包括：

- 当 RSI 指标上穿过 30（qtpylib.crossed_above(dataframe['rsi'], 30)）；
- TEMA（三重指数移动平均线）在布林带中线之下（dataframe['tema'] <= dataframe['bb_middleband']）；
- TEMA 比前一周期的值要大（dataframe['tema'] > dataframe['tema'].shift(1)）；
- 确保成交量不为零（dataframe['volume'] > 0）。

如果上述条件满足，就会在 "enter_long" 和 "enter_tag" 列中标记买入信号为 1，并附加一个标签 "rsi_cross"。

值得注意的是，最后返回的数据框应当包含新增的 "enter_long" 列，并保持原始的 "open", "high", "low", "close", "volume" 列不被修改，以确保这些列包含预期的数据。

### 卖出规则 
示例展示了如何在自定义策略中编写 populate_exit_trend() 方法来定义卖出（退出）信号的规则。

Sample from user_data/strategies/sample_strategy.py:
```python
def populate_exit_trend(self, dataframe: DataFrame, metadata: dict) -> DataFrame:
    """
    Based on TA indicators, populates the exit signal for the given dataframe
    :param dataframe: DataFrame populated with indicators
    :param metadata: Additional information, like the currently traded pair
    :return: DataFrame with buy column
    """
    dataframe.loc[
        (
            (qtpylib.crossed_above(dataframe['rsi'], 70)) &  # Signal: RSI crosses above 70
            (dataframe['tema'] > dataframe['bb_middleband']) &  # Guard
            (dataframe['tema'] < dataframe['tema'].shift(1)) &  # Guard
            (dataframe['volume'] > 0)  # Make sure Volume is not 0
        ),
        ['exit_long', 'exit_tag']] = (1, 'rsi_too_high')
    return dataframe
```
在这个示例中，卖出信号的规则包括：

- 当 RSI 指标上穿过 70（qtpylib.crossed_above(dataframe['rsi'], 70)）；
- TEMA（三重指数移动平均线）在布林带中线之上（dataframe['tema'] > dataframe['bb_middleband']）；
- TEMA 比前一周期的值要小（dataframe['tema'] < dataframe['tema'].shift(1)）；
- 确保成交量不为零（dataframe['volume'] > 0）。

如果上述条件满足，就会在 "exit_long" 和 "exit_tag" 列中标记卖出信号为 1，并附加一个标签 "rsi_too_high"。

这个方法的目的是为了根据定义的规则生成卖出信号，以便在实际交易中能够根据这些规则进行退出。最后，返回的数据框应包含新增的 "exit_long" 列，并保持原始的 "open", "high", "low", "close", "volume" 列不被修改。

### 最小投资回报 minimal roi
 `minimal_roi` 字典定义了交易达到特定收益率后应该退出的规则，与退出信号独立。它按照交易开始后经过的时间（以分钟为单位）来定义不同的收益率阈值。
```python
minimal_roi = {
    "40": 0.0,
    "30": 0.01,
    "20": 0.02,
    "0": 0.04
}
```
- 当交易达到 4% 的利润时退出（从交易开始算起）；
- 当交易达到 2% 的利润时退出（经过20分钟后生效）；
- 当交易达到 1% 的利润时退出（经过30分钟后生效）；
- 当交易达到不亏损状态时退出（经过40分钟后生效）。

这个配置**考虑了交易费用**的计算。如果你希望完全禁用ROI（收益率）策略，则将其设置为空字典 {}。
```python
minimal_roi = {}
```

此外，下面示例代码中展示了如何根据时间周期（timeframe）设置交易策略中的ROI。通过使用 `timeframe_to_minutes`函数，可以将时间周期转换为分钟数，这样你可以根据不同的时间周期来设置不同的ROI退出时间点。

```python
from freqtrade.exchange import timeframe_to_minutes

class AwesomeStrategy(IStrategy):

    timeframe = "1d"
    timeframe_mins = timeframe_to_minutes(timeframe)
    minimal_roi = {
        "0": 0.05,                      # 5% for the first 3 candles
        str(timeframe_mins * 3): 0.02,  # 2% after 3 candles
        str(timeframe_mins * 6): 0.01,  # 1% After 6 candles
    }
```


### 止损 Stoploss

设置止损对于保护你的资本免受市场大幅逆向波动是非常重要的。
Sample of setting a 10% stoploss:
```python
stoploss = -0.10
```
在这个示例中，设置了一个10%的止损。通常情况下，止损是通过一个负数来定义的，表示在价格下跌到一定百分比时需要止损出场。

在这个例子中，stoploss = -0.10 表示当交易亏损达到10%时触发止损，从而退出这个交易。

值得注意的是，止损的设置对于资本保护和风险控制至关重要。它可以帮助避免大幅度的亏损，并保护你的资金免受极端市场波动的影响。更多关于止损功能的详细文档可以查看专门的[止损页面](https://www.freqtrade.io/en/stable/stoploss/)。

### Timeframe

时间周期（Timeframe）是指交易机器人应该下载和用于分析的K线数据集合。常见的时间周期包括"1m"（1分钟）、"5m"（5分钟）、"15m"（15分钟）、"1h"（1小时），但你的交易所支持的所有时间周期都应该可以使用。

值得注意的是，相同的入场/出场信号可能在一个时间周期下运作良好，但在其他时间周期下可能效果不佳。不同的时间周期可能导致信号产生、趋势分析和结果的差异。

在策略方法中，你可以通过 self.timeframe 属性访问时间周期设置。这意味着你可以根据策略的需要，针对不同的时间周期设置不同的逻辑和参数，以更好地适应市场变化和波动。


### 做空 Can short
“Can short” 是指在期货/合约市场中使用做空信号。如果你希望在期货市场中使用做空信号，你需要设置 can_short=True。

但需要注意的是，启用了做空信号的策略可能在现货市场上无法加载。如果你禁用了做空信号，策略将忽略做空信号（即使在期货市场中也是如此）。

因此，通过设置 can_short=True，你可以在期货市场中启用策略的做空信号，但需要注意这可能会限制你在现货市场中使用这个策略。这个设置可以根据你的交易需求和市场选择来调整，以确保你的策略能够在合适的市场条件下发挥作用。

### Metadata dic
metadata 字典包含了额外的信息，在 populate_entry_trend、populate_exit_trend 和 populate_indicators 这些方法中都可以使用。目前，这个字典中包含了一个名为 pair 的信息，可以通过 metadata['pair'] 来访问，它会以 XRP/BTC 这样的格式返回一个交易对。

重要的是要注意，metadata 字典不应该被修改，而且它在多次调用中不会持久化信息。如果你需要跨多次调用保存信息，可以参考[存储信息](https://www.freqtrade.io/en/stable/strategy-advanced/#Storing-information)部分的方法。

这个 metadata 字典提供了一些关于当前交易对的信息，可以在你的策略中根据这些信息来做出决策或执行特定的操作。

### Strategy file loading
Freqtrade默认会尝试从 user_data/strategies 目录下的所有 .py 文件中加载策略。

假设你的策略叫做 AwesomeStrategy，存储在文件 user_data/strategies/AwesomeStrategy.py 中，你可以通过以下命令来启动 freqtrade：freqtrade trade --strategy AwesomeStrategy。需要注意的是，这里使用的是类名，而不是文件名。

你可以使用 freqtrade list-strategies 命令来查看Freqtrade能够加载的所有策略列表（即在正确文件夹下的所有策略文件）。这个命令会列出所有策略，并包括一个 "status" 字段，用于指示可能存在的问题。这样可以帮助你检查并确保你的策略文件是否可以被Freqtrade正确加载和识别。

### Informative Pairs

“Informative Pairs” 是指获取非交易对（参考对）的数据。对于某些策略来说，获取这些额外的交易对的OHLCV数据可能会很有益处。这些参考对的OHLCV数据会作为常规白名单刷新过程的一部分而被下载，并且可以通过 DataProvider 来访问，就像其他交易对的数据一样。但是，除非它们也被指定在交易对白名单中，或者被动态白名单选择到，否则这些参考对将不会被交易。

这些参考对需要以元组的形式指定，格式为 ("交易对", "时间周期")，交易对作为第一个参数，时间周期作为第二个参数。

举例：
```python
def informative_pairs(self):
    return [("ETH/USDT", "5m"),
            ("BTC/TUSD", "15m"),
            ]
```

**注意：** 由于这些参考对将作为常规白名单刷新的一部分而被刷新，最好保持这个列表的长度较短。可以指定所有时间周期和所有交易对，只要它们在所使用的交易所上是可用的（并且活跃的）。然而，最好在可能的情况下使用长时间周期的重新采样，以避免向交易所发送过多的请求并冒被阻止的风险。


## Additional data (DataProvider)¶

DataProvider是一个用于获取额外数据的工具，它可以在策略中使用。以下是一些可能的DataProvider选项：

- available_pairs - 属性，列出了缓存的交易对及其时间周期的元组列表。
- current_whitelist() - 返回当前白名单交易对的列表。对于访问动态白名单（例如VolumePairlist）非常有用。
- get_pair_dataframe(pair, timeframe) - 一个通用方法，根据需要返回历史数据（用于回测）或缓存的实时数据（用于Dry-Run和Live-Run模式）。
- get_analyzed_dataframe(pair, timeframe) - 返回经过分析的数据框（在调用populate_indicators()、populate_buy()、populate_sell()之后），以及最新分析的时间。
- historic_ohlcv(pair, timeframe) - 返回存储在磁盘上的历史数据。
- market(pair) - 返回交易对的市场数据：手续费、限制、精度、活跃标志等。参考ccxt文档以获取更多有关市场数据结构的详细信息。
- ohlcv(pair, timeframe) - 返回当前缓存的交易对的K线数据（OHLCV数据），返回DataFrame或空DataFrame。
- orderbook(pair, maximum) - 返回交易对的最新订单簿数据，一个带有最大条目数的买卖盘数据字典。
- ticker(pair) - 返回交易对的当前行情数据。参考ccxt文档以获取有关Ticker数据结构的更多详细信息。
- runmode - 属性，包含当前的运行模式。
示例用法：
```python
for pair, timeframe in self.dp.available_pairs:
    # 获取可用交易对和时间周期
    print(f"available {pair}, {timeframe}")
```
DataProvider提供了丰富的功能，可以帮助你获取多种类型的数据，在策略中使用这些数据来做出更好的决策。

