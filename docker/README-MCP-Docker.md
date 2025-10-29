# TrendRadar MCP Server Docker 部署指南

## 📦 文件说明

- `Dockerfile.mcp` - MCP Server 专用的 Docker 镜像定义
- `docker-compose-mcp.yml` - MCP Server 的 Docker Compose 配置

## 🚀 快速启动

### 1. 使用 Docker Compose 启动（推荐）

```bash
# 进入 docker 目录
cd docker

# 构建并启动服务
docker-compose -f docker-compose-mcp.yml up -d --build

# 查看日志
docker-compose -f docker-compose-mcp.yml logs -f

# 停止服务
docker-compose -f docker-compose-mcp.yml down
```

### 2. 使用 Docker 命令启动

```bash
# 构建镜像
docker build -f docker/Dockerfile.mcp -t trendradar-mcp:latest .

# 运行容器
docker run -d \
  --name trendradar-mcp-server \
  -p 3333:3333 \
  -v $(pwd)/config:/app/config:ro \
  -v $(pwd)/output:/app/output \
  -e TZ=Asia/Shanghai \
  --restart unless-stopped \
  trendradar-mcp:latest

# 查看日志
docker logs -f trendradar-mcp-server

# 停止容器
docker stop trendradar-mcp-server

# 删除容器
docker rm trendradar-mcp-server
```

## 🔧 配置说明

### 端口映射

- `3333:3333` - MCP Server HTTP 端点

### 卷挂载

- `../config:/app/config:ro` - 配置文件目录（只读）
- `../output:/app/output` - 数据输出目录（读写）

### 环境变量

- `TZ=Asia/Shanghai` - 时区设置
- `PYTHONUNBUFFERED=1` - Python 输出不缓冲
- `CONFIG_PATH=/app/config/config.yaml` - 配置文件路径
- `FREQUENCY_WORDS_PATH=/app/config/frequency_words.txt` - 关键词文件路径

## 📡 访问 MCP Server

启动后，MCP Server 将在以下地址提供服务：

```
http://localhost:3333/mcp
```

### 测试连接

```bash
# 测试健康检查
curl http://localhost:3333/mcp

# 或使用浏览器访问
open http://localhost:3333/mcp
```

## 🔍 故障排查

### 查看容器状态

```bash
docker ps -a | grep trendradar-mcp
```

### 查看实时日志

```bash
docker logs -f trendradar-mcp-server
```

### 进入容器调试

```bash
docker exec -it trendradar-mcp-server bash
```

### 查看健康检查状态

```bash
docker inspect --format='{{json .State.Health}}' trendradar-mcp-server | jq
```

## 🔄 更新服务

```bash
# 停止并删除旧容器
docker-compose -f docker-compose-mcp.yml down

# 拉取最新代码
git pull

# 重新构建并启动
docker-compose -f docker-compose-mcp.yml up -d --build
```

## 🌐 远程访问配置

如果需要从外部网络访问 MCP Server：

1. **配置防火墙**
```bash
# 开放 3333 端口
sudo ufw allow 3333/tcp
```

2. **使用 Nginx 反向代理（推荐）**

创建 Nginx 配置文件 `/etc/nginx/sites-available/trendradar-mcp`：

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location /mcp {
        proxy_pass http://localhost:3333/mcp;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

3. **启用 HTTPS（推荐）**
```bash
sudo certbot --nginx -d your-domain.com
```

## 📊 监控与日志

### 查看资源使用情况

```bash
docker stats trendradar-mcp-server
```

### 导出日志

```bash
docker logs trendradar-mcp-server > mcp-server.log 2>&1
```

## 🐛 常见问题

### 1. 容器启动失败

- 检查端口 3333 是否被占用
- 确认 config 目录存在且包含 config.yaml
- 查看容器日志排查具体错误

### 2. 无法连接到 MCP Server

- 确认容器状态为 healthy
- 检查防火墙规则
- 验证端口映射是否正确

### 3. 数据无法持久化

- 检查 output 目录的挂载权限
- 确认容器有写入权限

## 📝 与其他 Docker 配置的区别

| 文件 | 用途 | 启动命令 |
|------|------|----------|
| `docker-compose.yml` | 定时爬取任务（Cron） | supercronic |
| `docker-compose-mcp.yml` | MCP Server (HTTP) | python -m mcp_server.server |

**注意**：这两个服务可以同时运行，互不冲突。

## 🔐 安全建议

1. 不要在公网直接暴露 MCP Server，使用 Nginx + HTTPS
2. 配置文件（config.yaml）中的 webhook 敏感信息使用环境变量
3. 定期备份 output 目录的数据
4. 限制容器资源使用（CPU/内存）

```yaml
# 在 docker-compose-mcp.yml 中添加资源限制
deploy:
  resources:
    limits:
      cpus: '1.0'
      memory: 1G
    reservations:
      cpus: '0.5'
      memory: 512M
```

## 📚 相关文档

- [FastMCP 文档](https://github.com/jlowin/fastmcp)
- [Docker Compose 文档](https://docs.docker.com/compose/)
- [TrendRadar 主文档](../readme.md)
- [Cherry Studio MCP 配置](../README-Cherry-Studio.md)

