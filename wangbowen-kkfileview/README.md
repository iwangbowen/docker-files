# KKFileView

## 水印配置

测试发现，KKFileView的水印配置通过`docker-compose.yml`中的环境变量传递无效，必须通过配置文件`application.properties`进行配置。采用配置文件的方式，需要将配置文件挂载到容器中。这种方式也让配置更加灵活，因为`KKFileView`的配置项较多，直接通过环境变量传递不太方便，同时`KKFileView`本身也支持动态刷新配置文件，修改配置后无需重启容器。

## 镜像使用

官方镜像更新不及时，`docker-compose.yml`的镜像使用了自己修改过代码，自己构建的镜像。仓库地址：https://github.com/iwangbowen/kkFileView

- `wangbowen/kkfileview:4.4.2`基于`newer_libreoffice`分支的提交https://github.com/iwangbowen/kkFileView/commit/f8fe4421ebb2493b81876d414f682030e1bd6d5b，解决带有批注的`doc`和`docx`文档无法预览的问题。

- `wangbowen/kkfileview:4.4.1`基于`my_custom`分支制作，通过移除`docx`文档的批注来实现预览。因为采用了`newer_libreoffice`分支的方法来解决批注的问题，这一临时解决问题的分支后期应该不会维护了。
