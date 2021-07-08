## 一款基于Frp的Web管理面板：FrpMgr安装及使用

https://www.moerats.com/archives/958/



**说明：**`FrpMgr`是一个基于`Frp`的快速配置`Web`面板，可以一键配置生成客户端的`Frp`配置文件，远程安装`Frp`服务到任意一台服务器，让我们在使用配置`Frp`上方便很多。对于类似这种带`Web`面板的穿透工具，之前也发过不少，比如[nps](https://www.moerats.com/archives/891/)、[lanproxy](https://www.moerats.com/archives/727/)等，都挺不错的，有兴趣可以去了解下，这里就介绍下`FrpMgr`安装及使用。

## 截图

[![请输入图片描述](images/FrpMgr(1).png)](https://www.moerats.com/usr/picture/FrpMgr(1).png)
[![请输入图片描述](images/FrpMgr(2).png)](https://www.moerats.com/usr/picture/FrpMgr(2).png)

## 更新

```
【2019年11月21日】
新增远程桌面，ssh内网穿透，本地目录穿透。
【2019年6月27日】
新增状态查看功能，可查看流量、客户端连接数，连接状态等。
```

## 安装

**Github地址：**https://github.com/Zo3i/frpMgr

**说明：**由于该面板使用的`JAVA`、`Mysql 5.7`，所以`512M`的内存大部分是跑不起来的，如果内存太小，先加一点虚拟内存，可以使用`Swap`一键脚本→[传送门](https://www.moerats.com/archives/722/)。

**1、安装Docker**

```
#CentOS 6
rpm -iUvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
yum update -y
yum -y install docker-io
service docker start
chkconfig docker on

#CentOS 7、Debian、Ubuntu
curl -sSL https://get.docker.com/ | sh
systemctl start docker
systemctl enable docker
```

**2、安装Docker Compose**

```
curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

**3、安装git**

```
#Debian/Ubuntu系统
apt -y install git

#CentOS系统
yum -y install git
```

**4、安装FrpMgr**

```
#拉取源码
git clone https://github.com/Zo3i/frpMgr.git
#构建Mysql镜像
cd frpMgr/web/src/main/docker/final/mysql
docker build -t jo/mysql .
#构建frp并启动镜像
cd ..
chmod +x w.sh
docker-compose up -d
```

面板访问地址：`ip:8999/frp`，账号`admin`，密码`12345678`，登录成功后在面板修改密码即可。

## 使用

```
提示：这里安装面板的服务器是没有给你安装Frp的，你可以在下面服务器配置的时候，填上ip，就可以安装frp了。
```

1、首先去域名服务商解析一个泛域名(如`*.moerats.com`)到服务器`ip`。

2、点击左侧`FRP`服务器配置，域名只需要填主域名，这里默认的服务器端口为`22`。
[![请输入图片描述](images/FrpMgr(3).png)](https://www.moerats.com/usr/picture/FrpMgr(3).png)
填好后，点击远程安装，输入服务器密码即可，服务器端系统目前支持`CentOS 7`、`Debian 8+`、`Ubuntu 16+`，且注意防火墙需要打开`Web`端口。

3、点击左侧`FRP`客户端配置，填上二级域名(比如`rats`、后面就不要了)，本地端口就可以了。
[![请输入图片描述](images/FrpMgr(4).png)](https://www.moerats.com/usr/picture/FrpMgr(4).png)
最后点击右侧，下载`Win`或者`Mac`配置压缩包即可，`Win`的话解压出来打开`open.bat`即可，连接地址为`二级域名:Web端口`。

由于没有`Win`客户端开机自启，这里博主就额外说下`Windows`开机自启步骤。

```
1、新建一个vbs后缀的脚本，比如rats.vbs，脚本代码如下：
set ws=WScript.CreateObject("WScript.Shell")
ws.Run "C:\Users\Desktop\frp\frpc.exe -c C:\Users\Desktop\frp\frpc.ini",0
第二行为frp文件夹路径，不直接具体路径的，打开frp文件夹，左上角就是路径，复制即可

2、使用Win+R、输入shell:startup确认运行，将脚本放进弹出来的文件夹里面即可。
```

## 总结

该面板功能什么的目前还是挺简单的，不过对于要求不高的来说，基本可以满足了，如果你要求更高的话，可以试试文章开头介绍的`nps`、`lanproxy`等，最后作者表示会一直维护下去的，并逐渐增加功能，有想法的可以在下面评论，作者也会经常来查看的，然后有心的可以去`Github`给个`Star`鼓励下作者就可以了，毕竟`Frps`管理面板很少见。