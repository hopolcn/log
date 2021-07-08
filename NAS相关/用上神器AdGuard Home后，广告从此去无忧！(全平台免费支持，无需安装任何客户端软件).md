# 用上神器AdGuard Home后，广告从此去无忧！(全平台免费支持，无需安装任何客户端软件)

本文首发于：微信公众号「运维之美」，公众号 ID：Hi-Linux。

「运维之美」是一个有情怀、有态度，专注于 Linux 运维相关技术文章分享的公众号。公众号致力于为广大运维工作者分享各类技术文章和发布最前沿的科技信息。公众号的核心理念是：分享，我们认为只有分享才能使我们的团体更强大。如果你想第一时间获取最新技术文章，欢迎关注我们！

公众号作者 Mike，一个月薪 3000 的杂工。从事 IT 相关工作 15+ 年，热衷于互联网技术领域，认同开源文化，对运维相关技术有自己独特的见解。很愿意将自己积累的经验、心得、技能与大家分享交流，篇篇干货不要错过哟。如果你想联系到我，可关注公众号获取相关信息。

### 



## 什么是 AdGuard Home

`AdGuard Home` 是一款全网广告拦截与反跟踪软件，`AdGuard Home` 项目是著名广告拦截器提供商 `AdGuard` 开源的一个 `DNS Server` 版本。`AdGuard Home` 可以将广告与追踪相关的域名屏蔽，同时你不再需要安装任何客户端软件。`AdGuard Home` 的工作原理是在 `DNS` 的域名解析过程里拦截网页上的广告。

简单来说 `AdGuard Home` 是一个支持广告过滤和家长控制的开源公共 `DNS` 服务，如同 Google 的公共 DNS 服务 8.8.8.8。`AdGuard Home` 同时也支持 `DNS over TLS` 和 `DNS over HTTPS`。

> 项目地址：https://github.com/AdguardTeam/AdGuardHome

**AdGuard Home 的主要功能介绍**

- 拦截随处可见的广告
- 注重隐私保护
- 家庭保护模式
- 自定义过滤规则

在继续讲解前，我们先来看一看 `AdGuard Home` 强大的功能演示和管理后台。

![img](images/16d7ff7bd0f7b651)

## 安装 AdGuard Home

`AdGuard Home` 使用 `Golang` 开发，具有良好的原生跨平台性。它可以部署在 `X86` 架构的各种操作系统上，也可以部署在树莓派上，甚至你还可以借助 `Docker` 部署在群晖 `NAS` 上。

### 使用预编译的二进制版本安装

这里我们以 `Linux` 系统为例，其它系统可参考官方帮助文档：https://github.com/AdguardTeam/AdGuardHome/wiki/Getting-Started#installation 。

```
# 下载并解压 AdGuard Home
$ wget https://github.com/AdguardTeam/AdGuardHome/releases/download/v0.98.1/AdGuardHome_linux_amd64.tar.gz
$ tar -zxvf AdGuardHome_linux_amd64.tar.gz

# 为了方便使用，我们将二进制文件拷贝到 PATH 所包含的位置
$ cd AdGuardHome_linux_amd64
$ cp ./AdGuardHome /usr/local/bin/

# 启动 AdGuard Home
$ AdGuardHome复制代码
```

上面的方法，很显然是在前台运行的。前台运行必然还是存在一些弊端的，比如：当前 `SHELL` 中断必然会引起程序中断等。如果你想长期稳定的运行 `AdGuard Home`，最后好方法必然是将 `AdGuard Home` 运行成一个服务。要想将 `AdGuard Home` 在各平台部署为服务也是很简单的，只需运行下面这一条命令就可实现。

```
# Linux 下使用的服务管理器是 systemd 、Upstart 或 SysV，macOS 下使用的服务管理器是 Launchd。
$ AdGuardHome -s install复制代码
```

`AdGuard Home` 服务安装后好，你可以使用以下命令来管理它。

```
# 启动 AdGuardHome 服务
$ AdGuardHome -s start

# 停止 AdGuardHome 服务
$ AdGuardHome -s stop

# 重启 AdGuardHome 服务
$ AdGuardHome -s restart

# 查看 AdGuardHome 服务状态
$ AdGuardHome -s status

# 卸载 AdGuardHome 服务
$ AdGuardHome -s uninstall复制代码
```



### 使用 Docker 来安装

如果你会一点点 `Docker` 知识的话，我们当然还是建议你直接使用 `Docker` 来安装。虽然通过预编译的二进制版本安装已经很简单了，但如果使用 `Docker` 来安装，你会发现仅仅只需一条指令就可以搞定了。

```
$ docker pull adguard/adguardhome
# -v 参数后面指定的宿主机上的目录主要用作永久保存 AdGuard Home 的数据文件和配置文件，可自行根据实际情况修改。
$ docker run --name adguardhome -v /home/mike/workdir:/opt/adguardhome/work -v /home/mike/confdir:/opt/adguardhome/conf -p 53:53/tcp -p 53:53/udp -p 67:67/udp -p 68:68/tcp -p 68:68/udp -p 80:80/tcp -p 443:443/tcp -p 853:853/tcp -p 3000:3000/tcp -d adguard/adguardhome复制代码
```

你可能会发现上面一共是两条指令，前面不是说好了是一条指令的吗？是不是发现被骗了，我怎么可能骗你呢，这绝对是不可能的！其实这两条指令，你只需直接执行第 2 条指令就可以完成所有安装操作了。这里分开写出来仅仅是为了完整演示 `Docker` 整个运行过程，能让一些还不会 `Docker` 的同学能更容易理解一些。前面既然啰嗦了这么多，这里就再延伸说一点 `Docker` 容器的基本管理操作。

```
# 启动 AdGuard Home 容器
$ docker start adguardhome
# 停止 AdGuard Home 容器
$ docker stop adguardhome
# 删除 AdGuard Home 容器
$ docker rm adguardhome复制代码
```



## 使用 AdGuard Home



### 使用默认配置来设置 AdGuard Home

运行 `AdGuard Home` 后，我们需要通过浏览器打开 `http://IP:3000` 对 `AdGuard Home` 进行初始化设置。首次初始化会要求设置服务运行端口、账号、密码等信息，配置过程中设置的密码一定请牢记，下次登录管理后台时需要使用。

![img](images/16d7ff7fab86fc4d)

首先，我们点击 “开始配置” ，来设定网页管理界面和 `DNS` 服务的端口。

![img](images/16d7ff7febe246cc)

其次，点击 “下一步” 后，为 `AdGuard Home` 网页管理界面设置一个用户名和密码。

![img](images/16d7ff8013b6710b)

最后，点击 “下一步” 后，`AdGuard Home` 会展示以上配置的汇总信息。

![img](images/16d7ff80394a277e)

至此，使用 `AdGuard Home` 默认配置的设置就算大功告成了。

![img](images/16d7ff8064f70eda)

使用 `AdGuard Home` 默认配置设置完成后，我们可以在「仪表盘」上看到 `DNS` 查询次数、被过滤器封锁的网站、查询 `DNS` 请求的客户端 `IP` 地址等等信息。

![img](images/16d7ff8089245b92)

### AdGuard Home 配置进阶

`AdGuard Home` 默认的配置比较简单，为了更强力地拦截广告，我们可以对 `AdGuard Home` 配置进行一些优化。

1. 常规设置

`AdGuard Home` 默认配置的情况下只勾选了「使用过滤器和 Hosts 文件以拦截指定域名」这一个选项，你可以根据自身情况决定是否启用「使用 AdGuard 浏览安全网页服务」、「使用 AdGuard 家长控制服务」和「强制安全搜索」等特性。

不仅如此，你还可以很方便的屏蔽一些比较流行的网站。当然这些网站本来对我们都是不可用的，也就不用多此一举进行设置了，哈哈！

![img](images/16d7ff80b759a271)

1. 设置上游 DNS

`AdGuard Home` 默认使用 `Cloudflare` 的 `DNS over HTTPS` 作为上游服务器。如果你在国内使用 `Cloudflare DNS` 做为上游 `DNS`，可能延迟会比较高。

我们可以设置为国内的公共 `DNS`，如：腾讯的 `119.29.29.29`、阿里的 `223.5.5.5` 和 `114.114.114.114` 等，但坏处是这些国内公共 `DNS` 暂时不支持 `DNS over TLS`。

这里有一个比较折中的解决方法就是通过启用 「通过同时查询所有上游服务器以使用并行查询加速解析」选项来在每次查询的时候对所有的上游 `DNS` 同时查询，以加速解析速度。

![img](images/16d7ff80e10f5470)

1. 过滤器

虽然 `AdGuard Home` 本身内置了比较知名的 `AdGuard`、`AdAway` 广告过滤规则，但这些规则在国内显然有点水土不服。如果你想要更完美的实现广告屏蔽还需要自己添加规则，比较幸运的是 `AdGuard Home` 是可以兼容 `Adblock` 过滤规则语法的。这样，你就可以很方便的使用一些比较知名的 `Adblock` 过滤规则，比如：由 `Adblock Plus` 团队维护的 `EasyList`。

![img](images/16d7ff8116088116)

![img](images/16d7ff8144306c5e)

目前好用的广告过滤规则还是有很多的，它们都针对不同的用途。下面推荐一些比较常用的：

> \1. EasyList China : 国内网站广告过滤的主规则。
>
> 链接：https://easylist-downloads.adblockplus.org/easylistchina.txt
>
> \2. EasyPrivacy : EasyPrivacy 是隐私保护，不被跟踪。
>
> 链接：https://easylist-downloads.adblockplus.org/easyprivacy.txt
>
> \3. CJX's Annoyance List : 过滤烦人的自我推广，并补充 EasyPrivacy 隐私规则。
>
> 链接：https://raw.githubusercontent.com/cjx82630/cjxlist/master/cjx-annoyance.txt
>
> \4. 广告净化器规则 : 支持国内大部分视频网站的广告过滤。
>
> 链接：http://tools.yiclear.com/ChinaList2.0.txt
>
> \5. I don't care about cookies : 我不关心 Cookie 的问题，屏蔽网站的 cookies 相关的警告。
>
> 链接：https://www.i-dont-care-about-cookies.eu/abp/

除了使用已有的过滤规则外，当然你也可以根据自己的需求自定义过滤规则，要自定义过滤规则其实也很简单。

![img](images/16d7ff816ae3c858)

下面是自定义过滤规则的一些语法说明。

```
||example.org^ – 拦截 example.org 域名及其所有子域名
@@||example.org^ – 放行 example.org 及其所有子域名
127.0.0.1 example.org – 将会把 example.org（但不包括它的子域名）解析到 127.0.0.1。
! 注释符号，表示这是一行注释
# 这也是注释符号，同样表示这是一行注释
/REGEX/ – 正则表达式模式复制代码
```

更多规则可以参考官方帮助文档：https://kb.adguard.com/en/general/dns-filtering-syntax

1. 查询日志

`AdGuard Home` 管理界面中也为我们提供了 `DNS` 请求日志查询功能，在这里，我们不但能看见所有设备最近 5000 条的 `DNS` 请求日志记录。你还可以根据 `DNS` 请求日志记录来针对某个域名进行快速的拦截和放行操作。

![img](images/16d7ff81929eedd6)

1. 调整配置参数，以提升 QPS 能力

`AdGuard Home` 所有的配置参数都保存在一个名为 `AdGuardHome.yaml` 的配置文件中。这个配置文件默认路径通常为 `AdGuard Home` 二进制文件 `AdGuardHome` 所在的目录，比如：`/usr/local/bin/AdGuardHome.yaml`。

这里我们只需调整以下两个参数，就是可以明显提升 `AdGuard Home` 的 `QPS`  能力。

- ratelimit : `DDoS` 保护，客户端每秒接收的数据包数。默认值是 20，建议禁用该参数（将值改为 0）。
- blocked*response*ttl : `TTL` 缓存时间，默认值是 10，建议设置为 60 。

这里在把 `AdGuard Home` 的配置文件完整版本也展示一下，有兴趣的同学可以自行研究下其它参数的用途哟！。

```
$ cat AdGuardHome.yaml

bind_host: 0.0.0.0
bind_port: 80
auth_name: mike
auth_pass: "123456"
language: zh-cn
rlimit_nofile: 0
dns:
  bind_host: 0.0.0.0
  port: 53
  protection_enabled: true
  filtering_enabled: true
  blocking_mode: nxdomain
  blocked_response_ttl: 60
  querylog_enabled: true
  ratelimit: 0
  ratelimit_whitelist: []
  refuse_any: true
  bootstrap_dns:
  - 1.1.1.1:53
  - 1.0.0.1:53
  all_servers: true
  allowed_clients: []
  disallowed_clients: []
  blocked_hosts: []
  parental_block_host: ""
  safebrowsing_block_host: ""
  blocked_services: []
  parental_sensitivity: 13
  parental_enabled: true
  safesearch_enabled: true
  safebrowsing_enabled: true
  resolveraddress: ""
  rewrites: []
  upstream_dns:
  - https://1.1.1.1/dns-query
  - https://1.0.0.1/dns-query
  - 119.29.29.29
  - 223.5.5.5
tls:
  enabled: false
  server_name: ""
  force_https: false
  port_https: 443
  port_dns_over_tls: 853
  certificate_chain: ""
  private_key: ""
filters:
- enabled: true
  url: https://adguardteam.github.io/AdGuardSDNSFilter/Filters/filter.txt
  name: AdGuard Simplified Domain Names filter
  id: 1
- enabled: false
  url: https://adaway.org/hosts.txt
  name: AdAway
  id: 2
- enabled: false
  url: https://hosts-file.net/ad_servers.txt
  name: hpHosts - Ad and Tracking servers only
  id: 3
- enabled: false
  url: https://www.malwaredomainlist.com/hostslist/hosts.txt
  name: MalwareDomainList.com Hosts List
  id: 4
- enabled: true
  url: https://easylist-downloads.adblockplus.org/easylistchina.txt
  name: EasyList China
  id: 1569209532
user_rules:
- '@@mps.ts'
dhcp:
  enabled: false
  interface_name: ""
  gateway_ip: ""
  subnet_mask: ""
  range_start: ""
  range_end: ""
  lease_duration: 86400
  icmp_timeout_msec: 1000
clients: []
log_file: ""
verbose: false
schema_version: 4复制代码
```



### 设置客户端 DNS

所有以上设置完成后，最后当然是修改所有客户端的 `DNS` 设置，来享用 `AdGuard Home` 带来的强大的去广告功能。

这个其实真的不用写，我想聪明的你应该都知道这个怎么设置。写这个标题仅仅是为了保持文档完整性，如果你真的不会设置，那就请自行使用「[一些好用](https://mp.weixin.qq.com/s?__biz=MzI3MTI2NzkxMA==&mid=2247488197&idx=1&sn=1f722503d5f18ab8c6f4d9ba768d983b&chksm=eac533ecddb2bafa6044ce24599ebd20b6fd1cd083a31b53d16730f35daaaffbbb902fc8d449&token=614394592&lang=zh_CN#rd)」的搜索引擎搜索相关方法吧！

## 总结

`AdGuard Home` 不但支持了 `macOS`、`Windows`、`Linux`、树莓派等多个系统平台，也提供了二进制和 `Docker` 的部署方式，让安装变得非常简单。`AdGuard Home` 自身提供的强大和直观的管理和统计系统，让它使用起来也是非常方便的。如果你打算自建一个支持去广告功能的公共 `DNS`，`AdGuard Home` 是非常值得一试的不二选择。

## 参考文档



1. https://www.google.com
2. https://zhuanlan.zhihu.com/p/56804257
3. https://www.xiaoz.me/archives/12318
4. https://www.yangcs.net/posts/adguard-home/
5. https://github.com/AdguardTeam/AdGuardHome#getting-started


作者：运维之美
链接：https://juejin.im/post/5d91666ef265da5b7326d71d
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。