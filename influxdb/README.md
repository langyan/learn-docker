

基于官方最新文档，以下是完整配置。---

## 目录结构

```
influxdb3/
├── docker-compose.yml
├── .env
├── db/              # Explorer SQLite 数据（自动创建）
├── config/
│   └── config.json  # Explorer 预配置连接
└── data/            # InfluxDB 3 Core 数据
```

---

## `docker-compose.yml`

```yaml
# docker-compose.yml
version: '3.8'

services:

  # ─── InfluxDB 3 Core ──────────────────────────────────────────
  influxdb3:
    image: influxdb:3-core
    container_name: influxdb3-core
    restart: unless-stopped
    ports:
      - "8181:8181"
    command:
      - influxdb3
      - serve
      - --node-id=node0
      - --object-store=file
      - --data-dir=/var/lib/influxdb3/data
    volumes:
      - influxdb3-data:/var/lib/influxdb3/data
    networks:
      - influx-net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8181/health"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ─── InfluxDB 3 Explorer ──────────────────────────────────────
  explorer:
    image: influxdata/influxdb3-ui:1.6.2
    container_name: influxdb3-explorer
    restart: unless-stopped
    command: ["--mode=admin"]
    ports:
      - "8888:80"
    volumes:
      - ./db:/db:rw                        # SQLite 持久化（必须）
      - ./config:/app-root/config:ro       # 预配置连接（可选）
    environment:
      SESSION_SECRET_KEY: ${SESSION_SECRET_KEY}
    depends_on:
      influxdb3:
        condition: service_healthy
    networks:
      - influx-net

volumes:
  influxdb3-data:

networks:
  influx-net:
    driver: bridge
```

---

## `.env`

```bash
# 生成随机 key：openssl rand -hex 32
SESSION_SECRET_KEY=your_64_char_hex_string_replace_this_with_real_value_here_now
```

---

## `config/config.json`（预配置 Explorer 连接）

```json
{
  "DEFAULT_INFLUX_SERVER": "http://influxdb3:8181",
  "DEFAULT_INFLUX_DATABASE": "mydb",
  "DEFAULT_API_TOKEN": "",
  "DEFAULT_SERVER_NAME": "Local InfluxDB 3 Core"
}
```

> 因为两个容器在同一 Docker 网络，直接用容器名 `influxdb3` 而不是 `host.docker.internal`。

---

## 启动步骤

```bash
# 1. 创建必要目录
mkdir -p db config data

# 2. 生成 SESSION_SECRET_KEY 并写入 .env
echo "SESSION_SECRET_KEY=$(openssl rand -hex 32)" > .env

# 3. 创建 Explorer 预配置（可选）
cat > config/config.json << 'EOF'
{
  "DEFAULT_INFLUX_SERVER": "http://influxdb3:8181",
  "DEFAULT_INFLUX_DATABASE": "mydb",
  "DEFAULT_API_TOKEN": "",
  "DEFAULT_SERVER_NAME": "Local InfluxDB 3 Core"
}
EOF

# 4. 启动
docker compose up -d

# 5. 查看日志
docker compose logs -f
```

---

## 初始化 Token（首次必做）

InfluxDB 3 Core 默认无 token，需要手动生成：

```bash
# 进入容器创建 admin token
docker exec -it influxdb3-core influxdb3 create token --admin

# 输出示例：
# Token: apiv3_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

然后将此 token 填入 Explorer UI 的连接配置，或更新 `config/config.json` 的 `DEFAULT_API_TOKEN` 字段后重启 Explorer。

---

## 验证

```bash
# 写入测试数据
curl -X POST "http://localhost:8181/api/v3/write_lp?db=mydb" \
  -H "Authorization: Bearer apiv3_your_token_here" \
  -H "Content-Type: text/plain" \
  --data-binary 'cpu,host=server01 usage=42.5'

# SQL 查询
curl -X POST "http://localhost:8181/api/v3/query_sql" \
  -H "Authorization: Bearer apiv3_your_token_here" \
  -H "Content-Type: application/json" \
  -d '{"db":"mydb","q":"SELECT * FROM cpu LIMIT 10"}'
```

浏览器访问 **http://localhost:8888** 即可进入 Explorer UI。

---

## 关键说明

| 项目 | 说明 |
|---|---|
| Explorer 镜像 | `influxdata/influxdb3-ui:1.6.2`（官方最新稳定版） |
| Explorer 端口 | 容器内 `:80`，映射到宿主机 `:8888` |
| `--mode=admin` | 开启完整管理功能（数据库、token、插件管理） |
| `./db` 目录 | 存储 Explorer 的 SQLite，**删掉会丢失所有连接配置** |
| Token 认证 | Core 默认关闭，通过 `influxdb3 create token --admin` 开启 |
| 镜像 tag 警告 | 官方提示 `latest` 将于 2026-05-27 指向 Core，建议固定版本号 |