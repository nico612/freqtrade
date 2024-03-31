

测试 

freqtrade 入口函数参考：

`freqtrade/freqtrade/__main__.py`

入参数为字符串数组，最好在freqtrade目录下新建测试文件运行，否则可能会出现找不到配置文件的问题。
    
```python
from freqtrade import main
import argparse
# freqtrade backtesting --timerange 20230506- -c config_fast_btc.json --strategy FastTradeStrategy

CONFIG_PATH = '**/freqtrade/config.json'
# 回测命令配置
backtesting = ["backtesting", "--timerange", "20230505-20230507", "-c", CONFIG_PATH, "--strategy", "FastTradeStrategy"]
# 试运行命令配置
tryRun = ["trade", "-c", CONFIG_PATH,  "--strategy", "FastTradeStrategy"]

# 数据下载命令配置
download = ["download-data", "-c", CONFIG_PATH, "--timerange", "20230101-", "-t", "5m"]


if __name__ == '__main__':

    main.main(backtesting)
    
```
