Here's a quick reference for running Apache Flink with Docker:Here's a quick reference for running Apache Flink with Docker.Here's what you need to get started:

**docker-compose.yml**
```yaml
version: "3.8"
services:
  jobmanager:
    image: flink:latest
    ports:
      - "8081:8081"
    command: jobmanager
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
    networks:
      - flink-net

  taskmanager:
    image: flink:latest
    depends_on:
      - jobmanager
    command: taskmanager
    scale: 2
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
        taskmanager.numberOfTaskSlots: 2
    networks:
      - flink-net

networks:
  flink-net:
```

**Common commands:**
```bash
# Start the cluster
docker compose up -d

# Check logs
docker compose logs -f jobmanager

#
docker ps


# Submit a job (from inside the container)
docker exec -it <jobmanager-container> flink run /opt/flink/examples/streaming/WordCount.jar

# Scale task managers
docker compose up -d --scale taskmanager=4

# Tear down
docker compose down
```

**Key points:**
- The **Web UI** is at `http://localhost:8081` — use it to monitor jobs, view metrics, and cancel tasks.
- `jobmanager.rpc.address` must match the service name in compose so TaskManagers can discover the JobManager.
- Each TaskManager has `numberOfTaskSlots` slots; total cluster parallelism = slots × number of TMs.
- For a specific Flink version, swap `flink:latest` for something like `flink:1.19-scala_2.12`.

Click any box in the diagram above to ask a follow-up question about that component.



这个错误是因为你直接复制了文档里的占位符 `<jobmanager-container>`，没有替换成真实的容器名或 ID。

**第一步：查找真实的容器名**

```bash
docker ps
```

找到 Flink JobManager 对应的容器，复制 `NAMES` 列的值，例如 `learn-docker-jobmanager-1`。

**第二步：替换后执行**

```bash
docker exec -it learn-docker-jobmanager-1 flink run /opt/flink/examples/streaming/WordCount.jar
```

---

如果 `docker ps` 没有看到容器在运行，说明服务没启动，先执行：

```bash
docker compose up -d
```

然后再 `docker ps` 确认容器状态为 `Up` 后重试。