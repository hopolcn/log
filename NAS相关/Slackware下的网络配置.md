# Slackware下的网络配置

https://blog.csdn.net/y4y5hejin/article/details/4505321



Slackware 有个很有用的网络配置工具--netconfig,对于新手来说很容易上手的。除此之外我们可以手动修改相关的配置文件或运行ifconfig进行修改。 不管是运行netconfig还是ifconfig,都是间接地修改配置文件。下面我们来逐个介绍各个相关的配置文件。

网络配置相关的文件：/etc/HOSTNAME,/etc/resolv.conf,/etc/inetd,/etc/networks,/etc/hosts,/etc/rc.d/rc.inet1.conf,/etc/rc.d/rc.inetd,

\----------------------------------------
/etc/HOSTNAME,设置主机名和域名，可运行如下命令进行查看：

bash-3.1$ cat /etc/HOSTNAME
pirate.bigbigegg.cn

其中pirate为主机名，bigbigegg.cn为域名。
主机名是必须的，网络上的主机都必须又自己的主机名和域名（当然在本地你可以随便弄一个就行）。

\-----------------------------------------

/etc/resolv.conf,DNS服务器信息，不管你是静态IP还是拨号，这个都是必须的。可运行如下命令查看相关信息：

bash-3.1$ cat /etc/resolv.conf
search bigbigegg.cn
nameserver 202.117.112.3

其中bigbigegg.cn是域名，202.117.112.3为域名解析服务器地址。

\------------------------------------------

/etc/hosts,将主机和IP绑定；在这文件里你会发现如下的信息：

\# For loopbacking.
127.0.0.1        localhost
10.10.12.80       bigbigegg.cn pirate

其中127.0.0.1是本地保留的回环地址，与localhost绑定；10.10.12.80与域名为bigbigegg.cn中的主机名为pirate的主机绑定。

\------------------------------------------

/etc/networks,设置网络号相关的信息；

bash-3.1$ cat /etc/networks
\#
\# networks   This file describes a number of netname-to-address
\#        mappings for the TCP/IP subsystem. It is mostly
\#        used at boot time, when no name servers are running.
\#

loopback    127.0.0.0
localnet    10.10.12.0

\# End of networks.

10.10.12.80所在网络号为10.10.12.0,因为子网掩码为255.255.255.0

\---------------------------------------------

/etc/rc.d/rc.inet1.conf 里有更详细的配置，

\#这是对第一块网卡的配置信息
\# Config information for eth0:
IPADDR[0]="10.10.12.80"    /* IP地址 */
NETMASK[0]="255.255.255.0"   /* 子网掩码 */
USE_DHCP[0]=""     /* "yes"|"no" */
DHCP_HOSTNAME[0]=""     /* DNS服务器地址 */

若要使用静态IP则需配置好IPADDR,NETMASK项；若使用自动获取IP，则USE_DHCP="yes",DHCP服务器地址可选。下边为运行netconfig且选DHCP后的配置信息：

\# Config information for eth0:
IPADDR[0]=""         /* 自动获取 */
NETMASK[0]=""
USE_DHCP[0]="yes"       /* 使用DHCP */
DHCP_HOSTNAME[0]="111.222.333.444"   /* DHCP服务器地址（当然这个地址是不存在的，仅供演示参考） */

默认情况下一个网卡可配置4个IP地址，配置过程相似。

设置网关的相关信息也在这个文件里，如下是运行netconfig使用静态IP时使用的网关：

\# Default gateway IP address:
GATEWAY="10.10.12.254"    /* 没有可空 */

更多配置信息请参阅/etc/rc.d/rc.inet1.conf .

\-------------------------------------------------

/etc/rc.d/rc.inetd 为网络驻守进程，可启动/停止/重启网卡进程。

\-------------------------------------------------

（以下于2008-04-28添加）
网卡相关设置：

激活/关闭网卡：ifconfig eth0 up/down

获取动态ip:ifconfig eth0 --dynamic

注:ifconfig命令可使当前配置生效，但重启后将失效；

路由器相关操作：

开启路由功能：echo 1 >/proc/sys/net/ipv4/ip_forward
(1为打开，0为关闭)

添加/删除路由信息：route add/del -net 10.0.0.1 netmask 255.0.0.0 gw 192.168.0.1

添加／删除默认路由：route add/del -net 0.0.0.0 netmask 0.0.0.0 gw 192.168.0.1