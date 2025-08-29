# KKFileView

## 水印配置

测试发现，KKFileView的水印配置通过`docker-compose.yml`中的环境变量传递无效，必须通过配置文件`application.properties`进行配置。采用配置文件的方式，需要将配置文件挂载到容器中。这种方式也让配置更加灵活，因为`KKFileView`的配置项较多，直接通过环境变量传递不太方便，同时`KKFileView`本身也支持动态刷新配置文件，修改配置后无需重启容器。
