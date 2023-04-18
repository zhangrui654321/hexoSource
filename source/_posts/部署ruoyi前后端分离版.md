# 部署ruoyi前后端分离版

### 此步骤在不分离版后延续

### redis启动

docker pull redis:latest
docker run -d --rm -v /root/redis/data:/data --name redis -p 6379:6379 redis  redis-server  --appendonly yes

### nodejs

cd /data/tmp
tar -zxvf node-v14.15.5-linux-x64.tar.gz
mv node-v14.15.5-linux-x64 /data/service
cd /data/service
ll
配置环境变量
vim /etc/profile
export NODEJS_HOME=/data/service/node-v14.15.5-linux-x64
export PATH=$PATH:$NODEJS_HOME/bin
source /etc/profile

### nginx

cd /data/tmp
tar zxvf nginx-1.18.0.tar.gz
cd nginx-1.18.0
./configure --prefix=/data/service/ngnix
make && make install
/data/service/ngnix/sbin/nginx

### git拉代码并修改配置文件，打包

cd /data/gitee
git clone git@gitee.com:y_project/RuoYi-Vue.git
cd /data/gitee/RuoYi-Vue/ruoyi-admin/src/main/resources/
#修改application.yml的项目启动端口号与文件上传路径
vim application.yml
#修改application-druid.yml的数据源
vim application-druid.yml
cd /data/gitee/RuoYi-Vue/
mvn clean install -pl com.ruoyi:ruoyi-admin -am
mkdir -p /data/app/ruoyi-vue
cp /data/gitee/RuoYi-Vue/ruoyi-admin/target/ruoyi-admin.jar /data/app/ruoyi-vue/ruoyi-admin.jar
cd /data/app/ruoyi-vue/
#后台启动项目
nohup java -jar ruoyi-admin.jar &
#查看项目运行日志
tail -f nohup.out

### 启动前端项目

1、下载依赖并打包
cd /data/gitee/RuoYi-Vue/ruoyi-ui
npm install --registry=https://registry.npmmirror.com
npm install
mkdir /data/app/ruoyi-ui
mv dist/* /data/app/ruoyi-ui/

### 配置nginx

此处/代表根目录 root配置的dist的明确地址

/prod-api/配置的是转发路径，及前端请求都转发到 localhost:18081地址

​    location / {
​            root   /data/app/ruoyi-ui/dist;  此处为前端dist路径
​            index  index.html index.htm;
​        }
​	location /prod-api/ {
  proxy_set_header Host $http_host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header REMOTE-HOST $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_pass http://localhost:18081/; 此处为后端项目路径
}
重启nginx
/data/service/ngnix/sbin/nginx -s reload
访问 192.168.2..6:80