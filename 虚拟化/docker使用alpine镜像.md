## docker使用alpine镜像

http://www.bubuko.com/infodetail-3404471.html



## alpine介绍

### alpine简要介绍

 [Alpine](https://alpinelinux.org/) **的意思是“高山的”，比如 Alpine plants高山植物，Alpine skiing高山滑雪、the alpine resort阿尔卑斯山胜地。**

***\*特点\**** 

1. **小巧**：基于[Musl libc](https://www.musl-libc.org/)和[busybox](https://busybox.net/)，和busybox一样小巧，最小的Docker镜像只有5MB；
2. **安全**：面向安全的轻量发行版；
3. **简单**：提供APK包管理工具，软件的搜索、安装、删除、升级都非常方便。
4. **适合容器使用**：由于小巧、功能完备，非常适合作为容器的基础镜像。

## alpine镜像的使用

**官方 Alpine 镜像的文档：**http://gliderlabs.viewdocs.io/docker-alpine/

```
docker pull alpine
docker run -it --name myalpine alpine:v1
```

### 下载镜像

　　**1. 在线在docker官方仓库下载** **仓库在国外，不好拉取镜像**

 

```
#查看官方alpine镜像
docker search  alpine

#拉取镜像
docker  image pull  itsthenetework/nfs-server-alpine

# 导入nginx镜像(推荐使用)
docker load  -i  docker_nginx.tar.gz
```

　　2. **离线下载** [alpine-3.10下载地址](https://pan.baidu.com/s/1daXLYwE1p8sY4pHgY1oljA) [alpine-3.11下载地址](https://pan.baidu.com/s/15JLpliY5LTu1Ea0kDOC2rA)

```
# 导入nginx镜像(推荐使用)
docker load  -i  docker_nginx.tar.gz
```

### 更新镜像源

　　**因为官方源在国外，我们改成国内的**[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/alpine/)

```
docker run -it -p 80:80 alpine:latest 
/ # sed -i ‘s/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g‘ /etc/apk/repositories 
/ # apk update
```

　　**安装nginx**

```
/ # apk add nginx 
/ # mkdir /run/nginx
/ # nginx 
```

　　**注：** **打开网页出现404原因 答：没有原因，默认配置文件默认写的就是显示404**

```
return 404;
#替换
root /html;
index index.html;
```

　　**提交成镜像**

```
docker commit  <alpineID>  nginx_alpine:v1 
docker run -d -p 81:80 nginx_alpine:v1 nginx -g ‘daemon off;‘
```

## alpine安装kod

　　1. **启动alpine容器**

```
docker run -it -p 80:80 alpine:v1  
/ # sed -i ‘s/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g‘ /etc/apk/repositories 
/ # apk update
/ # apk add nginx 
/ # mkdir /run/nginx
/ # nginx 
/ # apk add php7 php7-fpm php7-opcache php7-curl php7-gd php7-mbstring php7-mysqli php7-json php7-iconv php7-exif php7-ldap php7-pdo php7-session php7-xml
```

**安装 rc-service**

```
apk add openrc --no-cache
```

***\*启动nginx\****

```
service nginx start
#或者rc-service nginx start
```

**遇到问题**

***\*WARNING: nginx is already starting\****

 

```
/#  service nginx status
* You are attempting to run an openrc service on a
 * system which openrc did not boot.
 * You may be inside a chroot or you may have used
 * another initialization system to boot this system.
 * In this situation, you will get unpredictable results!
 * If you really want to do this, issue the following command:
 * touch /run/openrc/softlevel
/#  touch /run/openrc/softlevel
touch: /run/openrc/softlevel: No such file or directory
/#  mkdir -p /run/openrc
/#  touch /run/openrc/softlevel
/#  service nginx status
 * status: stopped
/#  service nginx start
 * WARNING: nginx is already starting
/#  /sbin/openrc
 * Caching service dependencies ...                                                 [ ok ]
/#  rc-service nginx start
 * /run/nginx: creating directory
 * /run/nginx: correcting owner
 * Starting nginx ...                                                               [ ok ]
```

**启动php-fpm**

```
vi  /etc/php7/php-fpm.d/www.conf  
所属主和所著组为nginx
/#  service php-fpm7 start
 * Checking /etc/php7/php-fpm.conf ...
 * /run/php-fpm7: creating directory
 * Starting PHP FastCGI Process Manager ...                                        [ ok ]
```

**加入启动服务**

```
/#  rc-update add nginx default
/#  rc-update add php-fpm7 default
```

**安装可道云**

```
vim  /etc/nginx/conf.d/default.conf  
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    location / {
        root /html;
        index index.php  index.html;
    include /etc/nginx/conf.d./*.conf;
    location ~ \.php$ {
        root        /html;
          fastcgi_pass    127.0.0.1:9000;
    fastcgi_index    index.php;
    fastcgi_param    SCRIPT_FILENAME    /html$fastcgi_script_name;
    include        fastcgi_params;
    }
    }
    # You may need this to prevent return 404 recursion.
    location = /404.html {
        internal;
    }
}

nginx -t
```

**创建站点目录**

```
mkdir /html
cd /html
wget http://static.kodcloud.com/update/download/kodexplorer4.40.zip
chown -R nginx:nginx .
#重启nginx
service nginx restart 
service php-fpm7 restart
```

**启动脚本文件**

```
vim /init.sh
service nginx start
service nginx restart
service php-fpm7 start
service php-fpm7 restart 
tail -f /etc/hosts
```

**注：** **使用脚本时，nginx和php-fpm7启动一次失败，所以在这里在重启一次**

**提交镜像**

 

```
 docker commit  optimistic_hermann  alpine_kod:v1 
```

**验证镜像**

```
docker run -d -p 80:80 alpine_kod:v1 /bin/sh /init.sh
```

**登录网站`10.0.0.12:80`查看**