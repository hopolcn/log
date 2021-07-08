# openwrt 开启 Samba 作为共享中心

https://www.solarck.com/openwrt-samba.html



为 Openwrt 接入一个大 U 盘，不用来作共享中心的话实在没什么用处了，这也是为日后脱机 BT 下载提供一个基础。

### 安装

```
opkg update
opkg install samba36-server luci-app-samba
```

#### 配置文件

samba 的配置文件只有两个，而且默认配置稍作修改就可以使用

```
root@openwrt:~# vi /etc/samba/smb.conf.template
[global]
    netbios name = OpenWrt
    display charset = UTF-8
    interfaces = 127.0.0.1/8 lo 192.168.3.1/24 fd73:3a9a:156::1/60 br-lan #内网IP
    server string = OpenWrt
    unix charset = UTF-8
    workgroup = WORKGROUP
    browseable = yes
    deadtime = 30
    domain master = yes
    encrypt passwords = true
    enable core files = no
    guest account = nobody #匿名用户
    guest ok = yes #匿名用户
    invalid users = root
    local master = yes
    load printers = no
    map to guest = Bad User
    max protocol = SMB2
    min receivefile size = 16384
    null passwords = yes #无需密码
    obey pam restrictions = yes
    os level = 20
    passdb backend = smbpasswd
    preferred master = yes
    printable = no
    security = user
    smb encrypt = disabled
    smb passwd file = /etc/samba/smbpasswd
    socket options = TCP_NODELAY IPTOS_LOWDELAY
    syslog = 2
    use sendfile = yes
    writeable = yes #可写
root@openwrt:~# vi /etc/config/samba
config samba
    option 'name'           'OpenWrt'
    option 'workgroup'      'WORKGROUP'
    option 'description'        'OpenWrt'
    option 'homes'          '1'

config 'sambashare'
    option 'name' 'Shares'
    option 'path' '/share' #samba所在目录
#   option 'users' 'sandra'
    option 'guest_ok' 'yes'
    option 'create_mask' '0777' #所有用户可写
    option 'dir_mask' '0777' #所有用户可写
    option 'read_only' 'no'
```

我的配置是无需密码所有用户都可以访问，可上传可下载。

配置完还需要对目录进行权限提升

```
chmod a+w /share
```

或者更改文件夹用户

```
chown nobody:nobody /share
```

最后重启 samba 服务并开机启动

```
/etc/init.d/samba restart
/etc/init.d/samba enable
```

### 访问

Windows 用户很容易访问，在网络邻居（网络）里就可以看到 WORKGROUP—>OPENWRT—>Share 文件夹了，但是 linux 用户需要一些其他命令。
**1. 安装 g2sc**

```
yaourt -S g2sc
```

安装完就可以像 Windows 一样看到工作组和文件夹，但是只能下载，没有上传功能。

**2. sambclient** 安装工具

```
yaourt -S sambaclient
```

连接主机

```
kevin@kevin:pts/2 ~$: smbclient -L OPENWRT
Enter kevins password:  #没设密码直接回车

    Sharename       Type      Comment
    ---------       ----      -------
    Shares          Disk
    IPC$            IPC       IPC Service (OpenWrt)

    Server               Comment
    ---------            -------
    CHEN-PC
    OPENWRT              OpenWrt

    Workgroup            Master
    ---------            -------
    WORKGROUP            OPENWRT
kevin@kevin:pts/2 ~$: smbclient //OPENWRT/Shares #格式为//Servername/Sharename
smb: \>
```

出现了 smb 的命令行

```
get ****    #下载某个文件
put ****    #上传某个文件
```

更多命令输入?查看

**3. mount 挂载**

```
kevin@kevin:pts/2 ~$: mkdir /mnt/samba
kevin@kevin:pts/2 ~$: sudo mount -t cifs  -l //OPENWRT/Shares /mnt/samba
```

### 完成

由于安装了 Luci，所以开启了 uhttp 服务，把共享目录链接到/www 目录同样可以通过浏览器直接下载，相当于把 Samba 目录同样做成了 FTP 目录。

```
kevin@kevin:pts/2 ~$: ln -s /share /www/share
```

Samba 共享就全部完成，之后再继续研究 BT 下载，配合 Samba 的共享就等于免费拥有了一个简版 NAS。