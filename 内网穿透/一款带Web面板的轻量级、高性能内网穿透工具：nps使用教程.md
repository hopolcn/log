## 一款带Web面板的轻量级、高性能内网穿透工具：nps使用教程

https://www.moerats.com/archives/891/



**说明：**内网穿透工具之前已经介绍了不少了，比如`Frp`、`lanproxy`、`Holer`等，现在再介绍个带`Web`面板的穿透工具`nps`，之前叫`easyProxy`，只是改名了而已，该工具是一款使用`go`语言编写的轻量级、功能强大的内网穿透服务器。支持`tcp`、`udp`流量转发，支持内网`http`、`socks5`代理，同时支持`snappy`压缩(节省带宽和流量)、站点保护、加密传输、多路复用、`header`修改等。同时还支持`web`图形化管理。

## 截图

[![请输入图片描述](images/nps_ct(1).png)](https://www.moerats.com/usr/picture/nps_ct(1).png)
[![请输入图片描述](images/nps_ct(2).png)](https://www.moerats.com/usr/picture/nps_ct(2).png)

## 安装

**Github地址：**https://github.com/cnlh/nps

通常内网穿透工具都有服务端和客户端，安装要求如下：

```
服务端：需要安装在一个有公网IP的服务器上，系统为Linux/Windows/Mac均可。
客户端：一般安装在一个内网的VPS服务器或Windows/Mac电脑上使用。
```

**1、编译安装**

```
提示：编译安装主要讲的Linux系统，其它系统(Win/Mac，也包括Linux)建议直接使用作者编译好的文件即可。
```

安装`Go`语言：

```
#Debian/Ubuntu系统
apt-get -y install golang
#创建目录并定义GOPATH环境变量指向该目录
mkdir ~/workspace
echo 'export GOPATH="$HOME/workspace"' >> ~/.bashrc
source ~/.bashrc

#CentOS/RHEL系统
yum -y install golang
#创建目录并定义GOPATH环境变量指向该目录。
mkdir ~/workspace
echo 'export GOPATH="$HOME/workspace"' >> ~/.bashrc
source ~/.bashrc
```

安装`git`：

```
#Debian/Ubuntu系统
apt-get -y install git

#CentOS/RHEL系统
yum -y install git
```

安装源码：

```
go get github.com/cnlh/nps
```

编译服务端和客户端：

```
#进入指定目录
cd ~/workspace/src/github.com/cnlh/nps
#编译服务端
go build cmd/nps/nps.go
#编译客户端
go build cmd/npc/npc.go
```

编译好了后，就会在当前目录生成`npc`或`nps`二进制文件了，就可以直接拿来用了。

编译的时候可能出现的问题解决方法：

```
#只拿一种常见的错误做例子，有时候可能会出现很多种这样的提示
lib/kcp/crypt.go:14:2: cannot find package "golang.org/x/crypto/pbkdf2" in any of:
    /usr/lib/go-1.7/src/golang.org/x/crypto/pbkdf2 (from $GOROOT)
    /root/workspace/src/golang.org/x/crypto/pbkdf2 (from $GOPATH)

#意思是缺少这种包，然后记住提示的地址，比如上面的golang.org/x/crypto/pbkdf2，有时候也会提示的github地址。

然后再使用命令go get golang.org/x/crypto/pbkdf2命令安装一下就行了。
```

**2、直接安装**
除了自己编译外，作者也直接提供了编译好的文件给你使用，文件下载地址：[点击进去](https://github.com/cnlh/nps/releases)，然后再根据自己的系统架构下载对应的最新版服务端和客户端。

如果对于`Linux`服务器还是不知道怎么选择的，这里拿`Vultr`、搬瓦工大多数`VPS`为例。先使用命令`getconf LONG_BIT`获取系统版本，`32`位就选`386`，`64`就选`amd64`，具体还是以实际情况为准。

## 服务端使用

这里博主使用的是`Vultr Linux x64`服务器，直接使用命令：

```
#记得复制前先将下面链接替换成当前最新版地址
cd ~
#下载并解压服务端
wget https://github.com/cnlh/nps/releases/download/v0.0.14/linux_amd64_server.tar.gz && tar zxvf linux_amd64_server.tar.gz
#编辑配置文件
cd nps
nano conf/nps.conf
```

配置文件参数如下：

```
#web管理端口
httpport
#web界面管理密码
password
#服务端客户端通信端口
bridePort
#ssl certFile绝对路径
pemPath
#ssl keyFile绝对路径
keyPath
#域名代理https代理监听端口
httpsProxyPort
#域名代理http代理监听端口
httpProxyPort
#web api免验证IP地址
authip
#客户端与服务端连接方式kcp或tcp
bridgeType
```

然后启动服务端：

```
./nps install
./nps start

#重启/停止服务端
./nps stop|restart
```

然后打开地址`http://ip:8080`访问管理界面，具体端口以自己修改的为准，再使用密码登录进去，默认为`123`。

```
#如果打不开Web界面，就需要开启防火墙，一般CentOS系统出现情况最多
#Centos 6系统
iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
service iptables save
service iptables restart

#CentOS 7系统
firewall-cmd --zone=public --add-port=8080/tcp --permanent 
firewall-cmd --reload
```

对于有些服务器，比如阿里云，谷歌云等，还需要在服务器管理面板上开放`Web`端口才行。

## 客户端使用

**1、Linux系统**

```
#比如下载的客户端文件在根目录，先进入根目录
cd ~
#启动客户端，比如服务端公网IP为1.1.1.1，服务端配置文件中tcpport为8284
./npc -server=1.1.1.1:8284 -vkey=客户端的密钥
```

**2、Windows系统**
首先按住`Win+R`，输入`cmd`进入命令窗口，然后使用命令：

```
#比如下载的客户端文件在D盘，先进入到D盘
cd /d d:
#启动客户端，比如服务端公网IP为1.1.1.1，服务端配置文件中tcpport为8284
npc.exe -server=1.1.1.1:8284 -vkey=客户端的密钥
```

至于`Mac`系统启动参考上面就行。

## 使用场景

关于使用场景，`Github`文档写的很清楚了，这里大概的说下。

**1、tcp隧道模式**

```
适用：想在外网通过ssh连接内网的机器，做云服务器到内网服务器端口的映射，或者做微信公众号开发、小程序开发等。
```

详细教程→[点击查看](https://github.com/cnlh/nps#tcp隧道)。

**2、udp隧道模式**

```
适用：在非内网环境下使用内网dns，或者需要通过udp访问内网机器等。
```

详细教程→[点击查看](https://github.com/cnlh/nps#udp隧道)。

**3、http代理模式**

```
适用：在外网使用HTTP代理访问内网站点。
```

详细教程→[点击查看](https://github.com/cnlh/nps#http正向代理)。

**4、socks5代理模式**

```
适用：搭建一个内网穿透55，在外网如同使用内网v皮n一样访问内网资源或者设备。
```

详细教程→[点击查看](https://github.com/cnlh/nps#socks5代理)。

## 相关功能

**1、数据压缩支持**
由于是内网穿透，内网客户端与服务端之间的隧道存在大量的数据交换，为节省流量，加快传输速度，由此本程序支持`SNNAPY`形式的压缩。

- 所有模式均支持数据压缩，可以与加密同时使用
- 开启此功能会增加`cpu`和内存消耗
- 在`server`端加上参数`-compress=snappy`(或在`web`管理中设置)

**2、加密传输**
如果公司内网防火墙对外网访问进行了流量识别与屏蔽，例如禁止了`ssh`协议等，通过设置配置文件，将服务端与客户端之间的通信内容加密传输，将会有效防止流量被拦截。

- 开启此功能会增加`cpu`和内存消耗
- 在`server`端加上参数`-crypt=true`(或在web管理中设置)

**3、站点保护**
域名代理模式所有客户端共用一个`http`服务端口，在知道域名后任何人都可访问，一些开发或者测试环境需要保密，所以可以设置用户名和密码，`nps`将通过`Http Basic Auth`来保护，访问时需要输入正确的用户名和密码。

- `web`管理中可配置

**4、host修改**
由于内网站点需要的`host`可能与公网域名不一致，域名代理支持`host`修改功能，即修改`request`的`header`中的`host`字段。

- 在`web`管理中设置

**5、自定义header**
支持对`header`进行新增或者修改，以配合服务的需要。

**6、404页面配置**
支持域名解析模式的自定义`404`页面，修改`/web/static/page/error.html`中内容即可，暂不支持静态文件等内容。

**7、流量限制**
支持客户端级流量限制，当该客户端入口流量与出口流量达到设定的总量后会拒绝服务，域名代理会返回`404`页面，其他会拒绝连接。

**8、带宽限制**
支持客户端级带宽限制，带宽计算方式为入口和出口总和，权重均衡。

**9、负载均衡**
本代理支持域名解析模式的负载均衡，在`web`域名添加或者编辑中内网目标分行填写多个目标即可实现轮训级别的负载均衡。

**10、守护进程**
本代理支持守护进程，使用示例如下，服务端客户端所有模式通用，支持`linux`、`darwin`、`windows`。

```
./(nps|npc) start|stop|restart|status 若有其他参数可加其他参数
(nps|npc).exe start|stop|restart|status 若有其他参数可加其他参数
```

**11、KCP协议支持**
`KCP`是一个快速可靠协议，能以比`TCP`浪费`10%-20%`的带宽的代价，换取平均延迟降低`30%-40%`，在弱网环境下对性能能有一定的提升。可在`app.conf`中修改`bridgeType`为`kcp`。

- 当服务端为`kcp`时，客户端连接时也需要加上参数`-type=kcp`。

该工具很强大，更多的使用可以自行研究，如果有人知道`Frp`管理面板的话，可以给博主提供下。

## 相关教程

- [Frps一键安装脚本，带Frpc Windows便捷启动脚本](https://www.moerats.com/archives/797/)
- [一款带Web管理面板的内网穿透工具：lanproxy使用教程](https://www.moerats.com/archives/727/)
- [使用Holer远程登录家里或公司内网的电脑](https://www.moerats.com/archives/716/)