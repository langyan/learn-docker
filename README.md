# learn-docker
docker

# 以下是 Docker 常用命令速查表，按功能分组：几个常用的组合技巧：

```bash
# 删除所有已停止的容器
docker container prune

# 一键清理：停止容器 + 删镜像 + 删网络 + 删卷
docker compose down -v --rmi all

# 进入容器并执行命令（容器没有 bash 时用 sh）
docker exec -it <id> sh

# 启动时自动删除容器（适合临时调试）
docker run --rm -it ubuntu bash

# 查看镜像层历史
docker history <image>

# 把镜像导出为 tar 包（离线传输）
docker save -o app.tar app:latest
docker load -i app.tar
```



## docker compose 几个高频使用场景：

```bash
# 更新单个服务（不重启其他服务）
docker compose up -d --no-deps --build web

# 查看某服务最近 100 行日志并跟踪
docker compose logs -f --tail=100 web

# 临时运行一个命令（如 Django migrate），完成后自动删除容器
docker compose run --rm web python manage.py migrate

# 多环境配置（base + 覆盖文件）
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# 完全重置（删容器 + 卷 + 本地构建镜像）
docker compose down -v --rmi local && docker compose up -d --build
```


用户要的是 Markdown 表格文本，直接输出即可，不需要用 docx skill。

以下是 Docker 和 Docker Compose 常用命令整理成 Markdown 表格：

---

## Docker 常用命令

### 镜像管理

| 命令 | 说明 |
|------|------|
| `docker pull <image>` | 拉取镜像 |
| `docker images` | 列出本地镜像 |
| `docker build -t <name> .` | 从 Dockerfile 构建镜像 |
| `docker build --no-cache -t <name> .` | 不使用缓存构建 |
| `docker tag <image> <repo>:<tag>` | 给镜像打标签 |
| `docker push <repo>:<tag>` | 推送镜像到仓库 |
| `docker rmi <image>` | 删除镜像 |
| `docker image prune` | 清理悬空镜像 |
| `docker history <image>` | 查看镜像层历史 |
| `docker save -o app.tar <image>` | 导出镜像为 tar 包 |
| `docker load -i app.tar` | 从 tar 包加载镜像 |

### 容器管理

| 命令 | 说明 |
|------|------|
| `docker run -d -p 80:80 <image>` | 后台运行并映射端口 |
| `docker run --rm -it <image> bash` | 临时运行，退出后自动删除 |
| `docker run -v /host:/container <image>` | 挂载主机目录 |
| `docker ps` | 列出运行中的容器 |
| `docker ps -a` | 列出所有容器 |
| `docker stop <id>` | 停止容器 |
| `docker start <id>` | 启动已停止的容器 |
| `docker restart <id>` | 重启容器 |
| `docker rm <id>` | 删除容器 |
| `docker rm -f <id>` | 强制删除运行中的容器 |
| `docker container prune` | 删除所有已停止的容器 |

### 调试与日志

| 命令 | 说明 |
|------|------|
| `docker logs -f <id>` | 实时查看容器日志 |
| `docker logs --tail=100 <id>` | 查看最后 100 行日志 |
| `docker exec -it <id> bash` | 进入容器 shell |
| `docker inspect <id>` | 查看容器详细信息 |
| `docker stats` | 实时资源使用情况 |
| `docker top <id>` | 查看容器内进程 |
| `docker diff <id>` | 查看文件系统变更 |
| `docker cp <id>:/path .` | 从容器复制文件到本地 |

### 数据卷

| 命令 | 说明 |
|------|------|
| `docker volume create <name>` | 创建数据卷 |
| `docker volume ls` | 列出数据卷 |
| `docker volume inspect <name>` | 查看数据卷详情 |
| `docker volume rm <name>` | 删除数据卷 |
| `docker volume prune` | 清理未使用的数据卷 |

### 网络

| 命令 | 说明 |
|------|------|
| `docker network ls` | 列出网络 |
| `docker network create <name>` | 创建自定义网络 |
| `docker network inspect <name>` | 查看网络详情 |
| `docker network rm <name>` | 删除网络 |
| `docker network connect <net> <id>` | 将容器接入网络 |

---

## Docker Compose 常用命令

### 启动与停止

| 命令 | 说明 |
|------|------|
| `docker compose up` | 创建并启动所有服务（前台） |
| `docker compose up -d` | 后台运行 |
| `docker compose up --build` | 启动前强制重新构建镜像 |
| `docker compose up --scale web=3` | 指定服务副本数 |
| `docker compose down` | 停止并删除容器、网络 |
| `docker compose down -v` | 同时删除数据卷 |
| `docker compose down --rmi all` | 同时删除镜像 |
| `docker compose stop` | 停止服务（保留容器） |
| `docker compose start` | 启动已停止的服务 |
| `docker compose restart` | 重启所有服务 |
| `docker compose restart <service>` | 重启指定服务 |
| `docker compose pause / unpause` | 暂停 / 恢复服务 |

### 查看状态

| 命令 | 说明 |
|------|------|
| `docker compose ps` | 列出服务容器及状态 |
| `docker compose ps -a` | 包含已停止的容器 |
| `docker compose logs -f` | 实时跟踪所有服务日志 |
| `docker compose logs -f <service>` | 跟踪指定服务日志 |
| `docker compose logs --tail=50` | 只显示最后 50 行 |
| `docker compose top` | 查看各容器内进程 |
| `docker compose stats` | 实时资源使用情况 |
| `docker compose port <service> 80` | 查看服务映射的端口 |

### 构建与镜像

| 命令 | 说明 |
|------|------|
| `docker compose build` | 构建所有服务镜像 |
| `docker compose build <service>` | 只构建指定服务 |
| `docker compose build --no-cache` | 不使用缓存重新构建 |
| `docker compose pull` | 拉取所有服务的最新镜像 |
| `docker compose push` | 推送镜像到仓库 |
| `docker compose images` | 列出服务使用的镜像 |

### 执行与调试

| 命令 | 说明 |
|------|------|
| `docker compose exec <service> bash` | 进入运行中容器 shell |
| `docker compose run --rm <service> bash` | 临时启动容器，完成后自动删除 |
| `docker compose run <service> <cmd>` | 一次性执行命令 |
| `docker compose cp <service>:/path .` | 从容器复制文件 |
| `docker compose kill <service>` | 强制终止服务 |

### 配置与多环境

| 命令 | 说明 |
|------|------|
| `docker compose config` | 校验并输出合并后的配置 |
| `docker compose config --services` | 列出所有服务名 |
| `docker compose -f <file> up` | 指定配置文件 |
| `docker compose -f base.yml -f override.yml up` | 合并多个配置文件 |
| `docker compose --env-file .env.prod up` | 指定环境变量文件 |
| `docker compose -p <project> up` | 指定项目名（隔离命名空间） |

### 常用参数

| 参数 | 说明 |
|------|------|
| `-d` | 后台运行（detach） |
| `--build` | 启动前重新构建镜像 |
| `--no-deps` | 不启动依赖服务 |
| `--force-recreate` | 强制重建容器 |
| `--no-recreate` | 容器存在则不重建 |
| `--remove-orphans` | 删除配置中不存在的容器 |
| `--timeout 30` | 设置停止超时秒数 |
| `--profile dev` | 启用指定 profile 的服务 |

# 启动服务
docker compose up -d

# 查看日志
docker compose logs -f

# 停止并删除现有容器
 docker-compose down -v  

# 重启
 docker compose restart influxdb3 explorer