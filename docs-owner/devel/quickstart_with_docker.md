

## docker 安装

```shell
mkdir ft_userdata
cd ft_userdata

# Download the docker-compose file form the repository
curl https://raw.githubusercontent.com/freqtrade/freqtrade/stable/docker-compose.yml -o docker-compose.yml

# Pull the freqtrade image
docker compose pull

# Create user directory structure
docker compose run --rm freqtrade create-userdir --userdir user_data

# Create configuration - Requires answering interactive questions
docker compose run --rm freqtrade new-config --config user_data/config.json
```
编辑 `docker-compose.yml`文件 主要配置下方这个地方
```yaml
    ports:
      - "127.0.0.1:8282:8282"
    # Default command used when running `docker compose up`
    command: >
      trade
      --logfile /freqtrade/user_data/logs/freqtrade.log
      --db-url sqlite:////freqtrade/user_data/tradesv3.sqlite
      --config /freqtrade/user_data/config.json
      --strategy Strategy001 
    # --strategy: 指定策略的类名，注意不是文件名，最好是把文件名和类名写成一样不容易搞错
```

### 添加配置文件
建议直接将配置文件拷贝进去，用上面命令也可以根据需求创建一个基础的配置, 
在 `user_data/config.json` 中添加相关的配置、值支持环境变量，如果策略中也定义了相关参数、以代码中的参数为准

创建 `config.json` 配置文件
```shell
docker compose run --rm freqtrade new-config --config user_data/config.json
freqtrade - INFO - freqtrade 2023.10
? Do you want to enable Dry-run (simulated trades)? Yes
? Please insert your stake currency: USDT
? Please insert your stake amount (Number or 'unlimited'): 300
? Please insert max_open_trades (Integer or -1 for unlimited open trades): 50
? Time Have the strategy define timeframe.
? Please insert your display Currency (for reporting): USD
? Select exchange binance
? Do you want to trade Perpetual Swaps (perpetual futures)? Yes
? Do you want to enable Telegram? Yes
? Insert Telegram token ***
? Insert Telegram chat id **
? Do you want to enable the Rest API (includes FreqUI)? Yes
? Insert Api server Listen Address (0.0.0.0 for docker, otherwise best left untouched) 0.0.0.0
? Insert api-server username trader
? Insert api-server password *******
2023-11-16 15:06:39,359 - freqtrade.commands.build_config_commands - INFO - Writing config to `user_data/config.json`.
2023-11-16 15:06:39,359 - freqtrade.commands.build_config_commands - INFO - Please make sure to check the configuration contents and adjust settings to your needs.
```
配置api_server, 可以通过 浏览器 查看

```json
 "api_server": {
        "enabled": true,
        "listen_ip_address": "0.0.0.0",
        "listen_port": 8282,
        "verbosity": "error",
        "enable_openapi": false,
        "jwt_secret_key": "6befc703210808390885f1432ad59761df19db31586037f89d245342cc2eebe6",
        "ws_token": "A8KdO6c_ouyJfuX-Xor_33DfbM5UqDUhhg",
        "CORS_origins": [],
        "username": "trader",
        "password": "123456"
    },
```

### 添加策略
策略放入 `user_data/stategies`目录中，在指定策略类名时，**默认从该目录中查找**。

### 运行 freqtrade

执行 `docker compose up -d` 

### 查看日志：
`docker compose logs -f` 日志默认保存在`user_data/logs/freqtrade.log`中

### 数据库
数据库采用 SQLite 位于 `uesr_data/tradesv3.sqlite`  sqlite 常见操作见 [sqlite 文档](./sqlite.md)
### 更新
```shell
# Download the latest image
docker compose pull

# Restart the image
docker compose up -d
```
重启后注意观察日志是否重启成功


### 编辑 docker-compose 文件
可以编辑 docker-compose 文件以包含所有可能的选项或参数。

所有 freqtrade 命令选项都可以通过 `docker compose run --rm freqtrade <command> <option arguments>` 来执行

但是 `freqtrade trade <...>` 命令不能通过 `docker compose run ...` 来执行，可以通过 `docker compose up -d` 来代替。

`docker compose run --rm` 将会在执行完成后移除 container，强烈推荐使用该命令来执行除交易模式外（freqtrade trade command）的所有的模式


### 下载历史数据
```shell
docker compose run --rm freqtrade download-data --pairs ETH/BTC --exchange binance --days 5 -t 1h
```


更多下载历史数据命令选项： https://www.freqtrade.io/en/stable/data-download/

### 回测
```shell
docker compose run --rm freqtrade backtesting --config user_data/config.json --strategy SampleStrategy --timerange 20190801-20191001 -i 5m
```


更多用法见 ：https://www.freqtrade.io/en/stable/docker_quickstart/


### docker 常见命令
[](./docker.md)