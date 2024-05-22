# 启动步骤

## 启动MySQL

```bash
docker compose up -d
```

## 步骤

- 将配置文件复制到容器中

```bash
docker cp my.cnf mysql8:/etc/mysql/my.cnf
```
