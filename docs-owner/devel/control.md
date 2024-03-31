

## webUI服务配置
```
"api_server": {
    "enabled": true,
    "listen_ip_address": "0.0.0.0",
    "listen_port": 8282,
    "verbosity": "error",
    "enable_openapi": false,
    "jwt_secret_key": "jwt_secret_key",
    "CORS_origins": [],
    "username": "***",
    "password": "*****"
  }

```

## docker-compose.yml 配置

```solidity
docker-compose.yml

```solidity
ports:
      - "0.0.0.0:8282:8282"
```

## telegram push 配置

```json
"telegram": {
    "enabled": true,
    "token": "xxx:xxxxxxxxxxxxxx", // 格式
    "chat_id": "chat_id",
     "keyboard":[
            ["/daily", "/weekly", "/monthly", "/stats", "/balance", "/profit"],
            ["/count", "/start", "/stop", "/help"],
            ["/status", "/status table", "/performance"],
            ["/reload_config", "/count", "/logs"]
        ]
  },
```

