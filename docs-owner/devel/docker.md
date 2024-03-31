## docker 其他常见操作命令


- 查看日志 `docker compose logs -f`
- 拉取最新镜像 `docker compose pull <image-name>`
- 查看正在运行的实例 `docker compose ps`
- 查看所有实例包括已经停止的 `docker compose ps -a`
- 停止容器 `docker compose stop <container-id>` 、 `docker compose stop <container--name>`
- 删除容器 `docker compose rm <container-id> ` 、`docker compose rm <container-name>`
- 一条命令中停止并删除容器 `docker compose rm -f <container-id>`

