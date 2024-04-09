# 启动步骤

## 启动MySQL

```bash
docker compose up -d
```

## 步骤

- 创建bin-log存放路径

```bash
docker exec -it mysql bash
mkdir -p /data/log/mysql
```

- 修改权限

```bash
chown -R mysql:mysql /data/log/mysql

- 退出容器

```bash
exit
```

- 将配置文件复制到容器中

```bash
docker cp my.cnf mysql:/etc/mysql/my.cnf
```
