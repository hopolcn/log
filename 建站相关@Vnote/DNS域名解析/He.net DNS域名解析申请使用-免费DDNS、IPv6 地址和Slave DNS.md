# He.net DNS域名解析申请使用-免费DDNS、IPv6 地址和Slave DNS
Hurricane Electric 是一家位于美国的全球互联网服务提供商，该公司成立于1994年，提供IPv4和IPv6接入以及位于美国圣荷西的数据中心服务，是世界上最大的交换中心参与者，其参与的交换中心包括了所有73个Euro-IX的交换中心。

He.net 同时也提供免费的DNS域名解析服务，凭借Hurricane Electric强大的服务器与技术实力，[He.net DNS](https://wzfou.com/tag/he-net-dns/)的域名解析服务非常地稳定，并且一直以来He.net DNS都是可以免费使用，并且DNS支持A、CNAME、CAA等各大解析记录，非常全面。

He.net DNS可以满足绝大多数的DNS解析服务需求，另外值得一提的就是He.net DNS提供了免费DDNS（动态域名解析）和IPv6 地址接入服务，特别适合自建DNS解析服务。He.net 支持Slave DNS，即备用DNS，让你的DNS解析“高枕无忧”。

[![权威He.net DNS域名解析申请使用-免费DDNS、IPv6 地址和Slave DNS](images/20200219115952114_3880.webp)](https://wzfou.com/wp-content/uploads/2018/07/he.net_00.gif)

本篇文章就来分享一下He.net DNS域名解析申请与使用教程，想要寻找更多的DNS服务可以进入专题页面：[国内外免费DNS域名解析服务汇总列表](https://wzfou.com/mianfei-dns/)。关于[DNS域名解析](https://wzfou.com/tag/dns-yuming/)的经验与教程可以看看：

1. [京东云DNS免费域名解析-支持线路\\地域细分 可添加CAA解析记录](https://wzfou.com/jdcloud-dns/)
2. [放弃免费DNS改用付费DNS-Google cloud DNS申请使用与解析效果](https://wzfou.com/google-cloud-dns/)
3. [DNS域名解析启用DNSSEC防止DNS劫持-Google Cloud DNS设置DNSSEC](https://wzfou.com/dnssec-dns/)

**PS：2018年7月27日更新，**如果想要AnyCast DNS可以试试：[NS1 DNS域名解析使用-Master/Slave主从DNS和世界各地分区解析](https://wzfou.com/ns1-dns/)。

## 一、He.net DNS添加域名

网站：

1. https://dns.he.net/

用你的He.net DNS登录，没有账号的也可以直接免费注册一个。

[![He.net DNS登录账号](images/20200219115952004_22278.webp)](https://wzfou.com/wp-content/uploads/2018/07/he.net_01.gif)

接着，点击左边的添加域名，这里输入你要解析的域名。

[![He.net DNS添加域名](images/20200219115951895_21861.webp)](https://wzfou.com/wp-content/uploads/2018/07/he.net_03.gif)

然后，到你的域名注册商那里，将NS服务器修改为He.net DNS的NS服务器。

> ns2.he.net
> 
> ns3.he.net
> 
> ns4.he.net
> 
> ns5.he.net

如下图：

[![He.net DNS修改NS服务器](images/20200219115951787_26218.webp)](https://wzfou.com/wp-content/uploads/2018/07/he.net_04.gif)

添加完了域名就可以等新的NS服务器生效后添加DNS解析记录了。

[![He.net DNS等待生效](images/20200219115951667_426.webp)](https://wzfou.com/wp-content/uploads/2018/07/he.net_05.gif)

## 二、He.net DNS解析记录

He.net 支持的域名解析记录有：A、AAAA、CNAME、MX、NS、TXT、CAA、AFSDB、HINFO、RP、LOC、NAPTR、PTR、SSHFP、SPF、SRV，除了[DNSSEC](https://wzfou.com/tag/dnssec/)，几乎所有的记录He.net DNS都支持。

[![He.net DNS支持的解析记录](images/20200219115951552_16030.gif)](https://wzfou.com/wp-content/uploads/2018/07/he.net_06.gif)

He.net DNS可以设置最低TTL为5分钟，即300秒。

[![He.net DNS最低TTL](images/20200219115950442_28979.gif)](https://wzfou.com/wp-content/uploads/2018/07/he.net_07.gif)

## 三、He.net DDNS动态域名解析

有的时候我们可能需要用到DDNS，即动态域名解析，启用He.net DDNS也很简单，只需要在添加A记录时勾选**Enable entry for dynamic dns。**

[![He.net DNS启动动态解析](images/20200219115949430_28502.webp)](https://wzfou.com/wp-content/uploads/2018/07/he.net_08.gif)

然后回到域名管理界面，点击生成一个DDNS的Key。

[![He.net DNS生成Key](images/20200219115949320_2129.gif)](https://wzfou.com/wp-content/uploads/2018/07/he.net_09.gif)

**如何使用He.net DDNS？**如果你是在[树莓派Raspberry Pi](https://wzfou.com/tag/raspberry-pi/)等Linux操作系统中使用，可以直接使用定时任务，用Get或者Curl的方式搞定。代码示例如下：

```
自动检测 IPv4/IPv6 address:
curl -4 "https://ddns.wzfou.com:password@dyn.dns.he.net/nic/update?hostname=ddns.wzfou.com"					
curl -6 "https://ddns.wzfou.com:password@dyn.dns.he.net/nic/update?hostname=ddns.wzfou.com"					

指定 IPv4/IPv6 address:
curl "https://ddns.wzfou.com:password@dyn.dns.he.net/nic/update?hostname=ddns.wzfou.com&myip=192.168.0.1"					
curl https://ddns.wzfou.com:password@dyn.dns.he.net/nic/update?hostname=ddns.wzfou.com&myip=2001:db8:beef:cafe::1
```

如果你是用DD-WRT等路由器固件，可以直接在后台直接添加，如下图：

[![He.net DNS在路由器中添加](images/20200219115948109_32075.gif)](https://wzfou.com/wp-content/uploads/2018/07/he.net_16.gif)

## 四、He.net IPv6 地址

1. https://tsukkomi.org/post/get-free-ipv6-tunnel

登录你的账号，然后点击左侧的“Create Regular Tunnel”。

[![He.net DNS新建IPv6](images/20200219115947196_28245.gif)](https://wzfou.com/wp-content/uploads/2018/07/he.net_10.gif)

在左侧的菜单中填写你的IP地址，如果是VPS等服务器直接填写你的服务器IP地址，如果你是用的家庭网，要确保你的IP地址是固定的，否则无法通过。在下方选择隧道服务器节点。

[![He.net IPv6 选择节点](images/20200219115946084_16833.webp)](https://wzfou.com/wp-content/uploads/2018/07/he.net_11.gif)

确定好后点击提交，这时你就可以看到你已经成功申请到了[He.net IPv6](https://wzfou.com/tag/he-net-ipv6/) 地址了。

[![He.net IPv6申请到的地址](images/20200219115945976_7099.gif)](https://wzfou.com/wp-content/uploads/2018/07/he.net_12.gif)

**如何使用He.net IPv6 地址？**He.net 已经提供了非常详细的安装方法了，包含了Windows、Linux、Mac、OpenWRT等各类操作系统。

[![He.net IPv6一键安装](images/20200219115944740_6744.webp)](https://wzfou.com/wp-content/uploads/2018/07/he.net_13.gif)

选择一个操作系统后，[He.net](https://wzfou.com/tag/he-net/) 就会给出安装IPv6 隧道的方法，直接执行命令即可。

[![He.net IPv6给出命令](images/20200219115944632_18073.webp)](https://wzfou.com/wp-content/uploads/2018/07/he.net_14.gif)

如果是OpenWRT，你可以直接在后台添加，如下图：（点击放大）

[![He.net IPv6在路由器中添加](images/20200219115944522_5397.gif)](https://wzfou.com/wp-content/uploads/2018/07/he.net_17.gif)

## 五、He.net Slave DNS

Slave DNS即备用DNS或者叫从DNS，为确保DNS解析万无一失，一般地会设置主从DNS。He.net Slave DNS就是从DNS，你可以直接点击添加[He.net Slave DNS](https://wzfou.com/tag/he-net-slave-dns/)。

[![He.net 添加Slave DNS](images/20200219115943604_31048.webp)](https://wzfou.com/wp-content/uploads/2018/07/he.net_18.gif)

## 六、总结

[He.net DNS](https://wzfou.com/tag/he-net-dns/)是一个非常优秀的权威的免费DNS服务，虽然没有分区解析，DNS操作界面也有些难看，但是He.net DNS胜在稳定，支持各类DNS解析记录，功能强大。

[![He.net DNS功能强大](images/20200219115943492_1202.gif)](https://wzfou.com/wp-content/uploads/2018/07/he.net_15.gif)

He.net DNS作为国外的DNS服务，有人可能会担心国内无法使用。经过我的观测，He.net DNS的NS服务器在国内响应虽然不快，但是一直很稳定，监控地址：

1. https://ping.wzfou.com/?target=dnsjk.hedns

[![He.net DNS服务器稳定](images/20200219115942571_24402.gif)](https://wzfou.com/wp-content/uploads/2018/07/he.net_02.gif)
