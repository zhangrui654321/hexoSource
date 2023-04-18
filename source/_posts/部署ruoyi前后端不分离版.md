# 部署ruoyi前后端不分离版

### 项目目录

![](images/2022-08-18.png)

#tmp存放临时安装包
mkdir -p /data/tmp
#service存放软件环境
mkdir -p /data/service
#gitee存放代码版本控制库
mkdir -p /data/gitee

### 安装java

java
cd /data/tmp
tar -zxvf jdk-8u212-linux-x64.tar.gz
mv jdk1.8.0_212 /data/service
cd /data/service
ll
vim /etc/profile
export JAVA_HOME=/data/service/jdk1.8.0_212
export PATH=$PATH:$JAVA_HOME/bin
source /etc/profile

### 安装maven

cd /data/tmp
tar -zxvf apache-maven-3.8.5-bin.tar.gz
mv apache-maven-3.8.5 /data/service
cd /data/service
ll
vim /etc/profile
export MAVEN_HOME=/data/service/apache-maven-3.8.5
export PATH=$PATH:$MAVEN_HOME/bin
source /etc/profile
vim /data/service/apache-maven-3.8.5/conf/setting.xml
#配置本地仓库
<localRepository>/data/service/apache-maven-3.8.5/repository</localRepository>
#配置阿里云镜像
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>*</mirrorOf>
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>



### 安装git

yum -y groupinstall "Development Tools"
yum install wget unzip gettext-devel openssl-devel perl-CPAN perl-devel zlib-devel libcurl-devel expat-devel -y
cd /data/tmp
tar zxvf git-2.18.0.tar.gz
cd git-2.18.0
./configure --prefix=/data/service/git
make && make install
vim /etc/profile
export GIT_HOME=/data/service/git
export PATH=$PATH:$GIT_HOME/bin
source /etc/profile

### git公钥，和下载项目

git密匙
ssh-keygen -t rsa -C "15172537049@163.com"
cat ~/.ssh/id_rsa.pub
//粘到公匙
ssh -T git@gitee.com
cd /data/gitee
git clone git@gitee.com:y_project/RuoYi.git

### docker安装

docker安装
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/edge/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
yum install docker-ce docker-ce-cli -y
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://plb9xzjh.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

### mysql安装

docker pull mysql:5.7.30
mkdir -p /root/mysql/data /root/mysql/logs /root/mysql/conf
touch /root/mysql/conf/my.cnf
//密码是admin，-v则是挂载 –name容器名字，作为后面的启动删除名字
docker run -p 3306:3306 --rm --name mysql -v /root/mysql/conf:/etc/mysql/conf.d -v /root/mysql/logs:/var/log/mysql -v /root/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=admin -d mysql:5.7.30
docker start mysql
//进入容器
docker exec -it mysql /bin/bash
#登录docker容器（方式参考上方文档）后登录mysql
mysql -uroot -padmin
#修改登录者的权限
GRANT ALL ON *.* TO 'root'@'%';
#刷新命令生效
 flush privileges;

### 修改项目配置

cd /data/gitee/RuoYi/ruoyi-admin/src/main/resources/
#修改application.yml的项目启动端口号与文件上传路径
vim application.yml
#修改application-druid.yml的数据源
vim application-druid.yml

### 打包启动

cd /data/gitee/RuoYi/
mvn install
mkdir -p /data/app/ruoyi-admin
cp /data/gitee/RuoYi/ruoyi-admin/target/ruoyi-admin.jar /data/app/ruoyi-admin/ruoyi-admin.jar
cd /data/app/ruoyi-admin/
#后台启动项目
nohup java -jar ruoyi-admin.jar &
#查看项目运行日志
tail -f nohup.out

### 开放端口

重启防火墙
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --reload



### 