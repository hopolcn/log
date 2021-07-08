# [翻译]Macvlan 网络驱动入门

**简介：** Macvlan 网络驱动入门 Macvlan网络驱动是为了在Docker的用户的使用场景中提供一个稳定的，生产可用，高性能的网络驱动。本文翻译自Docker官方的macvlan文档， 原文链接：https://docs.

# Macvlan 网络驱动入门

Macvlan网络驱动是为了在Docker的用户的使用场景中提供一个稳定的，生产就绪的网络驱动。目前Libnetwork 允许用户控制IPv4和IPv6地址管理。对于需要将容器网络和底层网络集成的用户来说，VLAN的驱动也允许他们完全控制二层VLAN taggine。而对于使用不依赖于物理网络约束的overlay网络方式部署网络结构的用户，可以参考[multi-host overlay ](http://139.224.240.244/engine/userguide/networking/get-started-overlay/)的驱动部署容器的网络。

Macvlan是一种新的网络虚拟化技术。Linux使用非常轻量的方式实现Macvlan，因为Mavlan不使用传统的Linux bridge做隔离和区分，而是直接与Linux的以太网接口或者子接口关联，以实现在物理网络中网络和连接的隔离。

Macvlan提供了很多独特的功能以及为后面更多模式加入的接口。Macvlan中的那些模式的优势在于比较高的网络性能，以及不依赖于Linux Bridge技术，简化虚拟网络的结构。移除了传统的Docker宿主机的网络设备和容器中虚拟网络设备中间的Linux bridge，而使用容器的虚拟网络接口直接挂载到宿主机的网络接口删个。这样在Docker上就可以更加方便的暴漏服务出去，因为这种方式不需要端口映射就可以让外部访问到容器中的服务。

## 实现准备

- 下面这个示例中的宿主机是使用Docker 1.12.0以上的版本的单机上测试的。
- 所有的示例都可以在运行了Docker的宿主机上测试，示例中使用了sub-interface比如`eth0.10`可以替换成`eth0`或者别的可用的宿主机上的parent-interface。子接口名字中有`.`字符的可以即使的自动的被创建。如果不指定`-o parent`的参数值，就会自动创建一个`dummy`的接口保证本地宿主机容器的连通性。
- 内核要求：
- 使用`uname -r`命令输出和检查kernel版本
- 支持Macvlan的kernel版本：v3.9–3.19 和 4.0+

## Macvlan Bridge模式使用示例

Macvlan Bridge模式每个容器有一个独立的MAC地址，用于记录宿主机上的MAC地址的端口的映射。

- Macvlan驱动的网络会被挂载到一个宿主机上的网络接口，例如宿主机的物理网络接口`eth0`，用于802.1q VLAN tagging的子接口，例如`eth0.10`(`.10`表示VLAN `10`)，或者绑定宿主机的适配器，将两个以太网接口捆绑成一个逻辑接口。
- 指定的网关是宿主机的外部的基础网络设施提供的网关
- 每个Macvlan Bridge模式的Docker网络都是相互隔离的，并且一个网络接口不允许多个网络同时挂载。并且理论上一个宿主机的网络适配器最多只能挂载4,094个子接口。
- 在`macvlan bridge`中，同一个子网内的任何一个容器都可以在不需要网关的情况下与别的容器互相通信。
- `docker network`命令对于vlan的驱动同样适用
- 在Macvlan模式下，在不同网络/子网中的容器不能在没有外部路由的情况下互相访问，或者同一个网络拥有多个子网，子网间也是不能在没有外部路由的情况下互相访问。

在下面的示例中，docker宿主机的`eth0`接口有一个在`172.16.86.0/24`网络中的IP和默认的网关地址`172.16.86.1`。这个网关是一个地址是`172.16.86.1`的外部路由器。对于`bridge`模式中，宿主机的`eth0`是不需要配置IP地址的，它仅仅是为了转发到上层网络中的交换机和路由器。

![Simple Macvlan Bridge Mode Example](http://139.224.240.244/engine/userguide/networking/images/macvlan_bridge_simple.png)

注意 在Macvlan bridge模式中子网的配置需要跟宿主机的接口的配置一致。例如，使用和宿主机以太网接口一直的子网和网关配置，这个网络接口可以通过`-o parent=`选项指定。

- 这个例子中我们使用的父接口是`eth0`并且它在子网`172.16.86.0/24`的网段上。在`docker network`中看到的容器需要和`-o parent=`使用同样的子网配置。网关是一个网络中的外部的路由器，到这个路有器之间没有ip地址伪装和别的代理。
- 使用网络驱动需要在创建网络时指明`-d 驱动名`选项，在这个驱动既`-d macvlan`选项
- 父接口配置通过`-o parent=eth0`，如下：

```
ip addr show eth0
3: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    inet 172.16.86.250/24 brd 172.16.86.255 scope global eth0
```

创建macvlan网络，并启动两个容器挂载到这个网络中：

```
# Macvlan  (-o macvlan_mode= Defaults to Bridge mode if not specified)
docker network create -d macvlan \
    --subnet=172.16.86.0/24 \
    --gateway=172.16.86.1  \
    -o parent=eth0 pub_net

# Run a container on the new network specifying the --ip address.
docker  run --net=pub_net --ip=172.16.86.10 -itd alpine /bin/sh

# Start a second container and ping the first
docker  run --net=pub_net -it --rm alpine /bin/sh
ping -c 4 172.16.86.10
```

看一下这俩容器的IP和路由表：

```
ip a show eth0
    eth0@if3: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UNKNOWN
    link/ether 46:b2:6b:26:2f:69 brd ff:ff:ff:ff:ff:ff
    inet 172.16.86.2/24 scope global eth0

ip route
    default via 172.16.86.1 dev eth0
    172.16.86.0/24 dev eth0  src 172.16.86.2

# NOTE: the containers can NOT ping the underlying host interfaces as
# they are intentionally filtered by Linux for additional isolation.
# In this case the containers cannot ping the -o parent=172.16.86.250
```

你可以通过`-o mavlan_mode=bridge`参数指定`bridge`模式，如果不指定 You can explicitly specify the `bridge` mode option `-o macvlan_mode=bridge`. 而且在不指定是默认也是`bridge`模式.

虽然在Macvlan模式中父接口`eth0`并不需要IP地址，但一般接口不会没有IP地址吧，在创建网络的时候如果使用默认的IPAM的话，可以使用`--aux-address=x.x.x.x`的表示，这样在分配给容器IP地址时就会排除掉指定的IP地址，下面这个例子就是排除了`eth0`的网络地址的命令：

```
docker network create -d macvlan \
    --subnet=172.16.86.0/24 \
    --gateway=172.16.86.1  \
    --aux-address="exclude_host=172.16.86.250" \
    -o parent=eth0 pub_net
```

另外一个默认的Docker IPAM驱动提供的IP地址选取的参数是`ip-range=`。这个表示了驱动分配IP地址的范围是`ip-range`的地址段，而不是使用`--subnet=`的参数的段。例如下面这个例子，IP地址的分配将从`192.168.32.128`开始分配。

```
docker network create -d macvlan  \
    --subnet=192.168.32.0/24  \
    --ip-range=192.168.32.128/25 \
    --gateway=192.168.32.254  \
    -o parent=eth0 macnet32

# Start a container and verify the address is 192.168.32.128
docker run --net=macnet32 -it --rm alpine /bin/sh
```

网络可以通过如下方式删除:

```
docker network rm <network_name or id>
```

- 注意 在Macvlan你不能ping或者联通宿主机的namespace中的IP地址，比如你创建了容器，并试图ping Docker宿主机上的`eth0`的地址，这个是不通的。这个通信由于安全性和隔离性被kernel的内核模块拦截了。

Docker的网络命令可以参考 [使用Docker network命令](http://139.224.240.244/engine/userguide/networking/work-with-networks/)

## Macvlan 802.1q Trunk Bridge 模式使用示例

VLAN技术很长时间都被用作数据中心虚拟化网络的解决方案，并且在今天还有很多的使用。VLAN通过一个12位的标签，区分不同的域的方式实现对单个或者多个子网的逻辑的分组。对于网络工程师来说通过VLAN划分网络和配置例如`web`，`db`或者其他需要隔离的服务的安全是在正常不过的事了。

目前一个宿主机上需要连接多个虚拟网络是很正常的，Linux网络很早就支持VLAN的标签，它的标准是802.1q，用于管理网络中的数据隔离。可以通过配置连接到Docker宿主机的以太网连接支持802.1q的VLAN ID，以便于创建Linux的子接口，而每个子接口可以用唯一的VLAN ID。

![Multi Tenant 802.1q Vlans](http://139.224.240.244/engine/userguide/networking/images/multi_tenant_8021q_vlans.png)

众所周知，给Linux宿主机配置802.1q的操作时很复杂和痛苦的，它需要配置一系列的文件，以便于在系统重启后依然能保证配置。如果再引入了网桥的话，物理网卡的接口需要挂载到这个网桥上，然后每个网桥获得IP地址。因为还需要再不同网桥上隔离访问，这样就导致了非常复杂的网络配置。

和其他Docker网络驱动类似，驱动的首要目标即缓解管理网络资源时操作的痛苦。为了那个目的，当一个网络在创建网络的时候收到一个并不存在的子接口作为网络的父接口，那么驱动就会在创建网络的时候创建一个VLAN标记的接口。

当宿主机重启时，这个驱动会在Docker daemon重启的时候重新创建网络的网络接口，而不是修改那些复杂的网络配置文件。如果VLAN的子接口是Docker daemon在`docker network create`的时候创建的，那么在Docker重启后只会重新创建这个子接口，并在在`docker network rm`的时候删除这个子接口。

如果用户不希望Docker修改`-o parent`参数中的子接口，那么用户只简单的需要在这个参数中传入一个已经存在的接口作为网络的父接口。像`eth0`这样的父接口是不会被删除的，只有那些不是master的接口才能被删除。

这个驱动添加和删除的vlan子接口的格式是`interface_name.vlan_tag`。

例如：`eth0.50`意思是父接口是`eth0`，挂载在它上面的一个子接口是`eth0.50`，并且vlan id是`50`。这个配置等价到`ip link`的配置命令是： `ip link add link eth0 name eth0.50 type vlan id 50`。

Vlan ID 50

在Docker的宿主机上创建第一个VLAN标记和隔离的网络，通过在创建网络的时候指定了`-o parent=eth0.50`指定父接口为vlan id为`50`的接口`eth0.50`。这个父接口也可以手动指定别的名字，但如果那样就必须手动通过`ip link`或者配置文件创建和删除父接口。如果通过`-o parent`指定的接口存在的话，只要这个接口兼容与Linux 网络接口的标准就可以被使用。

```
# 现在创建网络挂载到VLAN标记的接口上
docker network  create  -d macvlan \
    --subnet=192.168.50.0/24 \
    --gateway=192.168.50.1 \
    -o parent=eth0.50 macvlan50

# 在两个不同的终端上，启动两个Docker容器，然后两个容器就分别可以ping通另外一个。
docker run --net=macvlan50 -it --name macvlan_test5 --rm alpine /bin/sh
docker run --net=macvlan50 -it --name macvlan_test6 --rm alpine /bin/sh
```

Vlan ID 60

创建另外一个隔离的，VLAN标记的隔离的网络，通过在创建网络的时候指定了`-o parent=eth0.60`指定父接口为vlan id为`60`的接口`eth0.60`，而`macvlan_mode=`参数的默认值为`macvlan_mode=bridge`，可以看到在下面这个示例中有相同的结果。

```
# 现在创建网络挂载到VLAN标记的接口上
docker network  create  -d macvlan \
    --subnet=192.168.60.0/24 \
    --gateway=192.168.60.1 \
    -o parent=eth0.60 -o \
    -o macvlan_mode=bridge macvlan60

# 在两个不同的终端上，启动两个Docker容器，然后两个容器就分别可以ping通另外一个。
docker run --net=macvlan60 -it --name macvlan_test7 --rm alpine /bin/sh
docker run --net=macvlan60 -it --name macvlan_test8 --rm alpine /bin/sh
```

例子: 多个子网的 Macvlan 802.1q Trunking

下面的例子除了添加了更多的用户选择的容器子网的绑定外和前面的例子是一样的。在MacVlan/Bridge模式，容器只能ping通同一个子网/广播域，除非有外部的路由去路由不同的子网的网络流量。

```
### 创建多个L2网络
docker network create -d ipvlan \
    --subnet=192.168.210.0/24 \
    --subnet=192.168.212.0/24 \
    --gateway=192.168.210.254  \
    --gateway=192.168.212.254  \
     -o ipvlan_mode=l2 ipvlan210

# 测试 192.168.210.0/24 容器间连接性
docker run --net=ipvlan210 --ip=192.168.210.10 -itd alpine /bin/sh
docker run --net=ipvlan210 --ip=192.168.210.9 -it --rm alpine ping -c 2 192.168.210.10

# 测试 192.168.212.0/24 容器间连接性
docker run --net=ipvlan210 --ip=192.168.212.10 -itd alpine /bin/sh
docker run --net=ipvlan210 --ip=192.168.212.9 -it --rm alpine ping -c 2 192.168.212.10
```

## 多层 IPv4 IPv6 Macvlan Bridge 模式

示例: Macvlan Bridge模式, 802.1q trunk, VLAN ID: 218, 多个子网 构成的多层

```
# 创建多个子网网段的macvlan网络
docker network  create  -d macvlan \
    --subnet=192.168.216.0/24 --subnet=192.168.218.0/24 \
    --gateway=192.168.216.1  --gateway=192.168.218.1 \
    --subnet=2001:db8:abc8::/64 --gateway=2001:db8:abc8::10 \
     -o parent=eth0.218 \
     -o macvlan_mode=bridge macvlan216

# 在第一个192.168.216.0/24网段创建一个容器
docker run --net=macvlan216 --name=macnet216_test --ip=192.168.216.10 -itd alpine /bin/sh

# 在第二个192.168.218.0/24网段创建容器
docker run --net=macvlan216 --name=macnet216_test --ip=192.168.218.10 -itd alpine /bin/sh

# 通过192.168.216.0/24的网段的容器Ping 在192.168.216.0/24网段中的第一个容器
docker run --net=macvlan216 --ip=192.168.216.11 -it --rm alpine /bin/sh
ping 192.168.216.10

# 通过192.168.218.0/24的网段的容器Ping 在192.168.218.0/24网段中的第一个容器
docker run --net=macvlan216 --ip=192.168.218.11 -it --rm alpine /bin/sh
ping 192.168.218.10
```

查看其中一个容器的详情:

```
docker run --net=macvlan216 --ip=192.168.216.11 -it --rm alpine /bin/sh

root@526f3060d759:/# ip a show eth0
    eth0@if92: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
    link/ether 8e:9a:99:25:b6:16 brd ff:ff:ff:ff:ff:ff
    inet 192.168.216.11/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 2001:db8:abc4::8c9a:99ff:fe25:b616/64 scope link tentative
       valid_lft forever preferred_lft forever
    inet6 2001:db8:abc8::2/64 scope link nodad
       valid_lft forever preferred_lft forever

# 指定的IP V4的网关地址是192.168.216.1
root@526f3060d759:/# ip route
  default via 192.168.216.1 dev eth0
  192.168.216.0/24 dev eth0  proto kernel  scope link  src 192.168.216.11

# 指定的IP V6的网关地址是 2001:db8:abc8::10
root@526f3060d759:/# ip -6 route
  2001:db8:abc4::/64 dev eth0  proto kernel  metric 256
  2001:db8:abc8::/64 dev eth0  proto kernel  metric 256
  default via 2001:db8:abc8::10 dev eth0  metric 1024原文链接：https://docs.docker.com/engine/userguide/networking/get-started-macvlan/#dual-stack-ipv4-ipv6-macvlan-bridge-mode
```