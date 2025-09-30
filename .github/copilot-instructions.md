# Docker Compose Services Collection

## 项目概述

这是一个Docker Compose配置文件的集合仓库，用于快速部署和管理常用的开发和生产环境服务。每个子目录包含一个独立服务的完整配置，设计用于WSL2环境下的Docker Desktop。

## 架构模式

### 目录结构约定

- **一服务一目录**：每个服务在根目录下有独立文件夹（如 `mysql/`, `elasticsearch-ik-cms/`, `postgres-17.6-alpine/`）
- **标准化命名**：服务目录名直接对应服务用途，带有版本或变体标识（如 `mysql8/`, `Kibana7/`, `elasticsearch-ik-cms/`）
- **配置文件组织**：
  - `docker-compose.yml` - 必需的服务编排文件
  - `README.md` - 服务特定的启动步骤和注意事项
  - 额外配置文件（如 `my.cnf`, `nginx.conf`, `application.properties`）直接挂载到容器

### 容器化服务类型

1. **基础存储服务**：MySQL (5.7/8), PostgreSQL (17.6-alpine), MongoDB, Redis, Neo4j
2. **搜索与分析**：Elasticsearch + IK分词器, Kibana, Graylog
3. **消息队列**：Kafka + Zookeeper
4. **应用运行时**：Nginx, Tabby (AI编码), XXL-Job (任务调度)
5. **开发工具**：Portainer (容器管理), Bytebase (数据库变更), NoCoDB, Stirling-PDF
6. **监控工具**：Uptime Kuma, Beszel, WatchYourLAN
7. **自定义镜像**：`pgvector-alpine/` (带pgvector扩展的PostgreSQL), `wangbowen/elasticsearch-ik` (中文分词)

## 关键开发工作流

### 启动新服务的标准流程

```bash
# 1. 进入服务目录
cd <service-name>/

# 2. 检查README.md中的前置条件和特殊配置
cat README.md

# 3. 启动服务
docker compose up -d

# 4. 验证服务状态
docker compose ps
docker compose logs -f
```

### 自定义镜像构建流程（以pgvector-alpine为例）

```bash
cd pgvector-alpine/

# 构建镜像（使用国内镜像源加速）
docker build -t mypgvector:pg17.6-alpine .

# 使用docker-compose启动
docker compose up -d

# 进入容器启用扩展
docker exec -it <container-name> psql -U postgres -d vector_db
# 在psql中执行: CREATE EXTENSION vector;
```

### 配置文件注入模式

某些服务需要在容器启动后注入配置：

**MySQL binlog配置示例** (`mysql/README.md`)：
```bash
# 1. 启动MySQL
docker compose up -d

# 2. 创建binlog目录
docker exec -it mysql bash
mkdir -p /data/log/mysql
chown -R mysql:mysql /data/log/mysql
exit

# 3. 注入配置文件
docker cp my.cnf mysql:/etc/mysql/my.cnf

# 4. 重启生效
docker compose restart
```

**KKFileView水印配置** (`wangbowen-kkfileview/README.md`)：
- ⚠️ 环境变量传递无效，必须挂载 `application.properties` 文件
- 支持动态刷新，修改后无需重启容器

## 项目特定约定

### 时区设置标准

所有服务统一使用 `TZ: Asia/Shanghai` 或 `TX: Asia/Shanghai` 环境变量（注意Elasticsearch使用TX）

### 数据持久化模式

1. **命名卷（推荐）**：`volumes: - data:/var/lib/mysql` + 顶层 `volumes: data:`
2. **本地目录挂载**：`./data:/usr/share/elasticsearch/data`（用于需要直接访问数据的服务）
3. **Docker Socket挂载**：`/var/run/docker.sock:/var/run/docker.sock`（仅Portainer等管理工具）

### 网络端口约定

- 保持服务默认端口映射（MySQL: 3306, PostgreSQL: 5432, Redis: 6379）
- Web服务使用8xxx端口（Tabby: 8888, XXL-Job: 18080）
- 管理界面使用9xxx端口（Portainer: 9000, Elasticsearch: 9200）

### 重启策略

- 开发环境：`restart: unless-stopped`（默认）
- 生产环境：`restart: always`（MySQL/Redis等核心服务）
- 调试阶段：不设置restart（Graylog等问题服务）

## 特殊案例和已知问题

### WSL2磁盘迁移（C盘空间不足）

根目录 `README.md` 包含WSL2迁移指南，参考：
- [move-wsl脚本](https://github.com/pxlrbt/move-wsl)
- [WSL Ubuntu子系统迁移教程](https://zhuanlan.zhihu.com/p/636079359)

### 国内网络环境优化

- **Docker镜像源**：使用 `README.md` 中的可用镜像源列表（截至2024年6月15日）
- **Alpine APK源**：Dockerfile中配置清华大学镜像（见 `pgvector-alpine/Dockerfile`）
- **容器仓库推送**：优先使用阿里云/腾讯云容器镜像服务（见 `pgvector-alpine/README.md`）

### Elasticsearch特殊配置

**elasticsearch-ik-cms目录**：
- 基于 [ChestnutCMS](https://github.com/liweiyi/ChestnutCMS) 项目定制
- IK分词器词库需要CMS项目运行才能动态更新
- 启动后需设置 `xpack.security.enabled=false` 环境变量解决访问问题（参考 `elasticsearch-ik-cms/README.md`）

### PostgreSQL版本选择

- **postgres-17.6-alpine**：解决 `program "postgres" is needed by initdb` 错误（参考 `postgres-17.6-alpine/README.md`）
- **pgvector-alpine**：禁用LTO编译（`make NO_LTO=1`）以兼容Alpine环境

### 已知废弃服务

- **Graylog**：文档问题严重，启动失败，已标记不可用（`Graylog/README.md`）

## 扩展新服务的最佳实践

1. **创建服务目录**：在根目录下创建 `<service-name>/` 文件夹
2. **编写docker-compose.yml**：
   - 使用 `version: '3.7'` 或更高版本
   - 设置 `container_name` 便于管理
   - 配置 `restart: unless-stopped`
   - 定义 `volumes` 持久化数据
3. **添加README.md**：记录特殊配置步骤、端口映射、默认凭证和已知问题
4. **更新.gitignore**：排除 `data/` 和 `logs/` 等运行时生成目录
5. **验证服务**：测试完整的启动→配置→验证流程

## 常用命令速查

```bash
# 启动服务
docker compose up -d

# 查看日志
docker compose logs -f [service-name]

# 停止服务
docker compose down

# 重启服务
docker compose restart

# 进入容器
docker exec -it <container-name> bash

# 清理未使用的资源
docker system prune -a --volumes

# 创建命名卷
docker volume create <volume-name>

# 查看卷位置
docker volume inspect <volume-name>
```

## 安全注意事项

- 所有示例密码（如 `MYSQL_ROOT_PASSWORD: root`）仅用于开发环境
- 生产部署前必须修改默认凭证
- Portainer等管理工具的 `/var/run/docker.sock` 挂载具有完整Docker权限，慎重使用
