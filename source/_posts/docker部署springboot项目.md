# docker部署springboot项目

本文主要介绍docker如何部署springboot项目

#### 创建一个springboot项目，打包成jar包

该项目端口为8081，logback日志目录为/home/docker/logs

![img](https://img2022.cnblogs.com/blog/1899220/202204/1899220-20220415153310412-1698748435.png)

#### 准备一台搭建好了docker的linux服务器

linux搭建docker攻略移步：https://www.cnblogs.com/lixianguo/p/13254950.html
创建/home/docker文件夹，将打包的jar包上传
创建/home/docker/logs文件夹存放日志文件，该路径与项目中logback中设置的一致
创建Dockerfile,内容如下

```dockerfile
#指定基础镜像，不需要另外安装jdk
FROM java:8
#维护者
MAINTAINER lxg
#将本地文件添加到容器中，并更名为myproject.jar
COPY springboot-docker-1.0-SNAPSHOT.jar myproject.jar
#指定访问端口，与yml文件中的端口一致
EXPOSE 8081
#容器启动时，运行该程序
ENTRYPOINT ["java", "-jar", "myproject.jar"]
```

目录截图如下

![img](https://img2022.cnblogs.com/blog/1899220/202204/1899220-20220415154509479-1294761548.png)

#### 使用命令构建镜像

最后空格和"."不可忽略

```
docker build -t myproject .
```

![img](https://img2022.cnblogs.com/blog/1899220/202204/1899220-20220415160735560-359695528.png)

#### 创建并启动容器

```shell
docker run -p 8080:8081 --name myproject \
> -v /home/docker/logs:/home/docker/logs \
> -d myproject
```

![img](https://img2022.cnblogs.com/blog/1899220/202204/1899220-20220415162800983-219739236.png)

8080是外界访问的端口，可以自定义，8081是Dockerfile中定义的端口。
-v是为了将容器中的日志目录挂载出来，冒号之前是linux宿主机自己创建的目录，可以自定义。后面的是logback文件中定义的输出日志的目录。

![img](https://img2022.cnblogs.com/blog/1899220/202204/1899220-20220415162939731-510580576.png)