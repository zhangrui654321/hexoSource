---
 wms部署
---

##redis部署


```bash
setenforce 0
docker pull redis
## 创建目录
mkdir -p /home/redis/conf
## 创建文件
touch /home/redis/conf/redis.conf
vim /home/redis/conf/redis.conf
docker run --name redis -p 6379:6379 \
-v /home/redis/data:/data \
-v /home/redis/conf/redis.conf:/etc/redis/redis.conf \
-d redis:latest redis-server /etc/redis/redis.conf
```

###redis配置文件，主要是密码和配置持久化
```bash
appendonly yes
requirepass zr525010
```


##mysql部署
```bash
mkdir -p /root/mysql/data /root/mysql/logs /root/mysql/conf
touch /root/mysql/conf/my.cnf
docker run \
--name mysql \
-d \
-p 3306:3306 \
--restart unless-stopped \
-v /root/mysql/conf:/etc/mysql/conf.d -v /root/mysql/logs:/var/log/mysql -v /root/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=admin \
mysql:5.7.37

```

##java部署
```bash
tar -zxvf jdk-8u161-linux-x64.tar.gz
vim /etc/profile
export JAVA_HOME=/data/runtime/jdk1.8.0_161
export PATH=$PATH:$JAVA_HOME/bin
#让配置生效
source /etc/profile
```

###启动项目
```bash
#启动项目
nohup java -jar yudao-server.jar \
--spring.profiles.active=test \
--server.port=8091 \
>/dev/null 2>&1 &
#停止项目
ps -ef | grep yudao-server.jar | grep java | awk '{print $2}' | xargs kill -9

```

##nginx部署
```bash
yum install gcc-c++
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel
#进入官网下载
nginx.org
cd /usr/local
mkdir nginx
tar -zxvf nginx-1.16.1.tar.gz
cd /usr/local/nginx
./configure --prefix=/usr/local/nginx
make && make install
启动nginx
cd /usr/local/nginx
./sbin/nginx
重启
./sbin/nginx -s reload
```

###nginx配置文件

```bash
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    gzip on;
    gzip_min_length 1k;     # 璁剧疆鍏佽鍘嬬缉鐨勯〉闈㈡渶灏忓瓧鑺傛暟
    gzip_buffers 4 16k;     # 鐢ㄦ潵瀛樺偍 gzip 鐨勫帇缂╃粨鏋
    gzip_http_version 1.1;  # 璇嗗埆 HTTP 鍗忚鐗堟湰
    gzip_comp_level 2;      # 璁剧疆 gzip 鐨勫帇缂╂瘮 1-9銆 鍘嬬缉姣旀渶灏忎絾鏈€蹇紝鑰9 鐩稿弽
    gzip_types gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript; # 鎸囧畾鍘嬬缉绫诲瀷
    gzip_proxied any;       # 鏃犺鍚庣鏈嶅姟鍣ㄧ殑 headers 澶磋繑鍥炰粈涔堜俊鎭紝閮芥棤鏉′欢鍚敤鍘嬬缉

    server {
        listen       7000;
        server_name  192.168.2.2; ## 閲嶈锛侊紒锛佷慨鏀规垚浣犵殑澶栫綉 IP/鍩熷悕

        location / { ## 鍓嶇椤圭洰
            root   /data/project/web/dist;
            index  index.html index.htm;
            try_files $uri $uri/ /index.html;
        }
         location /proxy-api/ {
            proxy_pass http://192.168.2.2:8091/;
         }


    }

}
```
