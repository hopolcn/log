# N1小钢炮安装zerotier 进行内网穿透

https://post.smzdm.com/p/a5k6qlz3/

**创作立场声明：**原创整理

近期买了只N1，参考灯大的教程刷了个小钢炮系统，挂载10T[硬盘](https://www.smzdm.com/fenlei/yingpan/)装小姐姐~为了方便以及安全着想，想进行内网穿透，frp映射了部分端口，接下啦介绍zerotier安装，zerotier 的使用不在本次介绍。

[![N1小钢炮安装zerotier 进行内网穿透](images/5d1989ef9f6659861.png_e680.jpg)](https://post.smzdm.com/p/a5k6qlz3/pic_2/)

一、安装entware

rm -rf /opt
mkdir /opt
cd /opt
wget -O - http://bin.entware.net/aarch64-k3.10/installer/alternative.sh | sh

将自带opkg改名为opkg_bak暂时停用 灯大固件更新可以改回来免重装系统更新软件

mv /usr/bin/opkg /usr/bin/opkg_bak

二、配置entware环境变量

vim /etc/profile

直接在前面/usr/sbin:这行下直接添加下面两行并保存退出（ESC+:wq+Enter）

/opt/bin:/opt/sbin:

使配置生效

source /etc/profile

三、检查entware环境安装情况看是否报错

opkg update
opkg list

四、安装zerotier

opkg install zerotier

启动zerotier

zerotier-one -d

配置开启自启

vim /etc/init.d/S50zerotier.sh

写入以下内容保存：

\#!/bin/shzerotier-one -d

然后给予权限：

chmod 777 S50zerotier.sh



