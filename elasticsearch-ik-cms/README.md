# 启动问题记录

## 说明

该项目基于[ChestnutCMS](https://github.com/liweiyi/ChestnutCMS)项目，需要启动CMS项目，才能在Elasticsearch中定时去访问配置中`IKAanlyzer.cfg.xml`配置的更新地址。

## 启动后无法访问

[docker elasticsearch8.4.2 启动后无法访问的问题备忘](https://blog.csdn.net/rwzhang/article/details/127128759)，参考本文增加了环境变量参数。

## 启动后添加中文分词器（如有需要）

使用的是[ChestnutCMS](https://github.com/liweiyi/ChestnutCMS)中配置的中文分词器。参考项目里对应的`DockerFile`和`docker-compose.yml`文件。
