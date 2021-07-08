# 用 tinc 组建局域网其实也简单

一直以来用n2n习惯了，觉得非常简单，一行代码就全搞定了。偶尔也尝试过 tinc 的布置，但是一看教程，都是代码帝，跟着跟着就跟丢了，失败了，也就兴趣全没了。这几天比较闲，受朋友们推荐，说这玩意儿好，有人在维护，稳定，又比较容易直连，。。。所以，为了弄清事实真相，那就必须亲自尝试一番。我是先配置通过了，然后删除以前的配置，再重新开始新的尝试，边实践边写下此教程的。你可以完全照搬里面的代码，走通了后再修改成自己的。 硬件情况及约定：要建立 tinc ，你必须要有一台具有公网 IP 的计算机（作服务器），并且要保证能实现端口转发（映射）到这台计算机（默认端口为 655）。这里我们共有 3 台机器：一台是装有 ubuntu-16.04-x86_64 的搬瓦工，我们给它取名 bwh1（其外网IP为12.181.40.118，tinc内网IP为10.0.1.1，它可以做服务器）；一台带 linux 精简系统的群晖 212j（tinc内网IP为10.0.1.2）；另外一台是群晖 213j（tinc内网IP为10.0.1.3）。三台机器的名字分别为 bwh1、212j、213j 如下。以#开头的是注释，#及后面的内容都可以删除，下同。

```
bwh1 10.0.1.1 12.181.40.118    # 做服务器
212j 10.0.1.2                  # 没有外网IP，一般机器
213j 10.0.1.3                  # 一样，一般机器
```

我们的目标是：通过带有外网IP的搬瓦工，实现这 3 台机器之间的互通互联，其实主要是为了实现没有公网IP的 212J 与 213J 之间的互通互联。实际上，在网络状况好的时候，212J 与 213J 可以直连通讯，不好的时候它们之间要通过搬瓦工 bwh1 来转发。 软件情况：请准备 winscp、putty、EditPlus，有了它们，我们就可以脱离代码帝，实现复制粘贴再加简单修改就可以了。 然后说说 tinc 的文件结构：以 bwh1 为例，下面的每行最后一个名字代表一个文件，前面的都是文件夹，假设我们要组建的局域网名字叫 hello，bwh1 上的文件结构如下（windows7 的都在 C:\Program Files\tinc 下）

```
etc__tinc__nets.boot               #（1）
       |___hello__rsa_key.priv     # 这个文件是后面使用命令自动产生的，不是手工编辑出来的
             |____tinc.conf        #（2）
             |____tinc-up          #（3）
             |____tinc-down        #（4）
             |____hosts____bwh1    #（5）此文件夹下最初只有这一个，但最后其他机器的也需放入
usr__sbin__tincd                   # 主程序，也不是手工编辑出来的
```

在后面标注了 1、2、3、4、5 的文件，都可以用 EditPlus 在本地计算机上产生，然后使用 winscp 上传到对应的机器上去即可。rsa_key.priv 是使用命令自动建立的，在生成这个文件的同时，bwh1 这个文件也会被追加 key 代码。 打开EditPlus，选择 文件--新建--标准文本，粘贴一些文本以后，再选择 文件--另存为，填写对应文件名，保存类型 All files (*.*)。第一个文件为 nets.boot，其中内容如下（填入局域网的名字hello）

```
##This file contains all names of the networks to be started on system startup.
hello
```

在本地按照上面的目录结构，我们来创建一个 tinc 的文件夹，在该文件夹下，我们放入第一个文件 nets.boot，然后在 tinc 文件夹下再创建一个 hello 的文件夹（局域网名字是什么就用什么文件夹名，本例为 hello） 使用 EditPlus 创建第二个文件 tinc.conf。

```
Name=bwh1
Interface=tun0
Mode=switch
```

tinc.conf 是 TINC VPN 的一个关键参数，设置好、CPU也好的话，可以跑满你的带宽，所以非常重要。下面做一些说明（许多是可选）：

```
Name = bwh1     # 主机名称
Interface = tun0 # 为网卡名，任意的英文数字皆可
Port = 655         # 默认的通讯端口是 655。如果改成其他的端口，需要在所有机器上增加这一条，并且在 hosts 下也要全加
Mode = switch   # 有三种模式，分别是 router / switch / hub ，相当于路由、交换机、集线器，默认为 router
KeyExpire = 3600
Compression = 0   # 0~11，11是数据快速压缩 0关闭数据压缩 机器cpu配置高选择开启
PingInterval = 10
Digest = sha512   # rsa 加密协议强度 可以选 sha128 sha1
Cipher = aes-128-cbc   # 加密类型，默认的是blowfish，较慢。可以指定任意一个 OpenSSL 所支持的加密算法，
         建议用aes加密，支持硬件加密、速度快、强度高。又如 aes-256-cbc
PMTU = 1460       # 这个非常关键 需要测试你网络运行商的 mtu 值，请搜索
ReplayWindow = 16
IndirectData = no
TCPOnly = no
ProcessPriority = high
```

第三个文件为tinc-down，其内容为

```
ifconfig $INTERFACE down
```

第四个文件为tinc-up，其内容为（内网 IP 地址按照上面的约定来，为 10.0.1.1）

```
ifconfig $INTERFACE 10.0.1.1 netmask 255.255.255.0
```

在 hello 文件夹下再创建 hosts 文件夹 第五个文件为 bwh1，其内容如下。tinc内网 IP 地址是 10.0.1.1，外网IP地址是 12.181.40.118（可以用域名），放入 hosts 文件夹下（不做服务器的机器没有第一行）

```
Address=12.181.40.118
Subnet=10.0.1.1/32
```

第一台机器 bwh1 的文件就这样创建好了。我们现在使用 putty 软件来登录 bwh1，先使用下面的命令来安装 tinc

```
apt-get install tinc
```

没什么出错就算安装好了（成功后显示的代码我就不贴了）。可以

*（用 winscp 软件）*在 /usr/sbin/ 下看到 tincd 这个文件 然后用 winscp 软件把刚才创建的 tinc 目录带文件全部上传到 bwh1 的 etc 目录下面去，并修改 tinc-up 和 tinc-down 的文件属性为 0755（鼠标放文件上，按右键，选属性，即可修改）。至此，bwh1 前期工作算是准备好了，以后也可以在 winscp 窗口下按右键弹出 编辑 对某些文件进行修改。 输入以下代码回车回车再回车，密码文件就自动生成好了。这时你打开 /etc/tinc/hello 文件夹，会看到多了一个 rsa_key.priv 文件（属性为0600），再以编辑的方式打开 /etc/tinc/hello/hosts/bwh1（属性为0644），会发现文件的后面多了一些 key 代码，把这个新的 bwh1 文件下载到本地的 目录 A 下备用。

```
tincd -n hello -K4096
```

在 putty 中输入以下代码启动调试模式（可以通过 Ctrl+\ 来终止）

```
tincd -n hello -D --debug=3
```

看到一个 Ready 结尾的代码就算启动好了，将 putty 这个调试界面保持在这里不动

继续 把 bwh1 的配置文件复制一份，在此基础上修改成 212j 的配置文件 修改 tinc.conf，其内容如下（注意这里增加了一个 ConnectTo=bwh1，是指向服务器的机器名bwh1，也可以用 AutoConnect=yes 代替，让它自动根据 hosts 目录下的设置去找，更方便；不过估计是版本比较老的缘故，我在 ubuntu 和 openwrt 下却不正常，它们的 Tincd 版本都是 1.0.26，而 padavan 下的是 1.1pre14）

```
Name=212j
ConnectTo=bwh1
Interface=tun0
Mode=switch
```

修改 tinc-up，内容如下，内网 IP 地址按开始的约定为 10.0.1.2

```
ifconfig $INTERFACE 10.0.1.2 netmask 255.255.255.0
```

将 bwh1 文件改名为 212j，其内容修改成下面这样的，注意内网 IP 是 10.0.1.2

```
Subnet=10.0.1.2/32
```

使用 putty 登录 212J，编辑主程序。

*使用 http://www.lucktu.com/archives/755.html 这里的方法，进入 Chroot 环境，在该环境下，使用 apt-get install tinc 的方式，安装主程序*，用 winscp 软件把 *Chroot 环境下生成的主程序* tincd 拷贝出来，放到 /usr/sbin 目录下，并修改其属性为 0755。 然后再使用 winscp，把刚才编辑好的 tinc 目录连带文件上传到 etc 目录下，并修改 tinc-down、tinc-up 两个文件的属性为0755。 回到 putty 界面下，输入以下代码回车回车再回车，

```
tincd -n hello -K4096
```

之后将 /etc/tinc/hello/hosts/212j 文件拷贝到本地 A 目录备用 如法炮制，完成对 213J 机器的操作（以 212j 机器的配置文件为蓝本进行修改） 修改 tinc.conf，其内容如下

```
Name=213j
ConnectTo=bwh1
Interface=tun0
Mode=switch
```

修改 tinc-up，其内容如下

```
ifconfig $INTERFACE 10.0.1.3 netmask 255.255.255.0
```

将 212j 文件改名为 213j，其内容为

```
Subnet=10.0.1.3/32
```

使用 putty 登录 213J，用上面的方法生成 tincd 主程序，用 winscp 把它放到 /usr/sbin 目录下，并修改其属性为 0755。 继续使用 winscp，把刚才编辑好的 tinc 目录连带文件上传到 etc 目录下，并修改 tinc-down、tinc-up 两个文件的属性为0755。 回到 putty 界面下，输入以下代码回车回车再回车，

```
tincd -n hello -K4096
```

之后将 /etc/tinc/hello/hosts/213j 文件拷贝到本地 A 目录 此时 A 目录下已经有 3 个文件了，他们分别是 bwh1、212j、213j，把他们全部拷贝到每一个机器的 /etc/tinc/hello/hosts 文件夹下 之后在 212j 和 213j 上面，使用 putty 输入以下代码进入调试模式（第一行一般机器不需要）

```
insmod /lib/modules/tun.ko
tincd -n hello -D --debug=3
```

此时会出现类似以下的信息

```
212j> tincd -n hello -D --debug=3
tincd: /lib/libcrypto.so.1.0.0: no version information available (required by ti                                                                                        ncd)
tincd 1.0.24 (Nov  8 2014 20:19:41) starting, debug level 3
/dev/net/tun is a Linux tun/tap device (tap mode)
Executing script tinc-up
Listening on 0.0.0.0 port 655
Listening on :: port 655
Ready
... ...
```

新开一个 putty，登录 212j，然后 ping 10.0.1.1（或 10.0.1.3），看能否 ping 通，通了就表示大功告成了。如果 ping 不通，请检查你的防火墙有没有阻止它的连接与转发。



上面的方法，意味着每增加一个客户端，就要在每一个机器的 hosts 目录增加一个文件，比较麻烦，下面介绍另外一个加入客户端的方法（在padvan上测试通过，V1.1pre14） 首先在服务器上，在 ssh 下运行下面的命令，其中 xyz 为被邀请者的机器名字。此时会自动在 /etc/tinc/hello/ 产生一个 invitations 的文件夹，里面有一个 key 文件和另外一个临时文件。

```
tinc -n hello invite xyz
```

得到一些信息，将如下语句复制给被邀请方，最好再指定一个内网 IP 给他（以免重复，假设为：10.0.1.4）。

```
12.181.40.118/WOL4l5wX5MsWlaCxYHlmooUSboQG9Ky3C1uLZ2HVbfu1x19G
```

被邀请者在得到这个信息以后，首先要在自己的机器上安装好 tinc，然后在 ssh 下运行下面的语句（ubuntu16.04x64 对于 padavan 的邀请，表示不接受，它的版本是tinc v1.0.26，估计是因为默认的太老了）

```
tinc join 12.181.40.118/WOL4l5wX5MsWlaCxYHlmooUSboQG9Ky3C1uLZ2HVbfu1x19G
```

得到类似下面的信息即算成功了。同时会自动生成 /etc/tinc/hello/ 文件夹及相应文件，也会在服务器的 invitations 文件夹下删除当初的那个临时文件，并在 hosts 下生成 xyz 文件。

```
/opt/home/admin # tinc join 12.181.40.118/WOL4l5wX5MsWlaCxYHlmooUSboQG9Ky3C1uLZHVbfu1x19G
Connected to 12.181.40.118 port 655..........
...............+++++ p
...............+++++ q
Configuration stored in: /opt/etc/tinc/hell
Invitation succesfully accepted.
```

接着修改 tinc-up 和 hosts 下的 xyz 文件，在其中加入 IP 信息分别如下两句

```
ifconfig $INTERFACE 10.0.1.4 netmask 255.255.255.0
Subnet=10.0.1.4/32
```

最后启动被邀请者的 tinc，即可成功加入网络。 **多服务器支持**：上例中，如果 213j 也有外网IP，那么 212j 和 bwh1 两台机器的 tinc.conf 都可以增加一行这样的文字 ConnectTo=213j ，同时 /etc/tinc/hello/hosts/213j 这个文件里面还应该有这样的语句（参考bwh1）Address=116.181.40.23 （此IP地址为 213J 实际的外网IP地址或域名）。这就是多服务器的意思，当一个服务器挂了，另外一个服务器就会起到维护整个局域网的作用。做服务器的主机，某通讯端口必须要可以被外网访问，要设置好端口映射。本例中使用的是默认的端口 655（防火墙上 TCP 和 UDP 都要打开）。你也可以在 /etc/tinc/hello/hosts/bwh1 和 /etc/tinc/hello/tinc.conf 文件里增加一行 Port=1190 以改变成新的端口 1190，并且全部的机器都得有。本例中针对 bwh1 不需设置端口映射即可使用，实际上你可能要设置的，还有防火墙也必须放行。例如用群晖 213j 做服务器时，上级路由器要设置 655 端口映射到 213j，213j 上也要设置 655 端口通行才可以。 关于计算机重启本服务也跟着启动的问题，针对ubuntu，只要按照前面的方法设置好 nets.boot 就可以了。而针对群晖则需要把下面的语句加入到启动文件中（/etc/rc.local）。一般机器第一行不需要。

```
insmod /lib/modules/tun.ko
tincd -n hello
```

另外，在 padavan 下，启动 entware 环境后，也可以直接安装 tinc（安装的版本是 1.1pre14）安装方法如下

```
opkg update
opkg install tinc
```

安装好以后，上传修改好的配置文件（配置文件位置在 /opt/etc 下面），并修改部分文件的属性，然后再生成密码。只是产生密码的方式，需如下操作。

```
tinc -n hello generate-keys
```

启动、停止、重启、重新加载配置文件的方法如下：

```
tinc -n hello start
tinc -n hello stop
tinc -n hello restart
tinc -n hello reload
```

加入 padavan 的开机启动的方法为，放到“在路由器启动后执行”里，命令前要带上路径，好像一次不行，要运行两次才可以起来！我的方法是在前面运行一个 ps 命令来代替，真奇怪！我的是纯净版 padavan，是这样。也许你的没有这个问题。

```
ps | grep tinc
/opt/sbin/tincd -n hello
```

windows 的设置方法见

**[这个链接](https://nwgat.ninja/simplified-tinc-1-1-on-windows-10/)**。其方法大概是说，先在服务器上发出邀请，然后在客户端（windows10）上加入即可。然而我的电脑是 windows7，安装目录 C:\Program Files\tinc 下面没有 tinc 这个主程序，只有 tincd 这个文件，而且我的程序是官方最新的 1.0.35，那就不能用这个方法，只能用首页 bwh1 的方法了，在 C:\Program Files\tinc 下面自建 hello 这个文件夹及其文件（所有命令是进入 C:\Program Files\tinc 运行）。 设置完成，如果以前没有安装虚拟网卡，要先进入 C:\Program Files\tinc\tap-win32，运行 addtap.bat 添加虚拟网卡*（我机器上装有 n2n，可以直接用它的网卡，这一步就可以省掉了）*。安装好后，先简单运行一下

```
tincd -n hello -D --debug=3
```

然后看看 “控制面板\网络和 Internet\网络连接” 中，哪一个网卡

*（我的 n2n 下安装了多个虚拟网卡）*随着 tincd 的启动而启动了，然后停掉上面的 tincd，设置这个网卡的 IP 为 10.0.1.5 和 子网掩码 255.255.255.0（设置 IP 地址的过程也可以参考上面 windows10 的方法设置）。然后使用下面的命令，正式后台运行 tinc 吧：

```
tincd -n hello
```

今日无意当中见到了 **[windows7 上的安装方法](http://tinc.link/examples/windows-install/)**，但是我简单设置了下（没完全按该教程走），没有成功，不折腾了，等今后确实需要了再来。

我觉得这个 tinc 可以布置在 路由器 和 群晖 等 linux 类服务器上去，然后借鉴本站 n2n 里面的方法，组建 网对网 的局域网，而不仅仅是单个机器之间的联网（遗憾的是目前 ubuntu16.04 上安装的 tinc 是 1.0.26-1 的，它做服务器，被你被 padvan 所用，但反过来可以，padvan 上的 tinc 是 1.1pre17，不知道谁更新，也找不到合适的版本）。

——————————————————————————观后絮语—————————————————————————— 我觉得，TINC 与 N2N 属于一类，完全自主、完全开源，适合有强迫症的一族。我简单对比一下他们的特点吧（TINC 我用的不多，经验不足，仅做参考）： 1、感觉 TINC 更成熟一些（2018-12-1），估计直连成功率和传输速度更好一些 2、TINC 的服务器私用，不能共享；N2N 可以共享，一个服务支持多个使用中，相互独立 3、N2N 设置更简单一些，一行代码走遍天下 4、也需正是因为它的一行代码，泄露了太多机密，N2N 可能没有 TINC 安全 自己觉得哪一个更适合自己就用哪一个吧！ 更多资料：https://github.com/qmcclab/blog/blob/master/2013-09-20-a-mesh-network.md