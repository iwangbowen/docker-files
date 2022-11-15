# Docker安装Redis

## 参考[Tutorial: Persist data in a container app using volumes in VS Code](https://learn.microsoft.com/en-us/visualstudio/docker/tutorials/tutorial-persist-data-layer-docker-app-with-vscode)

### 创建命名容器

    ```bash
    docker volume create redis-data
    ```

查看Docker实际创建的命名容器位置

    ```bash
    docker volume inspect redis-data
    ```
