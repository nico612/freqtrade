

## 数据下载
[官方文档](https://www.freqtrade.io/en/stable/data-download/)

### 下载命令
`freqtrade download-data` 如果不添加额外的参数选项，freqtrade 会下载 最近30天 "1m" 和 "5m" timeframes 的数据，交易所和交易对来自 `config.json` 配置

#### 指定交易所
```shell
freqtrade download-data --exchange binance
```
该命令会下载配置文件中所有的交易对

#### 指定交易对
```shell
freqtrade download-data --exchange binance --pairs ETH/USDT BTC/USDT
```
#### 支持正则
```shell
freqtrade download-data --exchange binance --pairs .*/USDT
```

#### 其他
- 指定特定的目录：`--datadir user_data/data/some_directory`
- 指定不同的confing：`-c/--config config.json`
- 在其他目录中使用 `pairs.json`： `--pairs-file some_other_dir/pairs.json`
- 指定天数默认30天：`--days 10`
- 指定时间范围：`--timerange 20200101-`、`--timerange 20200101-20210101`
- 指定timeframe：`-t/--timeframes 1m  5m`
- 指定交易模式：`--trading-mode {spot,margin,futures}, --tradingmode {spot,margin,futures}` 默认使用配置文件中的

#### 下载当前时间范围之前的附加数据
例如已经下载了20220101到现在的数据，但是现在想回测更早的数据，可以使用`--prepend` 结合 `--timerange` 指定结束日期
```shell
freqtrade download-data --exchange binance --pairs ETH/USDT XRP/USDT BTC/USDT --prepend --timerange 20210101-20220101
```
如果数据可用，Freqtrade 将在此模式下忽略结束日期，并将结束日期更新为现有数据起点。

#### Data format
freqtrade 默认数据格式为 feather - a dataformat based on Apache Arrow

#### 子命令
`freqtrade download-data -h` 查看更多子命令用法

## 回测

查看回测子命令：`freqtrade backtesting -h`
```shell
usage: freqtrade backtesting [-h] [-v] [--logfile FILE] [-V] [-c PATH] [-d PATH] [--userdir PATH] [-s NAME] [--strategy-path PATH] [--recursive-strategy-search] [--freqaimodel NAME]
                             [--freqaimodel-path PATH] [-i TIMEFRAME] [--timerange TIMERANGE] [--data-format-ohlcv {json,jsongz,hdf5,feather,parquet}] [--max-open-trades INT]
                             [--stake-amount STAKE_AMOUNT] [--fee FLOAT] [-p PAIRS [PAIRS ...]] [--eps] [--dmmp] [--enable-protections] [--dry-run-wallet DRY_RUN_WALLET]
                             [--timeframe-detail TIMEFRAME_DETAIL] [--strategy-list STRATEGY_LIST [STRATEGY_LIST ...]] [--export {none,trades,signals}] [--export-filename PATH]
                             [--breakdown {day,week,month} [{day,week,month} ...]] [--cache {none,day,week,month}] [--freqai-backtest-live-models]

options:
  -h, --help            show this help message and exit
  -i TIMEFRAME, --timeframe TIMEFRAME
                        Specify timeframe (`1m`, `5m`, `30m`, `1h`, `1d`).
  --timerange TIMERANGE
                        Specify what timerange of data to use.
  --data-format-ohlcv {json,jsongz,hdf5,feather,parquet}
                        Storage format for downloaded candle (OHLCV) data. (default: `feather`).
  --max-open-trades INT
                        Override the value of the `max_open_trades` configuration setting.
  --stake-amount STAKE_AMOUNT
                        Override the value of the `stake_amount` configuration setting.
  --fee FLOAT           Specify fee ratio. Will be applied twice (on trade entry and exit).
  -p PAIRS [PAIRS ...], --pairs PAIRS [PAIRS ...]
                        Limit command to these pairs. Pairs are space-separated.
  --eps, --enable-position-stacking
                        Allow buying the same pair multiple times (position stacking).
  --dmmp, --disable-max-market-positions
                        Disable applying `max_open_trades` during backtest (same as setting `max_open_trades` to a very high number).
  --enable-protections, --enableprotections
                        Enable protections for backtesting.Will slow backtesting down by a considerable amount, but will include configured protections
  --dry-run-wallet DRY_RUN_WALLET, --starting-balance DRY_RUN_WALLET
                        Starting balance, used for backtesting / hyperopt and dry-runs.
  --timeframe-detail TIMEFRAME_DETAIL
                        Specify detail timeframe for backtesting (`1m`, `5m`, `30m`, `1h`, `1d`).
  --strategy-list STRATEGY_LIST [STRATEGY_LIST ...]
                        Provide a space-separated list of strategies to backtest. Please note that timeframe needs to be set either in config or via command line. When using this
                        together with `--export trades`, the strategy-name is injected into the filename (so `backtest-data.json` becomes `backtest-data-SampleStrategy.json`
  --export {none,trades,signals}
                        Export backtest results (default: trades).
  --export-filename PATH, --backtest-filename PATH
                        Use this filename for backtest results.Requires `--export` to be set as well. Example: `--export-filename=user_data/backtest_results/backtest_today.json`
  --breakdown {day,week,month} [{day,week,month} ...]
                        Show backtesting breakdown per [day, week, month].
  --cache {none,day,week,month}
                        Load a cached backtest result no older than specified age (default: day).
  --freqai-backtest-live-models
                        Run backtest with ready models.

Common arguments:
  -v, --verbose         Verbose mode (-vv for more, -vvv to get all messages).
  --logfile FILE, --log-file FILE
                        Log to the file specified. Special values are: 'syslog', 'journald'. See the documentation for more details.
  -V, --version         show program's version number and exit
  -c PATH, --config PATH
                        Specify configuration file (default: `userdir/config.json` or `config.json` whichever exists). Multiple --config options may be used. Can be set to `-` to
                        read config from stdin.
  -d PATH, --datadir PATH, --data-dir PATH
                        Path to directory with historical backtesting data.
  --userdir PATH, --user-data-dir PATH
                        Path to userdata directory.

Strategy arguments:
  -s NAME, --strategy NAME
                        Specify strategy class name which will be used by the bot.
  --strategy-path PATH  Specify additional strategy lookup path.
  --recursive-strategy-search
                        Recursively search for a strategy in the strategies folder.
  --freqaimodel NAME    Specify a custom freqaimodels.
  --freqaimodel-path PATH
                        Specify additional lookup path for freqaimodels.

```


在进行回测之前下载回测数据
`freqtrade download-data --exchange binance --pairs ETH/USDT:USDT BTC/USDT:USDT --timerange 20230601- -t 5m -c user_data/config.json`


回测会默认使用`config.json`文件中配置的交易对和`user/data/<exchange>`的数据

所有的利润计算都包含了手续费，freqtrade将会使用交易所默认的手续费进行计算


### 使用动态交易对列表
当使用 StaticPairlist 以外的配对列表时，无法保证回溯测试结果的再现性，为了获得可重现的结果，最好通过 test-pairlist 命令生成一个配对列表并将其用作静态配对列表。

默认情况下 freqtrade 会将回测结果导入到 `user_data/backtest_results`目录中，导出的交易可用于[进一步分析]((https://www.freqtrade.io/en/stable/backtesting/#further-backtest-result-analysis))，也可通过脚本目录中的[绘图子命令]((https://www.freqtrade.io/en/stable/plotting/#plot-price-and-indicators)) ([freqtradeplot-dataframe]) 使用。

### 回测示例
```shell
freqtrade backtesting --strategy AwesomStartegy
```
可以使用使用`-s AwesomeStrategy` 指定策略类名，默认在`user_data/strategies`目录中查找

#### 指定 timeframe
```shell
freqtrade backtesting -s AwesomeStrategy  --timeframe 5m
```

#### 自定义开始余额
```shell
freqtrade backtesting -s AwesomeStrategy --dry-run-wallet 1000
```

#### 指定数据目录

```shell
freqtrade backtesting --strategy AwesomeStrategy --datadir user_data/data/binance/20200101
```

#### 比较多种策略

```shell
freqtrade backtesting --strategy-list SampleStrategy1 AwesomeStrategy --timeframe 5m
```

#### 指定导出交易到自定义文件呢
```shell
freqtrade backtesting --strategy backtesting --export trades --export-filename=backtest_samplestrategy.json
```
#### 提供自定义交易手续费
可以使用 --fee 命令行选项将此值提供给回溯测试。 该费用必须是一个比率，并且将应用两次（一次用于交易进入，一次用于交易退出）。

比如每次订单费率为 0.1% (0.001 作为 比率)
```shell
freqtrade backtesting --fee 0.001
```
**注意：** 仅当您想尝试不同的费用值时才提供此选项（或相应的配置参数）。 默认情况下，回测从交易对/市场信息中获取默认费用。

#### 提高回测准确性
回溯测试的一大限制是它无法知道价格在蜡烛内的走势（收盘前很高，还是反之亦然？）。 因此，假设您在 1 小时的时间范围内进行回溯测试，该蜡烛将有 4 个价格（开盘价、最高价、最低价、收盘价）。

虽然回溯测试确实对此做出了一些假设（请参阅上文），但这永远不可能是完美的，并且总是会以某种方式存在偏见。 为了缓解这种情况，freqtrade 可以使用较低（较快）的时间范围来模拟蜡烛内变动。

要利用此功能，您可以将 `--timeframe-detail` 5m 附加到常规回测命令中。

这将加载该时间范围内的 1h 数据以及 5m 数据。 该策略将使用 1 小时时间范围进行分析，进场订单将仅在主时间范围内下达，但订单执行和退出信号将在 5 分钟蜡烛上进行评估，模拟蜡烛内的走势。

`--timeframe-detail` 必须小于原始时间范围，否则回测将无法启动。

显然，这将需要更多的内存（5m 数据比 1h 数据大），并且还会影响运行时间（取决于交易量和交易持续时间）。 此外，数据必须已经可用/下载。