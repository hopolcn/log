# 使用 Tinc 组建大内网

# 介绍

Tinc VPN 是一个轻量型的 GNU 协议下的开源软件，通过隧道以及加密技术在互联网点与点之间创立隧道。VPN 是 IP 层面上的，所以可以像普通的网络设备那样，不需要去适配其他已经存在的软件。所以他就可以很安全的在点与点之间传输数据，并不需要担心泄露。他还有其他几大的特点：

- 加密 / 认证 / 压缩
- 自动全网状路由
- 易于扩展网络节点
- 能够进行网络的桥接
- 跨平台支持
- IPv6 支持

(上面的内容基本就是官网首页的一个简单的翻译，官方网站：https://www.tinc-vpn.org/)

# 源码

https://tinc-vpn.org/git/tinc

# 安装

Debian/Ubuntu

```
apt-get install tinc
```

CentOS

```
yum install tinc
```

ArchLinux

```
pacman -S tinc
```

Android

[Tinc App](https://tincapp.pacien.org/)

IOS(需越狱)

[Cydia packages](https://www.tinc-vpn.org/packages/cydia/)

Windows

[官方下载地址](https://www.tinc-vpn.org/packages/windows/tinc-1.0.35-install.exe)

MacOS

```
brew install tinc --devel
brew cask install tuntap
```

# 配置

## 目录结构

```
/etc/tinc
└── dock
    ├── hosts
    │   ├── Server
    │   └── Client
    ├── rsa_key.priv
    ├── tinc.conf
    ├── tinc-down
    └── tinc-up
```

- `/etc/tinc/dock` 目录下的文件都属于`dock`这个网络，可以根据自己的情况修改名称，相应下面的命令也需要修改
- `/etc/tinc/dock/hosts` 目录是存放其他用户或者说是其他网络的`public key`以及他们的 ip 地址
- `rsa_key.priv` 本网络的私钥
- `tinc.conf` 本网络的配置文件
- `tinc-down` 本网络关闭时执行的脚本
- `tinc-up` 本网络启动时执行的脚本

客户端和服务端都需要手动新建相同的目录结构，`Windows`端无需新建`tinc-up`和`tinc-down`两个文件

## 服务端配置

首先开启 Linux 转发，在`/etc/sysctl.conf`设置`net.ipv4.ip_forward = 1`，并通过`sysctl -p`来应用配置。

修改`tinc.conf`配置文件

```
Name = Server
Interface = tinc
Mode = switch
Compression=9
Cipher  = aes-256-cbc
Digest = sha256
PrivateKeyFile=/etc/tinc/dock/rsa_key.priv
```

- `Name` 主机名称
- `Interface` 隧道所使用的网卡名称
- `Mode` 有三种模式，分别是 `router` / `switch` / `hub` ，相对应我们平时使用到的路由、交换机、集线器 (默认模式 `router`)
- `Compression` UDP 数据包压缩级别。可选有 0 (关闭)，1 (fast zlib) 至 9 (best zlib)，10 (fast lzo) 和 11 (best lzo)
- `Cipher` 加密类型。可选 `aes-128-cbc` `aes-256-cbc` 等
- `Digest` rsa 加密协议强度。可选 `sha128` `sha1` 等
- `PrivateKeyFile` 服务器私钥的位置

```
Linux`和`MacOS`修改`tinc-up`和`tinc-down
tinc-up
#!/bin/sh
ifconfig $INTERFACE <内网ip> netmask 255.255.255.0
tinc-down
#!/bin/sh
ifconfig $INTERFACE down
```

运行命令添加执行权限

```
chmod +x tinc-*
```

在`hosts`文件夹内新建在`tinc.conf`设置的主机名称的节点配置文件

```
Subnet=10.1.3.1/32
Address=149.129.88.238
Port=57734
```

- `Subnet` 宣告的路由地址
- `Address` 服务器的外网 IP
- `Port` 指定 tinc 连接端口(默认端口`655`)

使用命令生成私钥和公钥

```
tincd -n dock -K4096
```

私钥生成在与`tinc.conf`配置文件相同的文件夹，生成的公钥会自动添加到`hosts`文件夹内的节点配置文件里

## 客户端配置

客户端的`tinc.conf`与服务器的参数基本上相同，只需要修改`Name`

在`hosts`文件夹内新建在`tinc.conf`设置的主机名称的节点配置文件

```
Subnet=10.1.3.2/32
```

`tinc-up`和`tinc-down`跟服务器配置基本一样，只需要修改`tinc-up`的内网ip，`Windows`客户端无需这两个文件

使用命令生成私钥和公钥

```
tincd -n dock -K4096
```

将服务端生成完公钥的节点配置文件放到客户端的`hosts`文件夹内，并将客户端生成完公钥的节点配置文件放到服务端的`hosts`文件夹内

# 运行

后台启动

Windows 端需要先安装虚拟网卡，在 tinc 的安装目录下有虚拟网卡的驱动安装包，安装完成后需要将虚拟网卡名称改为与`tinc.conf`文件中的`Interface`名称相同，并且手动设置虚拟网卡的 IP 地址和子网掩码，然后进入到`tinc`的安装目录下再以管理员的身份运行，运行后会自动创建系统服务，需要停止的时候在 Windows 系统服务管理中停止服务

```
# Linux/MacOS
tincd -n dock
# Windows(需要管理员权限)
tincd.exe -n dock
```

运行完后，使用`ifconfig`查看是否有`tinc.conf`设置的网卡名称，如果没有请检查`tinc-up`和`tinc-down`两个文件是否授予执行的权限；如果已经文件已经赋予权限还是没显示网卡，请使用调试模式运行查看问题

停止运行，该命令在`Windows`端会停止运行并删除系统服务

```
# Linux/MacOS
tincd -n dock -k
# Windows(需要管理员权限)
tincd.exe -n dock -k
```

调试模式

```
# Linux/MacOS
tincd -n dock -D -d 3
# Windows
tincd.exe -n dock -D -d 3
```

注意：调试模式时无法使用`Ctrl+C`停止运行，需要输入命令`tinc -n dock -k`才能停止运行

# 参考来源

- https://imlonghao.com/46.html
- https://blog.zjustin.me/post/tinc-memo/