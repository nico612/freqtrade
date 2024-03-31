# Configure the bot

## The Freqtrade configuration file

默认交易机器人会加载当前工作目录中的`config.json`文件，可以使用`-c`或`--config`命令指定不同配置文件。

创建新的基础配置文件命令：`freqtrade new-config --config config.json`

### Environment variables

可以通过设置环境变量来覆盖配置文件中的选项，比如yourExchangeKey、yourExchangeSecret等，环境变量的规则为

`FREQTRADE__{section}__{key}`的方式

例如：

```bash
FREQTRADE__TELEGRAM__CHAT_ID=<telegramchatid>					
FREQTRADE__TELEGRAM__TOKEN=<telegramToken>
FREQTRADE__EXCHANGE__KEY=<yourExchangeKey>
FREQTRADE__EXCHANGE__SECRET=<yourExchangeSecret>
```

### Multiple configuration files

#### 指定多个配置文件

可以使用多个配置文件来管理和组织您的交易机器人设置。这样，您可以将不同的配置参数分散到多个文件中，使得管理和修改配置更加灵活和方便。

要使用多个配置文件，您只需在启动 Freqtrade 时通过命令行参数传递多个配置文件路径。例如：

```bash
freqtrade trade -c config1.json -c config2.json -c config3.json
```

在这个例子中，我们传递了三个配置文件：`config1.json`、`config2.json` 和 `config3.json`。Freqtrade 会按照指定的顺序加载这些配置文件，并将它们合并为一个完整的配置。如果某个设置在多个文件中都有定义，那么后面指定的文件中的设置会覆盖先前的设置。

此外，您还可以将配置参数从标准输入流（stdin）传递给 Freqtrade。这对于需要在运行时动态生成配置的场景非常有用（在运行时动态生成配置的场景通常是指在启动交易机器人之前动态生成或修改配置，而不是在交易机器人已经运行时进行更改）。要使用此功能，您需要使用特殊的文件名 `-`（短横线）作为配置文件参数：

```bash
echo '{"stake_amount": 100, "stake_currency": "USDT"}' | freqtrade trade -c -
```

在这个例子中，我们通过 `echo` 命令将一个包含 `stake_amount` 和 `stake_currency` 设置的 JSON 字符串传递给 Freqtrade。Freqtrade 会从标准输入流读取这些配置参数并使用它们。

您还可以将多个配置文件与从 stdin 读取的配置一起使用。例如：

```bash
echo '{"stake_amount": 100, "stake_currency": "USDT"}' | freqtrade trade -c config1.json -c config2.json -c -
```

在这个例子中，我们同时使用了两个配置文件（`config1.json` 和 `config2.json`）以及从 stdin 传递的配置参数。注意，从 stdin 读取的配置参数具有最高优先级，它们会覆盖其他配置文件中的相应设置。

#### 通过add_config_files添加多个配置文件

在 Freqtrade 中，您可以使用 `add_config_files` 参数来指定和使用多个配置文件。通过这种方式指定的配置文件会与初始配置文件一起加载并合并。这些文件会相对于初始配置文件进行解析。

要使用 `add_config_files` 参数，您需要在初始配置文件中添加一个名为 `add_config_files` 的键，并将其值设置为一个包含其他配置文件路径的数组。例如，假设您有以下三个配置文件：`config1.json`（初始配置文件）、`config2.json` 和 `config3.json`。要将 `config2.json` 和 `config3.json` 添加到 `config1.json` 中，请在 `config1.json` 中添加如下配置：

```json
{
  "add_config_files": ["config2.json", "config3.json"],
  ...
}

```

现在，当您使用 `config1.json` 启动 Freqtrade 时，`config2.json` 和 `config3.json` 也会被自动加载并与 `config1.json` 合并：

```bash
freqtrade trade -c config1.json
```

请注意，这种方法仅适用于在 Freqtrade 运行时不需要更改的配置。如果您需要在运行时动态更改配置，请使用多个 `--config` 参数或从标准输入流（stdin）读取配置。

### 更改配置

在Freqtrade运行时更改配置通常是不支持的。要应用新的配置更改，您需要采取以下步骤：

1. 停止当前运行的交易机器人。确保您正在使用的命令行界面（CLI）或界面提供了停止命令。使用Docker部署Freqtrade时，您可以使用以下命令停止容器：

```bash
Copy code
docker-compose down
```

1. 修改配置文件。您可以直接编辑配置文件（如 `config.json`），根据需要更改相应的设置。保存更改后，关闭文件。
2. 重新启动交易机器人，应用新的配置。使用Docker时，可以使用以下命令重新启动交易机器人：

```bash
Copy code
docker-compose up -d
```

这将使交易机器人在新的配置下重新启动，新的设置将生效。

请注意，更改配置可能会影响当前运行的交易和策略。在修改配置之前，请务必了解每个设置的影响，并确保已备份当前配置文件以防止丢失重要设置。在修改配置时，建议您首先在回测或干运行模式下测试新设置，以确保新的配置按预期工作。

## Configuration parameters

官方比较全的配置示例

```json
{
    "max_open_trades": 3,										
    "stake_currency": "BTC",
    "stake_amount": 0.05,
    "tradable_balance_ratio": 0.99,
    "fiat_display_currency": "USD",
    "amount_reserve_percent": 0.05,
    "available_capital": 1000,
    "amend_last_stake_amount": false,
    "last_stake_amount_min_ratio": 0.5,
    "dry_run": true,
    "dry_run_wallet": 1000,
    "cancel_open_orders_on_exit": false,
    "timeframe": "5m",
    "trailing_stop": false,
    "trailing_stop_positive": 0.005,
    "trailing_stop_positive_offset": 0.0051,
    "trailing_only_offset_is_reached": false,
    "use_exit_signal": true,
    "exit_profit_only": false,
    "exit_profit_offset": 0.0,
    "ignore_roi_if_entry_signal": false,
    "ignore_buying_expired_candle_after": 300,
    "trading_mode": "spot",
    "margin_mode": "",
    "minimal_roi": {
        "40":  0.0,
        "30":  0.01,
        "20":  0.02,
        "0":  0.04
    },
    "stoploss": -0.10,
    "unfilledtimeout": {
        "entry": 10,
        "exit": 10,
        "exit_timeout_count": 0,
        "unit": "minutes"
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
    "exit_pricing":{
        "price_side": "same",
        "use_order_book": true,
        "order_book_top": 1,
        "price_last_balance": 0.0
    },
    "order_types": {
        "entry": "limit",
        "exit": "limit",
        "emergency_exit": "market",
        "force_exit": "market",
        "force_entry": "market",
        "stoploss": "market",
        "stoploss_on_exchange": false,
        "stoploss_price_type": "last",
        "stoploss_on_exchange_interval": 60,
        "stoploss_on_exchange_limit_ratio": 0.99
    },
    "order_time_in_force": {
        "entry": "GTC",
        "exit": "GTC"
    },
    "pairlists": [
        {"method": "StaticPairList"},
        {
            "method": "VolumePairList",
            "number_assets": 20,
            "sort_key": "quoteVolume",
            "refresh_period": 1800
        },
        {"method": "AgeFilter", "min_days_listed": 10},
        {"method": "PrecisionFilter"},
        {"method": "PriceFilter", "low_price_ratio": 0.01, "min_price": 0.00000010},
        {"method": "SpreadFilter", "max_spread_ratio": 0.005},
        {
            "method": "RangeStabilityFilter",
            "lookback_days": 10,
            "min_rate_of_change": 0.01,
            "refresh_period": 1440
        }
    ],
    "exchange": {
        "name": "binance",
        "sandbox": false,
        "key": "your_exchange_key",
        "secret": "your_exchange_secret",
        "password": "",
        "log_responses": false,
        // "unknown_fee_rate": 1,
        "ccxt_config": {},
        "ccxt_async_config": {},
        "pair_whitelist": [
            "ALGO/BTC",
            "ATOM/BTC",
            "BAT/BTC",
            "BCH/BTC",
            "BRD/BTC",
            "EOS/BTC",
            "ETH/BTC",
            "IOTA/BTC",
            "LINK/BTC",
            "LTC/BTC",
            "NEO/BTC",
            "NXS/BTC",
            "XMR/BTC",
            "XRP/BTC",
            "XTZ/BTC"
        ],
        "pair_blacklist": [
            "DOGE/BTC"
        ],
        "outdated_offset": 5,
        "markets_refresh_interval": 60
    },
    "edge": {
        "enabled": false,
        "process_throttle_secs": 3600,
        "calculate_since_number_of_days": 7,
        "allowed_risk": 0.01,
        "stoploss_range_min": -0.01,
        "stoploss_range_max": -0.1,
        "stoploss_range_step": -0.01,
        "minimum_winrate": 0.60,
        "minimum_expectancy": 0.20,
        "min_trade_number": 10,
        "max_trade_duration_minute": 1440,
        "remove_pumps": false
    },
    "telegram": {
        "enabled": false,
        "token": "your_telegram_token",
        "chat_id": "your_telegram_chat_id",
        "notification_settings": {
            "status": "on",
            "warning": "on",
            "startup": "on",
            "entry": "on",
            "entry_fill": "on",
            "exit": {
                "roi": "off",
                "emergency_exit": "off",
                "force_exit": "off",
                "exit_signal": "off",
                "trailing_stop_loss": "off",
                "stop_loss": "off",
                "stoploss_on_exchange": "off",
                "custom_exit": "off"
            },
            "exit_fill": "on",
            "entry_cancel": "on",
            "exit_cancel": "on",
            "protection_trigger": "off",
            "protection_trigger_global": "on",
            "show_candle": "off"
        },
        "reload": true,
        "balance_dust_level": 0.01
    },
    "api_server": {
        "enabled": false,
        "listen_ip_address": "127.0.0.1",
        "listen_port": 8080,
        "verbosity": "error",
        "enable_openapi": false,
        "jwt_secret_key": "somethingrandom",
        "CORS_origins": [],
        "username": "freqtrader",
        "password": "SuperSecurePassword",
        "ws_token": "secret_ws_t0ken."
    },
    "external_message_consumer": {
        "enabled": false,
        "producers": [
          {
            "name": "default",
            "host": "127.0.0.2",
            "port": 8080,
            "ws_token": "secret_ws_t0ken."
          }
        ],
        "wait_timeout": 300,
        "ping_timeout": 10,
        "sleep_time": 10,
        "remove_entry_exit_signals": false,
        "message_size_limit": 8
    },
    "bot_name": "freqtrade",
    "db_url": "sqlite:///tradesv3.sqlite",
    "initial_state": "running",
    "force_entry_enable": false,
    "internals": {
        "process_throttle_secs": 5,
        "heartbeat_interval": 60
    },
    "disable_dataframe_checks": false,
    "strategy": "SampleStrategy",
    "strategy_path": "user_data/strategies/",
    "recursive_strategy_search": false,
    "add_config_files": [],
    "reduce_df_footprint": false,
    "dataformat_ohlcv": "json",
    "dataformat_trades": "jsongz"
}

```

- max_open_trades: **required**

  允许您的机器人进行的未平仓交易数量，-1：表示不限制

- stake_currency：**required** 用于交易的加密货币，通常为usdt

- stake_amount：**required** 每次用于交易的金额，如果设置为unlimited表示交易机器可以使用所有的可用余额进行交易

- tradable_balance_ratio：指定了机器人可以用于交易的账户总余额的比例。这个参数的值应该是一个介于0.1和1.0之间的正浮点数。默认值为0.99，即99%。这意味着，如果您的账户总余额为1000美元，那么机器人将使用990美元（即99%）进行交易，其余的10美元（1%）将作为保留资金。

- available_capital：它指定了机器人可用的初始资本。这个参数对于在同一个交易所账户上运行多个机器人很有用，因为它可以确保每个机器人有一个单独的资本份额来管理和交易。参数的值应该是一个正浮点数。例如，如果您的账户总余额为2000美元，`available_capital` 设置为1000美元，`tradable_balance_ratio` 设置为0.75，那么机器人将使用750美元（75%）进行交易。

- fiat_display_currency：用于显示您的利润的法定货币。

- amend_last_stake_amount：default = false, 是 Freqtrade 配置中一个实验性的选项，当设置为 `true` 时，如果交易所或钱包余额不足以满足机器人的下一笔交易，它将尝试减少最后一次交易的头寸大小。换句话说，它会在最后一次交易时使用尽可能多的可用资金。

- last_stake_amount_min_ratio：Default = 0.5，用于在 `amend_last_stake_amount` 设置为 `true` 时定义剩余可用资金的最小头寸大小。这个选项的主要目的是确保当 `amend_last_stake_amount` 尝试减少最后一个头寸大小以适应剩余资金时，交易的头寸大小不会低于此比率。

  例如，如果 `last_stake_amount_min_ratio` 设置为 `0.5`，那么在最后一个头寸中，交易的头寸大小将不会低于总头寸的50%。这可以帮助避免交易太小的头寸，从而使得交易费用占比过高，影响策略的整体表现。

- amount_reserve_percent：default = 0.05(5%)

  它用于在计算每对交易的最小头寸大小时预留一定的资金。当设置这个选项时，机器人会在计算最小头寸大小时预留 `amount_reserve_percent` +止损值，以避免可能的交易拒绝。

  默认值为 0.05（5%）。这意味着，当计算每对交易的最小头寸大小时，机器人会预留额外的 5% 资金 +止损值。这有助于确保在止损触发时，您的账户中仍有足够的资金来执行交易。

  在使用此选项时，您应根据自己的风险容忍度和交易策略来调整 `amount_reserve_percent` 的值。请注意，过高的预留资金可能会限制您的交易机会，而过低的预留资金可能会导致止损订单被拒绝执行。因此，务必谨慎设置此值以达到您的策略目标。

- timeframe：设置周期，一般不在配置文件中设置，在策略中设置

- dry_run：**required**  default = true , 定义机器人运行是测试环境中还是正式环境中

- dry_run_wallet：试跑模式下的钱包余额，默认1000

- cancel_open_orders_on_exit：*Defaults to `false`* 当其值设为 `true` 时，将在以下情况下取消**所有未完成的订单**：通过 RPC 命令 `/stop` 停止机器人、按下 Ctrl+C 或者机器人意外停止。这允许您在市场崩盘等突发情况下使用 `/stop` 命令来取消未成交和部分成交的订单。需要注意的是，**这个选项不会影响到已经开启的头寸**。

- process_only_new_candles：*Defaults to `true`*  

  当其值设为 `true` 时，机器人只在新的 K 线数据到来时处理指标。这可以减少系统负载，因为机器人不会在同一个 K 线上多次处理指标。默认值是 `true`。

  如果将此选项设置为 `false`，机器人将在每次循环时处理指标，这意味着同一个 K 线将被多次处理，从而增加了系统负载。然而，在某些情况下，这可能对您的策略有帮助，尤其是当您的策略依赖于逐笔数据而非仅仅依赖于 K 线数据时。

- minimal_roi：**Required** 

  定水平时自动平仓。这个选项可以帮助您在交易中实现及时止盈，防止回报率在达到峰值后回撤。`minimal_roi` 配置是一个字典，其中键表示交易持续时间（以分钟为单位），值表示在此时间段内应达到的最小回报率。

  ```json
  "minimal_roi": {
      "0": 0.1,
      "30": 0.05,
      "60": 0.01,
      "120": 0
  }
  ```

  此配置的含义是：

  - 在交易开始后的前 30 分钟内，如果达到 10% 的回报率，将平仓（止盈）。
  - 在 30 分钟至 60 分钟之间，如果达到 5% 的回报率，将平仓（止盈）。
  - 在 60 分钟至 120 分钟之间，如果达到 1% 的回报率，将平仓（止盈）。
  - 在交易进行了 120 分钟（2 小时）之后，可以在任何回报率水平上平仓。

- stoploss：**Required.**

  用于设置交易策略中的止损值。止损是一种风险管理策略，用于在交易亏损达到预定的百分比时自动平仓，以减少进一步的损失。`stoploss` 的值通常是一个负数，表示允许的最大亏损百分比。

  例如，以下是一个 `stoploss` 配置的示例：

  ```json
  "stoploss": -0.05
  ```

   此配置表示，如果某个交易的亏损达到或超过 5%，则将自动平仓。在实际交易中，根据您的策略和风险承受能力来设置合适的止损值非常重要。过低的止损值可能会导致频繁的交易失败，而过高的止损值可能会导致您在不良交易中承受过多的损失

- trailing_stop：用于启用或禁用跟踪止损。它根据市场价格的变化自动调整止损价格。跟踪止损的初始值与静态止损相同。要启用跟踪止损，您需要设置如下参数：。

  ```json
  "stoploss" = -0.10
  "trailing_stop": true
  ```

  此配置表示启用跟踪止损。请注意，跟踪止损是基于配置文件或策略文件中的 `stoploss` 值。

  这将激活一个算法，每当资产价格上涨时，它会自动将止损价格向上移动。

  以下是一个简化的示例：

  1. 机器人以100美元的价格购买一种资产。
  2. 止损定义为-10%。
  3. 当资产价格跌至90美元以下时，止损将被触发。
  4. 假设资产价格现在上涨至102美元。
  5. 现在的止损将是102美元的-10% = 91.8美元。
  6. 现在，资产价格下跌至101美元，止损价格仍然是91.8美元，并在91.8美元触发。

  总之，止损价格将根据观察到的最高价格调整，始终保持-10%（根据您的设置）的位置。这意味着，只要市场价格继续上涨，止损价格也会随之上升，从而锁定部分利润并减少潜在亏损。

  还可以结合其他相关参数，如 `trailing_stop_positive`（指定跟踪止损触发的盈利百分比）和 `trailing_stop_positive_offset`（指定跟踪止损点与当前市场价格的最小间距）。

  在实际交易中，使用跟踪止损可以帮助您更好地锁定利润，但也可能导致在市场波动较大时过早平仓。因此，为您的交易策略选择合适的跟踪止损参数至关重要

- trailing_stop_positive：用于设置跟踪止损触发的盈利百分比。当市场价格上涨并达到这个盈利百分比时，跟踪止损将被触发并开始跟随市场价格。跟踪止损点会随着市场价格的上涨而上涨，但不会随着价格下跌而下降。这样可以锁定部分利润，减少损失的风险。

  ```json
  "trailing_stop_positive": 0.01
  ```

  此配置表示当市场价格上涨 1% 时，跟踪止损将被触发。请注意，这个参数仅在启用 `trailing_stop` 时生效。

  在实际交易中，为您的交易策略选择合适的 `trailing_stop_positive` 参数至关重要。设置过小的值可能导致市场波动将您的止损点推得太近，导致过早平仓。相反，设置过大的值可能无法充分锁定利润。因此，根据您的交易策略和市场情况选择合适的值非常重要。

- trailing_stop_positive_offset：用于设置跟踪止损触发的盈利百分比偏移量。它的作用是在达到 `trailing_stop_positive`（跟踪止损触发的盈利百分比）之前为止损点提供一个额外的缓冲区。在市场价格达到 `trailing_stop_positive` 加上 `trailing_stop_positive_offset` 时，跟踪止损才会生效。

  `trailing_stop_positive_offset` 的值是一个浮点数，表示盈利百分比。例如：

  ```json
  "trailing_stop": true
  "trailing_stop_positive": 0.01,
  "trailing_stop_positive_offset": 0.02
  ```

  1. 市场价格首次上涨2%（`trailing_stop_positive_offset`），跟踪止损触发。
  2. 市场价格继续上涨。在这个阶段，止损价格将始终保持在市场价格下方 1% 的位置（`trailing_stop_positive`）。

  所以，一旦触发跟踪止损，止损价格将根据市场价格的变化调整，但始终保持与市场价格之间由 `trailing_stop_positive` 设置的距离。更多止损参看止损篇

- trailing_only_offset_is_reached：*Defaults to `false`.* 

  用于确定在启用跟踪止损偏移量之前是否需要达到指定的偏移量。这个选项默认值是 `False`。当设置为 `True` 时，跟踪止损将仅在达到 `trailing_stop_positive_offset` 设定的偏移量时启动。这意味着在达到这个偏移量之前，将使用默认的 `stoploss`。

  ```
  stoploss = -0.9
  trailing_stop = True
  trailing_stop_positive = 0.1
  trailing_stop_positive_offset = 1.5
  trailing_only_offset_is_reached = True
  ```

  在这个例子中：

  1. 机器人以100美元的价格购买一种资产。
  2. 止损定义为-10%。
  3. 当资产价格跌至90美元以下时，止损将被触发。
  4. 假设资产价格现在上涨至104美元（达到5%的偏移量）。
  5. 由于 `trailing_only_offset_is_reached` 设置为 `True`，跟踪止损现在将启动，并将以104美元的1%（即103.96美元）为止损价格。
  6. 如果资产价格继续上涨，止损将以1%的幅度跟随价格上涨。

  通过设置 `trailing_only_offset_is_reached` 为 `True`，您可以确保跟踪止损仅在达到预定的偏移量时启用。在达到该偏移量之前，将使用默认的止损值。

- fee：配置选项用于在回测和试运行期间设置交易费用。通常情况下，不需要配置此选项，因为 Freqtrade 会默认使用交易所的默认费用。费用以比率形式设置（例如，0.001 = 0.1%）。每笔交易费用将被应用两次，一次在买入时，另一次在卖出时。

  如果您希望在回测和模拟交易期间使用特定的费率（而不是交易所的默认费率），可以通过 `fee` 配置项来设置。这有助于更准确地估计策略在实际交易时可能产生的收益或损失。

  请注意，在实际交易过程中，交易所会根据您的交易量和其他因素自动计算和应用实际费用，因此，在实际交易中，这个配置选项不会影响您的交易成本。

- futures_funding_rate：*Defaults to None.*

  配置选项用于在历史融资费率无法从交易所获取时使用用户指定的融资费率。这不会覆盖实际的历史费率。建议将此设置为0，除非您正在测试特定的币种，并且了解融资费率如何影响 Freqtrade 的利润计算。

  融资费率是衍生品市场（如期货和永续合约）中常见的费用，用于在合约到期前平衡合约价格与实际资产价格之间的差异。这些费用通常每8小时支付一次，并在交易过程中自动计算和结算。

  在回测期间，如果无法获取历史融资费率，您可以使用 `futures_funding_rate` 配置选项来设置一个固定的融资费率。这有助于更准确地估计策略在实际交易时可能产生的收益或损失。

  例如，如果您希望在回测期间使用每8小时0.05%的融资费率，可以在配置文件中添加以下内容：

  ```json
  {
      "futures_funding_rate": 0.0005
  }
  ```

  请注意，在实际交易过程中，交易所会根据市场条件自动计算和应用实际融资费率，因此，在实际交易中，这个配置选项不会影响您的交易成本。

- trading_mode：*Defaults to `"spot"`.*  配置选项允许您指定您想要进行的交易类型。您可以选择进行常规交易（现货交易）、杠杆交易或交易与加密货币价格匹配的合约。这里是不同的交易模式：

  - "spot": 现货交易。这是默认模式，代表您将进行常规的加密货币交易，买入和卖出实际的数字资产。
  - "margin": 杠杆交易。在这种模式下，您可以借用交易所提供的资金进行交易，以增加您的交易规模并提高潜在回报。请注意，杠杆交易也增加了风险，因为亏损可能会更大。
  - "futures": 期货交易。在这种模式下，您将进行**合约交易**，这些合约的价格是根据加密货币价格派生的。这允许您在不实际持有资产的情况下进行交易，并可能实现更高的回报。

  在配置文件中设置您想要的交易模式，例如：

  ```json
  {
      "trading_mode": "futures"
  }
  ```

  在开始使用杠杆交易或期货交易之前，请确保您了解相关风险，并熟悉这些交易模式的特点和操作方式。此外，您还需要确保您选择的交易所支持您要使用的交易模式。

- margin_mode

  配置选项允许您在使用杠杆交易时选择保证金模式。您可以选择共享保证金模式或独立保证金模式。这决定了您的保证金是否在不同交易对之间共享或隔离。

  - "cross": 跨保证金模式（全仓）。在这种模式下，您的账户中的保证金将在所有交易对之间共享。这意味着，如果一个交易对的仓位产生亏损，它可以使用您账户中的其他资金来平仓。这种模式可能会降低爆仓风险，但在某些情况下也可能导致更大的损失。
  - "isolated": 隔离保证金模式（逐仓）。在这种模式下，每个交易对都有单独的保证金，不与其他交易对共享。这意味着，如果一个交易对的仓位产生亏损，它只能使用为该交易对分配的保证金来平仓。这可能有助于限制单个仓位的损失，但可能增加爆仓的风险。

  要在配置文件中设置您喜欢的保证金模式，例如：

  ```json
  {
      "margin_mode": "isolated"
  }
  ```

- liquidation_buffer：*Defaults to `0.05`.* 用于设置在强平价格和止损价格之间留出多大的安全距离。这可以帮助防止您的仓位达到强平价格。这个比率用于计算在强平价格之上设置止损价格的距离。

  例如，如果您设置了 `liquidation_buffer` 为 0.05 (5%)，那么止损价格将在强平价格上方 5% 的位置设置。这样，如果市场价格快速下跌，止损订单将在到达强平价格之前被触发，从而避免强制平仓。

  请注意，`liquidation_buffer` 仅在使用杠杆交易（`trading_mode` 设置为 "margin" 或 "futures"）时适用。在设置这个值时，请确保理解它是如何影响您的止损订单和潜在风险的。同时确保您选择的交易所支持查看强平价格和设置这样的止损策略。

- unfilledtimeout.entry：**Required.**  用于设置交易机器人等待未成交的入场订单完成的时长（以分钟或秒为单位）。在此时间后，如果订单仍未完成，它将被取消并在当前（新）价格重新下单，只要仍然存在交易信号。

  例如，如果您将 `unfilledtimeout.entry` 设置为 5 分钟，当一个入场订单被创建后，机器人将等待 5 分钟以获得完全成交。如果在 5 分钟内订单仍未完全成交，则机器人将取消订单，并在仍有交易信号时重新评估入场。

  要在配置文件中设置 `unfilledtimeout.entry`，您可以这样做：

  ```json
  {
      "unfilledtimeout": {
          "entry": 5
      }
  }
  ```

  这里的值表示 5 分钟。您也可以使用秒数，例如：

  ```json
  {
      "unfilledtimeout": {
          "entry": "300s"
      }
  }
  ```

  这里的值表示 300 秒（即 5 分钟）。请确保您根据自己的交易策略和市场状况选择合适的值。

- unfilledtimeout.exit：它表示交易机器人会等待开仓的交易是否成功，在多长时间内等待未被成交的交易。如果在等待时间内交易仍未被成功成交，机器人将取消这笔交易，并重新根据当前市场价格下单。这个参数的单位可以是分钟或秒。

- unfilledtimeout.unit：*Defaults to `minutes`.*  设置超时所用的单位，也可设置为`seconds`秒

- unfilledtimeout.exit_timeout_count：指定了一个订单的超时次数，如果一个订单达到了超时次数，会触发紧急退出（emergency exit）。0表示禁用，允许无限制的订单取消

- entry_pricing：是用来确定进入交易的价格的配置参数。具体来说，entry_pricing有以下几个配置参数：

  - price_side：选择用于获取进入价格的买卖盘面。默认值为"same"，表示使用与交易方向相同的买卖盘面，也可以设置为"ask"、"bid"、"other"，表示使用卖盘或买盘。
  - use_order_book：*Defaults to `True`.*一个布尔值，表示是否使用订单簿来确定进入价格。如果为False，表示不使用订单簿，则会根据设置的entry_pricing.price_side和entry_pricing.price_last_balance参数，从市场深度中选择一个最佳的价格进行买入。
  - order_book_top：*Defaults to `1`.*当使用订单簿时，指定要使用订单簿中的第几个价格来确定进入价格。默认为1，表示使用订单簿中最顶部的价格。
  - price_last_balance：*Defaults to `0`.* 当不使用订单簿时，用于确定进入价格的系数。默认为0，表示使用与交易方向相同的最佳价格，为1表示使用最近的成交价格，0到1之间的值表示使用两者之间的价格插值。
  - check_depth_of_market.enabled：*Defaults to `false`* 一个布尔值，表示是否检查订单簿中买卖盘面的深度。默认为False，表示不进行检查。
  - check_depth_of_market.bids_to_ask_delta：*Defaults to `0`.* 如果启用了订单簿深度检查，则此参数定义买盘和卖盘之间的差异比率。值小于1表示卖盘的数量更大，大于1表示买盘的数量更大。默认值为0，表示不进行检查。

- exit_pricing：用于确定计算交易出场价格的方法。它包含多个子参数，可配置以定义出场价格计算方法。可用的子参数包括：

  - price_side：*Defaults to `same`.* 选择机器人应查看哪个价差方面以获取出场价格。
  - price_last_balance：*Defaults to `0`.* 用于在不使用orderbook时，用于确定卖出价格的系数。默认为0，表示使用与交易方向相同的最佳价格，为1表示使用最近的成交价格，0到1之间的值表示使用两者之间的价格插值。
  - use_order_book：*Defaults to `True`.* 确定机器人是否应使用订单簿计算出场价格。 
  - order_book_top：指定订单簿中要用作出场价格的条目。默认为1（顶部条目）。
  -  order_book_depth：确定在计算出场价格时要考虑多少条目。默认为0（使用所有可用条目）。 
  - check_depth_of_market.enabled：确定机器人在退出交易前是否应检查市场深度。默认为False。 
  - check_depth_of_market.bids_to_ask_delta：在订单簿中找到的买入订单和卖出订单的差异比率。值低于1表示卖单大小更大，而值大于1表示买单大小更大。默认为0（不检查市场深度）。

  这些子参数可以一起使用，以定义最适合您的交易策略的自定义出场价格计算方法。

- custom_price_max_distance_ratio：*Defaults to `0.02` 2%).*  用于设置当前价格和自定义入场或出场价格之间的最大距离比率，如果距离比率超过了这个值，Freqtrade将不会使用自定义价格而使用当前价格。默认值为0.02（即2%）。

- **exchange**：交易所相关配置

  - name：**Required.** 交易所名

  - key：交易所API KEY

  - secret：交易所API SECRET

  - sandbox：用于指定是否使用交易所的沙盒（sandbox）环境进行交易。沙盒环境是交易所提供的一种风险免费的集成测试环境，用户可以在其中模拟交易，以便测试和调试他们的交易策略和代码。

    如果`exchange.sandbox`设置为True，则Freqtrade将使用交易所的沙盒环境进行模拟交易。否则，Freqtrade将使用实际的交易所进行交易。如果 `dry_run` 为 `False`，设置 `exchange.sandbox` 为 `True`，则将尝试与实际的沙盒环境交互，而非真实的交易所环境。在这种情况下，您的交易将被视为实际的交易，并且会在沙盒环境中进行结算，但是您不会使用真实的资金进行交易。因此，在实际的交易中，应将 `exchange.sandbox` 设置为 `False`。

  - pair_whitelist：交易对列表设置

    ```json
    "pair_whitelist": [
          "BTC/USDT:USDT",
          "ETH/USDT:USDT"
        ],
    ```

  - pair_blacklist：交易对黑名单，如禁止使用BNB相关交易对

    ```json
    "pair_blacklist": ["BNB/.*"]
    ```

  - ccxt_config： 是用于传递给 ccxt 的其他参数，这些参数可以是特定于某个交易所的参数，需要在 ccxt 文档中查找。它适用于同时使用同步和异步的情况。请注意，不应在此处添加交易所密钥等敏感信息，以免泄露到日志中。

  - **markets_refresh_interval**：*Defaults to `60` minutes.* 用于设置市场数据的刷新间隔时间。这个参数表示多少分钟重新加载市场数据。默认值为60分钟，也就是每隔一小时重新加载一次市场数据。如果是短周期交易最好把这个参数设置为更短

  - skip_pair_validation ：*Defaults to `false`* 可以在启动时跳过交易对列表的验证。默认值为false，也就是说，在启动时Freqtrade将验证所有配置的交易对是否可用。如果将其设置为true，Freqtrade将跳过交易对列表的验证。请注意，这可能会导致交易对无法使用或产生意外结果，因此应谨慎使用。

  - skip_open_order_update ：*Defaults to `false`*：否在启动时跳过对未完成订单的更新，以防止交易所出现问题。仅适用于实时交易环境。默认值为 false。

  - unknown_fee_rate：*Defaults to `None`* 是计算交易手续费时使用的回退值。对于手续费以非交易货币计算的交易所来说，这可能很有用。这里提供的值将与“手续费成本”相乘。默认为None。

  - log_responses：*Defaults to `false`* 用于控制是否记录交易所的响应内容。如果设置为True，在调试模式下会记录所有的响应内容，但是需要谨慎使用，因为可能会涉及到一些敏感信息。

  - block_bad_exchanges：*Defaults to `true`.* 用于阻止已知无法正常工作的交易所。默认情况下，建议保持此参数为True，除非您想测试该交易所现在是否可以工作。如果您遇到交易所问题，可以先尝试将此参数设置为False，以便继续尝试运行。
  
- pairlists：配对列表处理程序定义机器人应该交易的配对列表（配对列表）。在配置中可以使用[`StaticPairList`](https://www.freqtrade.io/en/stable/plugins/#static-pair-list)和动态列表[`VolumePairList`](https://www.freqtrade.io/en/stable/plugins/#volume-pair-list) 处理程序，此外 [`AgeFilter`](https://www.freqtrade.io/en/stable/plugins/#agefilter), [`PrecisionFilter`](https://www.freqtrade.io/en/stable/plugins/#precisionfilter), [`PriceFilter`](https://www.freqtrade.io/en/stable/plugins/#pricefilter), [`ShuffleFilter`](https://www.freqtrade.io/en/stable/plugins/#shufflefilter), [`SpreadFilter`](https://www.freqtrade.io/en/stable/plugins/#spreadfilter) and [`VolatilityFilter`](https://www.freqtrade.io/en/stable/plugins/#volatilityfilter)作为列表过滤器，删除某些对和/或移动它们在配对列表中的位置。

  - StaticPairList：*Defaults to `StaticPairList`.* 静态列表，保持列表处理不变

    ```json
    "pairlists": [
        {"method": "StaticPairList"}
    ],
    ```

  - VolumePairList：使用成交量来排序或者过滤

    ```json
    "pairlists": [
        {
            "method": "VolumePairList",
            "number_assets": 20,				//交易对数量前20
            "sort_key": "quoteVolume",  //根据成交量排序
            "min_value": 0,							//最小成交量
            "refresh_period": 1800			//刷新时间
        }
    ]			
    ```

- order_types：订单类型

  1. **市价单**（Market Order）：市价单是指以市场上当前的最佳价格进行交易的订单类型，市价单是以最快的速度完成交易的方式，但并不保证订单被完全执行，因为市场价格可能会波动。
  2. **限价单**（Limit Order）：限价单是指以指定价格进行交易的订单类型。交易所会在价格达到指定价格时立即执行交易。但是，限价单无法保证交易的成交数量，因为订单可能无法完全匹配市场上的买卖盘。
  3. 止损限价单（Stop-Limit Order）：止损限价单是一种结合了限价单和止损单的交易订单类型。当价格达到设定的触发价格时，止损限价单会被激活，然后会以限价单的价格执行交易。止损限价单是一种帮助交易者在价格下跌时保护自己的投资的方式。
  4. 止损市价单（Stop-Market Order）：止损市价单是一种结合了市价单和止损单的交易订单类型。当价格达到设定的触发价格时，止损市价单会被激活，然后会以市价单的价格执行交易。止损市价单通常用于市场上出现急剧波动的情况下
  
  ```solidity
  order_types: {
          'entry': 'limit',   //买入使用限价单，也可设置为 market 市价单
          'exit': 'limit',		//卖出订单类型
          'stoploss': 'market',  //止损订单类型
          'stoploss_on_exchange': False //指定是否在交易所上设置止损单。如果为True，则在交易所上设置止损单；如果为False，则在本地设置止损单，不会发送到交易所。默认为False。
      }
  ```
  
  
