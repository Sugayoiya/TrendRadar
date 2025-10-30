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

