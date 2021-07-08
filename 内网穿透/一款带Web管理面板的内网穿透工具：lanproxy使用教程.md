## 一款带Web管理面板的内网穿透工具：lanproxy使用教程

https://www.moerats.com/archives/727/



**说明：**博主今天找`Frp`管理面板的时候，无意间发现了`lanproxy`内网穿透工具，自带`Web`管理面板，让我们在服务端配置方便了很多，而且一个服务端可以支持多个客户端连接，看起来还不错，这里就分享下。

## 简介

`lanproxy`是一个将局域网个人电脑、服务器代理到公网的内网穿透工具，目前仅支持`tcp`流量转发，可支持任何`tcp`上层协议，可用作访问内网网站、本地支付接口调试、`SSH`访问、远程桌面等等，而且带`Web`在线管理面板，添加端口配置十分简单。

## 截图

[![请输入图片描述](images/lanproxy(1).png)](https://www.moerats.com/usr/picture/lanproxy(1).png)
[![请输入图片描述](images/lanproxy(2).png)](https://www.moerats.com/usr/picture/lanproxy(2).png)

## 相关链接

**主页地址：**https://nat.io2c.com/
**Github地址：**https://github.com/ffay/lanproxy
**发布包下载：**https://seafile.cdjxt.net/d/2e81550ebdbd416c933f/

## 服务端安装

服务端需要安装在一个有公网`IP`的服务器上，系统为`Linux/Windows`均可。

**1、安装JAVA**
`java`版本至少为`1.7`，查看命令为`java -version`，如果没安装可参考：[Linux/Windows系统安装最新版JAVA教程](https://www.moerats.com/archives/92/)。

**2、Linux系统安装**
首先下载发布包，服务端发布包下载地址：[点击进入](https://seafile.cdjxt.net/d/2e81550ebdbd416c933f)。

```
#下载最新发布包
wget -O proxy-server-0.1.zip 'https://seafile.cdjxt.net/d/2e81550ebdbd416c933f/files/?p=/proxy-server-0.1.zip&dl=1'
#解压发布包
unzip proxy-server-0.1.zip
#进入到文件夹
cd proxy-server-0.1
```

然后编辑配置文件`conf/config.properties`，参考如下：

```
server.bind=0.0.0.0

#与代理客户端通信端口
server.port=4900

#ssl相关配置
server.ssl.enable=true
server.ssl.bind=0.0.0.0
server.ssl.port=4993
server.ssl.jksPath=test.jks
server.ssl.keyStorePassword=123456
server.ssl.keyManagerPassword=123456

#这个配置可以忽略
server.ssl.needsClientAuth=false

#WEB在线配置管理相关信息
config.server.bind=0.0.0.0
config.server.port=8090
config.admin.username=admin
config.admin.password=admin
```

运行`lanproxy`：

```
cd /root/proxy-server-0.1/bin
chmod +x startup.sh
./startup.sh
```

然后打开地址`http://ip:8090`，使用上面配置中配置的用户名密码登录，进入`Web`管理面板，且配置数据存放在`~/.lanproxy/config.json`文件中。

```
#如果打不开Web界面，就需要开启防火墙，一般CentOS系统出现情况最多
#Centos 6系统
iptables -I INPUT -p tcp --dport 8090 -j ACCEPT
service iptables save
service iptables restart

#CentOS 7系统
firewall-cmd --zone=public --add-port=8090/tcp --permanent 
firewall-cmd --reload
```

**3、Windows系统安装**
方法参考上面，只是启动的时候双击`bin`文件夹里的`startup.bat`即可运行。

## 客户端使用

客户端一般安装在一个内网的`VPS`服务器或`Windows`电脑上使用。这里说下`JAVA`和非`JAVA`两个客户端的使用方法，客户端下载地址：[点击进入](https://seafile.cdjxt.net/d/2e81550ebdbd416c933f)。

**1、配置服务端**
首先我们通过`http://ip:8090`进入服务端`Web`管理界面，先添加客户端，名称随便填。
[![请输入图片描述](images/lanproxy(3).png)](https://www.moerats.com/usr/picture/lanproxy(3).png)
然后点击刚刚添加的客户端名称，再添加配置，设置公网端口，后端`IP:端口`。
[![请输入图片描述](images/lanproxy(4).png)](https://www.moerats.com/usr/picture/lanproxy(4).png)
截图的配置意思是将内网的`888`端口映射到服务器的`8080`端口，也就是访问`服务器ip:8080`等于访问`内网ip:888`。

这时候基本配置好了一个客户端节点，且该节点可以供多个客户端使用。

**2、JAVA客户端使用**
本版本需要安装`java`，且版本依然至少为`1.7`，查看命令为`java -version`，如果没安装可参考：[Linux/Windows系统安装最新版JAVA教程](https://www.moerats.com/archives/92/)。

然后进入客户端下载地址，下载[proxy-java-client-0.1.zip](https://seafile.cdjxt.net/d/2e81550ebdbd416c933f/files/?p=/proxy-java-client-0.1.zip&dl=1)，再将文件解压到服务器或者`Windows`电脑上，编辑`conf/config.properties`配置文件，修改如下：

```
#与在proxy-server配置后台创建客户端时填写的秘钥保持一致；
client.key=
ssl.enable=true
ssl.jksPath=test.jks
ssl.keyStorePassword=123456

#这里填写实际的proxy-server地址；没有服务器默认即可，自己有服务器的更换为自己的proxy-server（IP）地址
server.host=lp.thingsglobal.org

#proxy-server ssl默认端口4993，默认普通端口4900
#ssl.enable=true时这里填写ssl端口，ssl.enable=false时这里填写普通端口
server.port=4993
```

最后运行`lanproxy`：

```
#运行方法可参考服务端运行步骤
linux（mac）系统：直接进入bin目录，然后运行startup.sh脚本
windows系统：直接双击bin目录下的startup.bat
```

**3、非JAVA客户端使用**
该方法可以不用安装`java`即可在客户端运行`lanproxy`，首先下载对应版本的[JAVA客户端](https://seafile.cdjxt.net/d/2e81550ebdbd416c933f/)，然后解压出来，再运行以下命令：

```
#以下需要使用的参数是服务端IP，服务端端口，客户端密匙
1、普通端口连接
#mac 64位
nohup ./client_darwin_amd64 -s SERVER_IP -p SERVER_PORT -k CLIENT_KEY &
#linux 64位
nohup ./client_linux_amd64 -s SERVER_IP -p SERVER_PORT -k CLIENT_KEY &
#windows 64 位
./client_windows_amd64.exe -s SERVER_IP -p SERVER_PORT -k CLIENT_KEY

2、SSL端口连接
#mac 64位
nohup ./client_darwin_amd64 -s SERVER_IP -p SERVER_SSL_PORT -k CLIENT_KEY -ssl true &
#linux 64位
nohup ./client_linux_amd64 -s SERVER_IP -p SERVER_SSL_PORT -k CLIENT_KEY -ssl true &
#windows 64 位
client_windows_amd64.exe -s SERVER_IP -p SERVER_SSL_PORT -k CLIENT_KEY -ssl true
```

这里单独说下`Windows`电脑使用方法，首先按住`Win+R`，输入`cmd`进入命令窗口。

```
#如果你将客户端exe文件解压到了D盘的RATS文件夹，则使用命令进入RATS文件夹
cd /d d:\RATS
#如果你是SSL端口连接，先替换自己的IP，端口，CLIENT_KEY后运行，普通端口命令参考上面
client_windows_amd64.exe -s SERVER_IP -p SERVER_SSL_PORT -k CLIENT_KEY -ssl true
```

最后客户端运行后，服务端`Web`界面的配置状态显示在线即连接成功。
[![请输入图片描述](images/lanproxy(5).png)](https://www.moerats.com/usr/picture/lanproxy(5).png)
如果显示不在线检查下防火墙端口和配置是否正确什么的。