

## 常用命令


### 通过模版创建一个新的策略
```shell
freqtrade new-strategy --strategy AwesomeStrategy
```
该策略默认保存到 `user_data/strategies`目录中

### 数据下载
```shell
freqtrade download-data --exchange binance --pairs ETH/USDT:USDT BTC/USDT:USDT --timerange 20230601- --timeframe 5m -c user_data/config.json
```

### 回测
```shell
freqtrade backtesting --strategy Strategy001 --pairs ETH/USDT:USDT BTC/USDT:USDT --timerange 20231110-  --timeframe 5m -c user_data/strategy001_config.json
```
- 指定多个策略: `--strategy-list SampleStrategy1 AwesomeStrategy`
- 指定数据目录：`--datadir user_data/data/binance/futures`
- 指定蜡烛变动的时间范围(提高回测准确性)：`--timeframe-detail`


### 获取交易所可用的交易对
```shell
freqtrade list-pairs --quote USD --print-json -c user_data/config.json
```
指定交易所

```shell
$ freqtrade list-markets --exchange binance --all
```

### 动态交易对 Test pairlist

使用 `test-pairlist` 子命令测试动态配对列表的配置。
获取配置文件中USDT的动态交易对
```shell
freqtrade test-pairlist --config config.json --quote USDT
```
