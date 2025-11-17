# CloudBeaver

CloudBeaver 是一个基于 Web 的数据库管理工具，支持多种数据库（PostgreSQL、MySQL、MariaDB、SQL Server、Oracle、DB2、Firebird、SQLite、H2 等）。

## 快速启动

```bash
cd cloudbeaver/
docker compose up -d
```

## 服务访问

- **Web 界面**: http://localhost:8978
- **首次访问**: 需要创建管理员账户

## 端口映射

| 宿主机端口 | 容器端口 | 说明 |
|-----------|---------|------|
| 8978      | 8978    | CloudBeaver Web 界面 |

## 数据持久化

- **工作空间数据**: `cloudbeaver_workspace` 命名卷 → `/opt/cloudbeaver/workspace`
  - 存储数据库连接配置
  - 用户设置和权限
  - 查询历史记录

## 初始配置

1. 首次访问 http://localhost:8978 时，会引导你创建管理员账户
2. 设置管理员用户名和密码（生产环境请使用强密码）
3. 配置服务器参数（可选）：
   - 会话过期时间
   - 匿名访问权限
   - 启用的身份验证提供程序

## 连接数据库

### 方式一：连接本地数据库

如果数据库运行在宿主机或其他容器中：

- **宿主机数据库**: 使用 `host.docker.internal` 作为主机名（Windows/Mac）或宿主机 IP（Linux）
- **其他容器数据库**: 使用容器名称或在同一 Docker 网络中的服务名

### 方式二：与数据库容器在同一网络

修改 [docker-compose.yml](docker-compose.yml) 添加外部网络：

```yaml
services:
  cloudbeaver:
    # ... 其他配置
    networks:
      - my_database_network

networks:
  my_database_network:
    external: true
```

然后可以直接使用数据库容器名作为主机名连接。

## 常用命令

```bash
# 查看服务状态
docker compose ps

# 查看实时日志
docker compose logs -f

# 重启服务
docker compose restart

# 停止服务
docker compose down

# 停止并删除数据
docker compose down -v
```

## 环境变量配置

可在 [docker-compose.yml](docker-compose.yml) 中添加以下环境变量：

```yaml
environment:
  - TZ=Asia/Shanghai          # 时区设置
  - CB_SERVER_NAME=CloudBeaver # 服务器名称
  - CB_SERVER_URL=http://localhost:8978  # 服务器 URL
  - CB_ADMIN_NAME=admin       # 预设管理员用户名（仅首次启动）
  - CB_ADMIN_PASSWORD=admin   # 预设管理员密码（仅首次启动）
```

## 版本说明

- **镜像**: `dbeaver/cloudbeaver:latest`
- **官方仓库**: https://github.com/dbeaver/cloudbeaver
- **官方文档**: https://dbeaver.com/docs/cloudbeaver/

## 已知问题

1. **连接本地数据库**：
   - Windows/Mac: 使用 `host.docker.internal`
   - Linux: 需要使用宿主机实际 IP 地址

2. **权限问题**：
   - 如需修改工作空间文件权限，进入容器：`docker exec -it cloudbeaver bash`

3. **内存配置**：
   - 默认 JVM 堆内存为 2GB
   - 如需调整，添加环境变量：`JAVA_OPTS=-Xmx4g`

## 安全建议

- 生产环境请修改默认管理员密码
- 建议配置 HTTPS 反向代理（如 Nginx）
- 启用身份验证并定期审核用户权限
- 定期备份 `cloudbeaver_workspace` 卷数据

## 参考资料

- [官方 Docker 部署文档](https://github.com/dbeaver/cloudbeaver/wiki/CloudBeaver-Community-deployment-from-docker-image)
- [Docker Hub 镜像](https://hub.docker.com/r/dbeaver/cloudbeaver)
