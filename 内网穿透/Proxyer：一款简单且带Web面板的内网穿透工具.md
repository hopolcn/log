## Proxyer：一款简单且带Web面板的内网穿透工具

https://www.moerats.com/archives/1026/



**说明：**关于内网穿透的工具，博主已经介绍的非常多了，比如[frp](https://www.moerats.com/archives/958/)、[lanproxy](https://www.moerats.com/archives/727/)、[nps](https://www.moerats.com/archives/891/)、[holer](https://www.moerats.com/archives/991/)、[sish](https://www.moerats.com/archives/1002/)和[serveo](https://www.moerats.com/archives/990/)等，用起来都还行，不过有些在安装和使用上对于一些新手来说，还是比较复杂的，最近博主发现了个新的内网穿透项目`Proxyer`，目前仅支持`TCP`协议、虽然看起来功能比较简单，但基本可以满足日常使用了，特别是在安装和使用方面，对于新手是比较友好的，这里就分享下。

## 截图

[![请输入图片描述](images/proxyer(1).png)](https://www.moerats.com/usr/picture/proxyer(1).png)
[![请输入图片描述](images/proxyer(2).png)](https://www.moerats.com/usr/picture/proxyer(2).png)

## 服务端

**Github地址：**https://github.com/khvysofq/proxyer

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
curl -L "https://get.daocloud.io/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

**3、安装Proxyer**

```
wget https://raw.githubusercontent.com/khvysofq/proxyer/master/docker-compose.yml
#请将后面1.1.1.1改成你的服务器ip地址后再运行
export PROXYER_PUBLIC_HOST=1.1.1.1
docker-compose up -d
```

安装完成后，就可以通过`ip:6789`访问服务端`WEB`管理面板了，进去后需要设置一个客户端认证密码。

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

## 客户端

进入服务端面板后，界面会提供`Linux`、`Windows`、`macOS`客户端版本，然后自行根据自身系统下载指定版本的压缩包即可。

`Windows`可以直接下载界面版本，然后双击可执行文件，会弹出一个网页界面，输入上面的认证密码，即可开始配置穿透。

`Linux`下载压缩包后，解压出二进制文件，直接在当前目录使用`./proxyer`命令运行即可。

最后使用起来还是很简单的，由于是新项目，功能可能不是很丰富，看作者后期会不会慢慢完善了。