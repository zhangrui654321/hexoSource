# docker部署jenkins和gitlab总结

## jenkins安装

### 1拉取docker镜像并启动

docker pull jenkinsci/blueocean
docker run -e JAVA_OPTS="-Dhudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE_CSRF_PROTECTION=true" --name jenkins -u root --rm  -d -p 7005:8080 -p 50000:50000 -v /data/service/jenkins:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v /data/service/apache-maven-3.8.5:/usr/local/maven -v /data/service/jdk1.8.0_212:/usr/local/jdk jenkinsci/blueocean

注意，这里的run -e JAVA_OPTS="-Dhudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE_CSRF_PROTECTION=true" --name 是用来允许跨网站请求伪造，方便后来部署webhook

--rm代表stop容器后就删除了容器 -v代表挂载，这里分别挂载jenkins目录，maven目录，jdk目录 -name代表容器名是jenkens ，这个可以直接通过docker rm Jenkins就删除了jenkins目录 -p代表端口映射，这里是代表容器在7005端口启动

启动后访问 ip:7005

### 2查看密码登录

查看密码 ，目录文件为/data/service/Jenkins

cat /data/service/jenkins/secrets/initialAdminPassword

### 3配置环境变量和git

设置全局工作变量 ，git和java是有的，不用配置

这里只用配置maven环境变量，地址是docker容器内的

![](images/20220822-1.png)

 首先生成公钥

进入jinkins容器后执行

docker exec -it jenkins bash

ssh-keygen -t rsa -C ‘15019474951@163.com’

cat ~/.ssh/id_rsa.pub

#### 将公钥填入

![](images/20220822-2.png)

新建一个自由风格项目

![](images/20220822-3.png)

这里将刚才的公钥填入

### 4安装ssh插件并执行远程ssh连接

搜索 Publish over SSH 插件安装，搜索 Maven Integration 插件安装

在系统设置下配置远程连接，这里我配置主机地址,找到**Publish over SSH**

![](images/20220822-5.png)

接着构建项目

主要流程是先打包项目，然后把git目录下的项目的包发送

![](images/20220822-8.png)

这里首先是Soure set这里代表git根目录下的地址

remove prefix代表去除项目前缀，可以不配置

关键是remote directory 这里是用户的目录地址下的，这是个坑，即如果是root目录，则是/root/data/test，即用户打包的东西会传到这个目录下

这里的shell命令就是删除原来的进程然后构建

## webhook配置

#### webhook配置

\1.   安装**Generic Webhook Trigger**插件

\2.   加入git的公钥到jenkins

\3.   生成key  ssh-keygen -t rsa -C '15172537049@163.com‘ cat ~/.ssh/id_rsa.pub

\4.    将构建触发器勾上!![](images/20220822-9.png)

\5.                 

 

配置apitoken，这里是后面APItoken的地址

这里是在用户配置哪里，点设置，进去之后就能看到apitoken

![](images/20220822-10.png)

先找到设置然后打勾

![](images/20220822-11.png)

\1.   找到WebHooks配置界面，在POST地址框中输入如下格式内容：

\2.  ![](images/20220822-12.png)

\3.   http://<User ID>:<API Token>@<Jenkins IP地址>:端口/generic-webhook-trigger/invoke 

\4.   添加url

http://root:1148e42061461a8d68883794319c3dcd82@192.168.2.2:7005/generic-webhook-trigger/invoke

## 配置gitlab

### 拉docker 并启动 

docker pull gitlab/gitlab-ce
docker run -d  -p 443:443 -p 81:80 -p 222:22 --name gitlab --restart always -v /root/gitlab/config:/etc/gitlab -v /root/gitlab/logs:/var/log/gitlab -v /root/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce

#### 配置启动路径

gitlab.rb文件内容默认全是注释

$ vim /root/gitlab/config/gitlab.rb

配置http协议所使用的访问地址,不加端口号默认为80

external_url 'http://192.168.2.7'

external_url 'http://120.77.247.123'

配置ssh协议所使用的访问地址和端口

gitlab_rails['gitlab_ssh_host'] = '192.168.2.7 '
gitlab_rails['gitlab_shell_ssh_port'] = 222 # 此端口是run时22端口映射的222端口
:wq #保存配置文件并退出
重启gitlab容器



$ docker restart gitlab
GitLab占用内存非常恐怖，解决方法很简单
     修改/root/gitlab/config/gitlab/gitlab.rb 文件，将 unicorn['worker_processes'] = 2 去掉注释就可以了。在注释的情况下默认是服务器上的所有线程。

### 重置root密码

docker exec -it gitlab /bin/bash
启用docker里面gitlab的ruby

#### 注意，这里一定要等到控制台打印信息后才能输入下面的user

gitlab-rails console -e production

 找到管理员用户
user = User.where(id: 1).first
 更改密码
user.password = 'abcd1234'
user.password_confirmation = 'abcd1234'
记得保存
user.save!

### 

 

​                               











