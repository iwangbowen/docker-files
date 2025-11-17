# CLAUDE.md

此文件为 Claude Code (claude.ai/code) 在此仓库中工作时提供指导。

## 项目概述

这是一个 Docker Compose 配置文件集合仓库，用于在 WSL2 环境下快速部署和管理开发与生产环境服务。每个服务在独立目录中，包含完整的配置和文档。

## 核心架构原则

### 目录组织模式

- **一服务一目录**：每个服务独立存在于根目录下（如 `mysql/`、`elasticsearch-ik-cms/`、`postgres-17.6-alpine/`）
- **标准文件结构**：
  - `docker-compose.yml`（必需）：服务编排配置
  - `README.md`（强烈推荐）：启动步骤、端口映射、默认凭证、已知问题
  - 额外配置文件：直接挂载到容器的配置文件（如 `my.cnf`、`nginx.conf`、`application.properties`）
  - `Dockerfile`（如需自定义镜像）：仅在需要构建自定义镜像时存在

### 配置注入的两种模式

1. **环境变量模式**（推荐）
   - 适用于大多数服务的基础配置
   - 在 `docker-compose.yml` 的 `environment` 部分定义

2. **配置文件挂载模式**（特殊场景）
   - 用于复杂配置或环境变量无效的场景
   - **MySQL binlog 配置**：容器启动后需 `docker cp my.cnf` 注入配置文件，然后重启容器
   - **KKFileView 水印配置**：环境变量传递无效，必须挂载 `application.properties` 文件（支持动态刷新）

## 常用命令

### 基础操作

```bash
# 在服务目录下启动服务
cd <service-name>/
docker compose up -d

# 查看服务状态
docker compose ps

# 查看实时日志
docker compose logs -f

# 停止服务
docker compose down

# 重启服务
docker compose restart

# 进入容器内部
docker exec -it <container-name> bash
```

### 自定义镜像构建（以 pgvector-alpine 为例）

```bash
cd pgvector-alpine/

# 构建镜像
docker build -t mypgvector:pg17.6-alpine .

# 使用 docker-compose 启动
docker compose up -d

# 进入容器启用扩展
docker exec -it <container-name> psql -U postgres -d vector_db
# 在 psql 中执行: CREATE EXTENSION vector;
```

### 维护命令

```bash
# 清理未使用的资源
docker system prune -a --volumes

# 创建命名卷
docker volume create <volume-name>

# 查看卷详细信息
docker volume inspect <volume-name>
```

## 项目特定约定

### 时区配置标准

- 统一使用 `TZ: Asia/Shanghai` 环境变量
- **特例**：Elasticsearch 使用 `TX: Asia/Shanghai`

### 数据持久化策略

1. **命名卷**（推荐）：`volumes: - data:/var/lib/mysql` + 顶层 `volumes: data:`
2. **本地目录挂载**：`./data:/usr/share/elasticsearch/data`（需要直接访问数据时）
3. **Docker Socket 挂载**：`/var/run/docker.sock:/var/run/docker.sock`（仅 Portainer 等管理工具）

### 端口分配约定

- 保持服务默认端口：MySQL (3306)、PostgreSQL (5432)、Redis (6379)
- Web 服务使用 8xxx 端口：Tabby (8888)、XXL-Job (18080)
- 管理界面使用 9xxx 端口：Portainer (9000)、Elasticsearch (9200)

### 重启策略规范

- 开发环境：`restart: unless-stopped`（默认）
- 生产环境核心服务：`restart: always`
- 调试阶段：不设置 restart

## 关键技术细节

### PostgreSQL 相关

**pgvector-alpine 自定义镜像**：
- 基于 `postgres:17.6-alpine3.22`
- 使用清华大学 Alpine 镜像源加速构建（`/etc/apk/repositories`）
- 禁用 LTO 编译（`make NO_LTO=1`）以兼容 Alpine 环境
- 启动后需手动执行 `CREATE EXTENSION vector;` 启用扩展

**postgres-17.6-alpine 版本选择**：
- 解决 `program "postgres" is needed by initdb` 错误（参考 `postgres-17.6-alpine/README.md`）

### Elasticsearch 特殊配置

**elasticsearch-ik-cms 目录**：
- 基于 [ChestnutCMS](https://github.com/liweiyi/ChestnutCMS) 项目定制
- IK 分词器词库需要 CMS 项目运行才能动态更新
- 启动后需设置 `xpack.security.enabled=false` 环境变量解决访问问题（参考 `elasticsearch-ik-cms/README.md`）

### KKFileView 配置

**wangbowen-kkfileview 自定义镜像**：
- 官方镜像更新不及时，使用自定义镜像：https://github.com/iwangbowen/kkFileView
- `wangbowen/kkfileview:4.4.2` 基于 `newer_libreoffice` 分支，解决带批注的 `doc`/`docx` 文档无法预览的问题
- 水印配置必须通过 `application.properties` 文件挂载，环境变量无效

### MySQL Binlog 配置流程

```bash
# 1. 启动 MySQL
docker compose up -d

# 2. 创建 binlog 目录
docker exec -it mysql bash
mkdir -p /data/log/mysql
chown -R mysql:mysql /data/log/mysql
exit

# 3. 注入配置文件
docker cp my.cnf mysql:/etc/mysql/my.cnf

# 4. 重启生效
docker compose restart
```

## 扩展新服务的标准流程

1. **创建服务目录**：在根目录下创建 `<service-name>/`
2. **编写 docker-compose.yml**：
   - 设置 `container_name` 便于管理
   - 配置 `restart: unless-stopped`
   - 定义 `volumes` 持久化数据
   - 设置 `TZ: Asia/Shanghai` 时区
3. **编写 README.md**：记录特殊配置步骤、端口映射、默认凭证和已知问题
4. **更新 .gitignore**：排除 `data/`、`logs/` 等运行时生成目录
5. **验证服务**：测试完整的启动→配置→验证流程

## 国内网络环境优化

### Docker 镜像源
- 使用根目录 `README.md` 中的可用镜像源列表（截至 2024 年 6 月 15 日）

### Alpine APK 源
- Dockerfile 中配置清华大学镜像：`https://mirrors.tuna.tsinghua.edu.cn/alpine/`（见 `pgvector-alpine/Dockerfile`）

### 容器仓库推送
- 优先使用阿里云/腾讯云容器镜像服务（见 `pgvector-alpine/README.md`）

## 已知问题和注意事项

- **Graylog**：文档问题严重，启动失败，已标记不可用（`Graylog/README.md`）
- **WSL2 磁盘迁移**：C 盘空间不足时，参考根目录 `README.md` 中的迁移指南
- **安全警告**：所有示例密码（如 `MYSQL_ROOT_PASSWORD: root`）仅用于开发环境，生产部署前必须修改
- **Portainer Socket 挂载**：`/var/run/docker.sock` 挂载具有完整 Docker 权限，生产环境慎重使用

## 服务分类参考

1. **基础存储**：MySQL (5.7/8)、PostgreSQL (17.6-alpine)、MongoDB、Redis、Neo4j
2. **搜索与分析**：Elasticsearch + IK 分词器、Kibana、Graylog
3. **消息队列**：Kafka + Zookeeper
4. **应用运行时**：Nginx、Tabby (AI 编码)、XXL-Job (任务调度)
5. **开发工具**：Portainer (容器管理)、Bytebase (数据库变更)、NoCoDB、Stirling-PDF
6. **监控工具**：Uptime Kuma、Beszel、WatchYourLAN
7. **自定义镜像**：pgvector-alpine (带 pgvector 扩展的 PostgreSQL)、elasticsearch-ik (中文分词)
