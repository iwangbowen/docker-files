# PostgreSQL with pgvector (Alpine-based)

这是一个基于 `postgres:17.6-alpine3.22` 的自定义Docker镜像，集成了 [pgvector](https://github.com/pgvector/pgvector) 扩展，用于向量相似性搜索。

## 功能特性

- 基于Alpine Linux，轻量级镜像
- 集成pgvector扩展，支持向量操作
- 使用国内镜像源加速构建
- 禁用LTO以兼容Alpine环境

## 构建镜像

### 前置要求

- Docker Desktop 或 Docker Engine
- 网络连接（用于下载源码）

### 构建步骤

1. 克隆或下载此仓库到本地。

2. 进入 `pgvector-alpine` 文件夹：

   ```bash
   cd pgvector-alpine
   ```

3. 构建镜像：

   ```bash
   docker build -t mypgvector:pg17.6-alpine .
   ```

   - 构建过程将下载pgvector源码并编译安装，大约需要几分钟。

4. 验证构建成功：

   ```bash
   docker images | grep mypgvector
   ```

## 使用镜像

### 使用 Docker Compose

1. 确保 `docker-compose.yml` 文件存在（已包含在仓库中）。

2. 启动容器：

   ```bash
   docker-compose up -d
   ```

3. 连接到数据库：
   - 主机：`localhost`
   - 端口：`5432`
   - 用户名：`postgres`
   - 密码：`your_password_here`（请在 `docker-compose.yml` 中修改）
   - 数据库：`vector_db`

4. 在数据库中启用pgvector扩展：

   ```sql
   CREATE EXTENSION vector;
   ```

5. 测试向量功能：

   ```sql
   CREATE TABLE items (id bigserial PRIMARY KEY, embedding vector(3));
   INSERT INTO items (embedding) VALUES ('[1,2,3]'), ('[4,5,6]');
   SELECT * FROM items ORDER BY embedding <-> '[3,1,2]' LIMIT 5;
   ```

### 直接使用 Docker

```bash
docker run -d \
  --name pgvector-postgres \
  -e POSTGRES_DB=vector_db \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=your_password \
  -p 5432:5432 \
  mypgvector:pg17.6-alpine
```

## 推送到容器仓库

### 推送到 Docker Hub

1. **登录 Docker Hub**：

   ```bash
   docker login
   ```

   - 输入你的 Docker Hub 用户名和密码。

2. **打标签**：

   ```bash
   docker tag mypgvector:pg17.6-alpine your-username/mypgvector:pg17.6-alpine
   ```

   - 将 `your-username` 替换为你的 Docker Hub 用户名。

3. **推送镜像**：

   ```bash
   docker push your-username/mypgvector:pg17.6-alpine
   ```

4. **验证推送**：
   - 访问 https://hub.docker.com 查看你的镜像。

### 推送到国内仓库（推荐，网络更稳定）

#### 阿里云容器镜像服务

1. **登录阿里云控制台**，创建容器镜像服务实例。

2. **获取仓库地址**，格式如：`registry.cn-hangzhou.aliyuncs.com/your-namespace`

3. **登录仓库**：

   ```bash
   docker login registry.cn-hangzhou.aliyuncs.com
   ```

4. **打标签**：

   ```bash
   docker tag mypgvector:pg17.6-alpine registry.cn-hangzhou.aliyuncs.com/your-namespace/mypgvector:pg17.6-alpine
   ```

5. **推送镜像**：

   ```bash
   docker push registry.cn-hangzhou.aliyuncs.com/your-namespace/mypgvector:pg17.6-alpine
   ```

#### 腾讯云容器镜像服务

类似阿里云，仓库地址格式：`ccr.ccs.tencentyun.com/your-namespace`

1. **登录仓库**：

   ```bash
   docker login ccr.ccs.tencentyun.com
   ```

2. **打标签**：

   ```bash
   docker tag mypgvector:pg17.6-alpine ccr.ccs.tencentyun.com/your-namespace/mypgvector:pg17.6-alpine
   ```

3. **推送镜像**：

   ```bash
   docker push ccr.ccs.tencentyun.com/your-namespace/mypgvector:pg17.6-alpine
   ```

### 故障排除

- **构建失败**：检查网络连接，确保能访问GitHub。
- **推送失败**：检查仓库权限和网络连接。
- **扩展加载失败**：确保PostgreSQL版本兼容（17.6）。

## 自定义配置

- 修改 `docker-compose.yml` 中的环境变量来自定义数据库设置。
- 如需修改pgvector版本，编辑 `Dockerfile` 中的 `git clone` 命令。

## 许可证

此项目基于 PostgreSQL 和 pgvector 的许可证。
