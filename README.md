# FloatCTF 平台

基于 Docker 部署的自托管 CTF（夺旗赛）竞赛平台。

## 架构说明

平台由 4 个核心服务组成：

| 服务 | 镜像 | 说明 |
|------|------|------|
| `floatctf-db` | PostgreSQL 17 | 数据库 |
| `floatctf-rustfs` | rustfs/rustfs | S3 兼容对象存储 |
| `floatctf-nginx` | Nginx 1.26 | 反向代理和静态文件服务 |
| `floatctf-api` | Alpine + floatctf | 后端 API |

## 环境要求

- Docker 和 Docker Compose
- 约 10GB 可用磁盘空间

## 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/FloatCTF/floatctf-docker.git
cd floatctf-docker
```

### 2. 配置环境变量

编辑 `.env` 配置文件：

```env
API_ELF_URL="https://github.com/FloatCTF/floatctf/releases/latest/download/floatctf-linux-amd64-musl"
SQL_DIST_URL="https://github.com/FloatCTF/floatctf/releases/latest/download/sql.tar.gz"
HTML_DIST_URL="https://github.com/FloatCTF/floatctf-web/releases/latest/download/download/html.tar.gz"

INSTALLER_DIR="./app"

# API CONF
API_SERVER_IP="floatctf-api"
API_SERVER_PORT=9090
API_USER="floatctf_api"
NODE_IP="127.0.0.1"

# nginx
NGINX_SERVER_HTTP_PORT=80
NGINX_SERVER_HTTPS_PORT=443
NGINX_USER="nginx"

# database
PG_HOST="floatctf-db"
PG_PORT=5432
PG_USER=postgres
PG_PASSWORD=postgres
PG_DB=floatctf_db

# rustfs
RUSTFS_ACCESS_KEY=rustfsadmin
RUSTFS_SECRET_ACCESS_KEY=rustfsadmin
RUSTFS_ADDRESS="floatctf-rustfs:9000"

DOCKER_HOST_PATH="/var/run/docker.sock"
```

### 3. 初始化平台

```bash
chmod +x init.sh
./init.sh
```

初始化脚本会执行以下操作：
- 创建所需目录
- 生成 SSL 自签名证书
- 下载 API 程序、SQL 架构和前端文件
- 配置环境变量文件
- 设置 Nginx 配置

### 4. 启动服务

```bash
docker compose --env-file ./.env --env-file ./app/.env up -d
```

### 5. 访问平台

- Web 界面：`https://localhost:9443`
- API：`https://localhost:9443/api/`

## 目录结构

```
floatctf-docker/
├── docker-compose.yml    # 服务编排配置
├── init.sh              # 初始化脚本
├── README.md
├── LICENSE
├── .gitignore
└── app/
    ├── .env             # 环境变量配置
    ├── bin/             # floatctf API 程序
    ├── html/            # 前端静态文件
    ├── data/            # RustFS 数据卷
    ├── keys/            # SSL 证书
    ├── logs/            # 应用日志
    │   ├── api/
    │   ├── nginx/
    │   └── rustfs/
    ├── nginx/conf/      # Nginx 配置
    ├── tmp/             # 临时文件
    │   └── sql/         # 数据库架构
    └── 1.py             # S3 存储桶初始化脚本
```

## 服务说明

### PostgreSQL 数据库
- 端口：5432（内部）
- 版本：17
- 持久化卷：`pgdata`

### RustFS (S3 存储)
- 端口：9000（内部）
- 存储桶：
  - `floatctf-public` - 公共资源（图片、武器、题目）
  - `floatctf-private` - 私有文件（Writeups）

### Nginx
- HTTP：9980
- HTTPS：9443
- 提供静态文件服务和 API 反向代理

### FloatCTF API
- 运行 floatctf 程序
- 连接 PostgreSQL 和 RustFS

## 常用命令

```bash
# 查看日志
docker compose --env-file ./.env --env-file ./app/.env logs -f

# 查看指定服务日志
docker compose --env-file ./.env --env-file ./app/.env logs -f floatctf-api
docker compose --env-file ./.env --env-file ./app/.env logs -f floatctf-nginx

# 重启服务
docker compose --env-file ./.env --env-file ./app/.env restart

# 停止服务
docker compose --env-file ./.env --env-file ./app/.env down

# 重建并重启
docker compose --env-file ./.env --env-file ./app/.env up -d --force-recreate
```

## 故障排查

### API 无法连接数据库
- 检查 `.env` 中的 `DATABASE_URL` 配置是否正确
- 确认 PostgreSQL 正在运行：`docker compose ps floatctf-db`

### RustFS 连接问题
- 检查 `RUSTFS_ADDRESS` 是否与容器主机名匹配
- 验证 API 环境中的 `RUSTFS_ENDPOINT_URL`

### SSL 证书错误
- 默认会生成自签名证书
- 将 `app/keys/fullchain.pem` 和 `app/keys/privkey.pem` 替换为你的正式证书

## 许可证

详见 [LICENSE](LICENSE) 文件。
