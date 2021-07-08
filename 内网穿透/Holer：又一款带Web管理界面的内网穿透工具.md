## Holer：又一款带Web管理界面的内网穿透工具

https://www.moerats.com/archives/991/



**说明：**博主去年介绍过一个免费的内网穿透工具[Holer](https://www.moerats.com/archives/716/)，它可以将局域网服务器代理到公网的内网穿透工具，支持转发基于`TCP`等协议的报文，不过那时候服务端并未开源，由作者免费提供服务，现在服务端代码已经开源了，而且带`Web`管理面板，该类似面板博主介绍过不少了，这里就大概说下，我们就可以拿来自建一个内网穿透服务器，使用效果还不错。

## 截图

[![请输入图片描述](images/holer(1).png)](https://www.moerats.com/usr/picture/holer(1).png)
[![请输入图片描述](images/holer(2).png)](https://www.moerats.com/usr/picture/holer(2).png)
[![请输入图片描述](images/holer(3).png)](https://www.moerats.com/usr/picture/holer(3).png)

## 安装服务端

**Github地址：**https://github.com/Wisdom-Projects/holer

**支持系统：**`Windows`、`Linux`系统，这里只说`Linux`搭建，建议直接`Debain`。

**说明：**由于该面板使用的`JAVA`，所以还是比较消耗内存的，如果内存太小，建议先加一点虚拟内存，可以使用`Swap`一键脚本→[传送门](https://www.moerats.com/archives/722/)。

**1、安装JAVA**

```
#CentOS系统
yum install java-1.8.0-openjdk -y

#Debian/Ubuntu系统
apt update
apt install default-jdk -y
```

**2、安装Mysql**

```
#CentOS 6系统
rpm -ivh http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
yum install mysql-community-server -y
service mysqld start
chkconfig mysqld on

#CentOS 7系统
rpm -ivh http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
yum install mysql-community-server -y
systemctl start mysqld
systemctl enable mysqld

#Debian/Ubuntu系统
apt install mysql-server -y
```

如果`Debian`或`Ubuntu`在安装期间有弹出窗口要你输入密码就设置一个，没有的话密码就是空格。

修改数据库密码：

```
#CentOS系统，第一行登录数据库的时候直接Enter跳过，第二行moerats为要修改的密码，自行修改
mysql -u root -p
mysql> set password=password("moerats");
mysql> exit;

#Debian、Ubuntu系统，第一行登录数据库的时候直接Enter跳过，第二行moerats为要修改的密码，自行修改
mysql -u root -p
mysql> UPDATE mysql.user SET authentication_string=PASSWORD('moerats'), PLUGIN='mysql_native_password' WHERE USER='root';
mysql> exit;
```

最后修改过密码的还需要重启数据库：

```
#CentOS系统
service mysqld restart

#Debian和Ubuntu系统
systemctl restart mysql
```

此时`Mysql`算是安装完成了。

**3、安装源码**
安装`unzip`：

```
#CentOS系统
yum install unzip -y

#Debian和Ubuntu系统
apt install unzip -y
```

下载源码：

```
wget https://github.com/wisdom-projects/holer/releases/download/v1.1/holer-server-1.1.zip
unzip holer-server-1.1.zip && rm -rf holer-server-1.1.zip
#移动到opt目录，然后进入到源码文件夹
mv holer-server /opt/holer && cd $_
#修改配置文件
nano resources/application.yaml
```

关键配置如下：

```
#运行端口
server:
  port: 600

#Mysql数据库用户名和密码
spring:
  datasource:
    username: root
    password: moerats

#域名和nginx目录，可以直接全部删掉，用ip不需要，域名的话，有点不好用
holer
  domain:
    name: your-domain.com
  nginx:
    #home: /usr/local/nginx
    home: C:/nginx-1.14.2
```

修改后使用`Ctrl+x`、`y`保存退出，或者可以直接使用`FTP`等工具直接编辑。

再修改管理员用户名和密码，使用命令：

```
nano resources/conf/holer-data.sql
```

`admin`和`admin123`为管理员用户名和密码，自行修改，修改完成后同样的使用`Ctrl+x`、`y`保存退出。

最后启动：

```
chmod +x holer
./holer start
```

如果想开机自启的话，这里可以建一个简单的`systemd`配置文件，且不适用`CentOS 6`，使用命令：

```
#将以下代码一起复制到SSH运行
cat > /etc/systemd/system/holer.service <<EOF
[Unit]
Description=holer
After=network.target

[Service]
Type=simple
ExecStart=$(command -v java) -server -Xms256m -Xmx512m -jar holer-server-1.1.jar
WorkingDirectory=/opt/holer
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

然后启动并设置开机自启：

```
systemctl start holer
systemctl enable holer
```

然后就可以使用`ip:600`访问管理界面了，具体端口以你修改的为准。

然后`CentOS`系统建议关闭防火墙使用，或者打开部分端口也行，关闭命令：

```
#CentOS 6系统
service iptables stop
chkconfig iptables off

#CentOS 7系统
systemctl stop firewalld
systemctl disable firewalld
```

像阿里云等服务器，还需要去安全组那里开放下端口。

## 客户端使用

首先我们需要去用户列表新建一个用户，然后再去端口映射选择该用户，新建一个穿透规则，这里根据需求自行选择，然后设置好时长。

然后就可以直接在客户端使用了，一般客户端有`JAVA`和`GO`版，使用`JAVA`的话，需要先安装`JAVA`环境，所以这里直接选择`GO`版本，简单粗暴。

首先根据直接的系统和架构下载指定的`GO`版客户端，每个压缩包里都包含`32`位和`64`位，下载地址→[传送门](https://github.com/wisdom-projects/holer/tree/master/Binary/Go)。

这里拿我们常见的`Linux`服务器架构来说，直接使用命令：

```
#下载并解压
wget https://github.com/wisdom-projects/holer/raw/master/Binary/Go/holer-linux-x86.tar.gz
tar -zxvf holer-linux-x86.tar.gz

#32位启动，分别为访问秘钥和服务端ip地址
nohup ./holer-linux-386 -k 7aa8d973bc8e40 -s ip地址 &
#64位启动
nohup ./holer-linux-amd64 -k 7aa8d973bc8e40 -s ip地址 &
```

如果是`Windows`系统，先把压缩包下载并解压到`D`盘根目录，然后按住`Win+R`，输入`cmd`进入命令窗口，使用命令：

```
#进入到D盘根目录
cd D:\

#32位启动，分别为访问秘钥和服务端ip地址
.\holer-windows-386.exe -k 7aa8d973bc8e40 -s ip地址
#64位启动
.\holer-windows-amd64.exe -k 7aa8d973bc8e40 -s ip地址
```

到这里基本上就运行成功了。

## 域名反代

如果你想使用域名来配置服务器面板的话，就需要安装`Web`服务器了，这里就直接使用`Nginx`。

**1、安装Nginx**

```
#CentOS 6系统
rpm -ivh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm
yum install nginx -y
service nginx start
chkconfig nginx on

#CentOS 7系统
rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
yum install nginx -y
systemctl start nginx
systemctl enable nginx

#Debian/Ubuntu系统
apt install nginx -y
```

**2、申请SSL证书**

这里就使用简单粗暴的`webroot`方式签发`Let's Encrypt`证书，首先解析好域名并生效。

安装`letsencrypt`：

```
#CentOS系统
yum install letsencrypt -y

#Debian/Ubuntu系统
apt install letsencrypt -y
```

申请`SSL`证书：

```
#CentOS系统
letsencrypt certonly --webroot -w /usr/share/nginx/html --domain www.moerats.com

#Debian/Ubuntu系统
letsencrypt certonly --webroot -w /var/www/html --domain www.moerats.com
```

请替换成自己域名后运行，期间会要你输入邮箱和`A`选项啥的，申请后证书文件在`/etc/letsencrypt/live`。

**3、新建conf文件**

```
#将下面域名修改成自己的，然后证书路径也修改下，再一起复制进SSH客户端运行
cat > /etc/nginx/conf.d/holer.conf << 'EOF'
server {
    listen 443;
    server_name www.moerats.com;    
    ssl on;
    ssl_certificate /etc/letsencrypt/live/www.moerats.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.moerats.com/privkey.pem;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers "EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5";
    ssl_session_cache builtin:1000 shared:SSL:10m;
    charset utf-8;
    location /{
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;

        client_max_body_size       1024m;
        client_body_buffer_size    128k;

        proxy_connect_timeout      90;
        proxy_send_timeout         90;
        proxy_read_timeout         90;
        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;
        proxy_pass http://127.0.0.1:600/;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
server
    {
        listen 80;
        server_name www.moerats.com;
        rewrite ^(.*) https://www.moerats.com$1 permanent;
    }
EOF
```

重启`Nginx`生效：

```
systemctl restart nginx
```

最后连接的时候，就可以填域名了。

最后要是觉得搭建服务器麻烦，或者不想搭建的，可以使用作者提供的免费服务，更多使用方法移至→[传送门](http://blog.wdom.net/)。

## 相关面板

- [一款基于Frp的Web管理面板：FrpMgr安装及使用](https://www.moerats.com/archives/958/)
- [一款带Web面板的轻量级、高性能内网穿透工具：nps](https://www.moerats.com/archives/891/)
- [一款带Web管理面板的内网穿透工具：lanproxy](https://www.moerats.com/archives/727/)