# virmach 1刀神机硬盘安装 Alpine Linux 系统教程
参考:[https://liujia.anqun.org/index.php/archives/1192/](https://liujia.anqun.org/index.php/archives/1192/)

挂载 Windows Offline NT Password & Registry Editor 镜像

![image.png](https://i.loli.net/2019/12/06/3eVIloDYawGkRqE.png)

设置启动方式为优先 CDROM

![image.png](https://i.loli.net/2019/12/06/Rw9JdBksArGK5ti.png)

reboot 进 VNC ，选择 Finnix 64位启动

![image.png](https://i.loli.net/2019/12/06/sRSmocJpMyrig91.png)

passwd \# 设置 root 密码  
vi /etc/ssh/sshd_config  \# 修改 ssh 配置文件允许root登录(PermitRootLogin 改为yes)  
service ssh start \# 开启 ssh

接下来用 xshell 或 putty 等 ssh 工具登录 Finnix 方便操作

mkfs.ext4 /dev/sda \# 格式化1g硬盘  
mkdir /mnt/custom  
mount /dev/sda /mnt/custom/ \# 挂载硬盘

apt update  
apt install wget  
wget http://dl-cdn.alpinelinux.org/alpine/latest-stable/main/x86_64/apk-tools-static-2.10.4-r2.apk \# 下载 apk tools  
tar xzvf apk-tools-static-2.10.4-r2.apk  
./sbin/apk.static -X http://dl-cdn.alpinelinux.org/alpine/v3.1/main -U --allow-untrusted --root /mnt/custom/ --initdb add alpine-base \# 安装 alpine3.1，最新版内核太高，内存不够无法启动

cp /etc/resolv.conf /mnt/custom/etc/  
mkdir -p /mnt/custom/root  
mkdir -p /mnt/custom/etc/apk  
echo "http://dl-cdn.alpinelinux.org/alpine/v3.1/main" \> /mnt/custom/etc/apk/repositories \# 设置apk源

创建相应设备目录：

mknod -m 666 /mnt/custom/dev/full c 1 7  
mknod -m 666 /mnt/custom/dev/ptmx c 5 2  
mknod -m 644 /mnt/custom/dev/random c 1 8  
mknod -m 644 /mnt/custom/dev/urandom c 1 9  
mknod -m 666 /mnt/custom/dev/zero c 1 5  
mknod -m 666 /mnt/custom/dev/tty c 5 0

挂载目录

mount -t proc none /mnt/custom/proc  
mount -o bind /sys /mnt/custom/sys  
mount -o bind /dev /mnt/custom/dev

![image.png](images/20200220020858034_17839.png)  在控制台的 network 中点击 ip 查看网卡信息，也可以等开机点控制台的 Reconfigure Networking 按钮自动配置网卡（装完以后如果没网可以这样修复）

`chroot /mnt/custom /bin/sh -l` # chroot 到 apline 目录

`vi /etc/network/interfaces` # 创建网卡配置文件，写入自己的网卡配置：

 auto lo  
 iface lo inet loopback  
​  
 auto eth0  
 iface eth0 inet static  
 address 107.175.xxx.xxx  
 gateway 107.175.xxx.xxx  
 netmask 255.255.255.224  
 dns-nameservers 8.8.8.8 8.8.4.4

设置 ssh

apk add openssh \# 安装 openssh  
vi /etc/ssh/sshd_config  \# 修改 ssh 配置文件允许root登录(PermitRootLogin 改为yes)  
passwd \# 创建 root 密码

设置可启动的服务

rc-update add devfs sysinit  
rc-update add dmesg sysinit  
rc-update add mdev sysinit  
rc-update add hwclock boot  
rc-update add modules boot  
rc-update add sysctl boot  
rc-update add hostname boot  
rc-update add bootmisc boot  
rc-update add syslog boot  
rc-update add mount-ro shutdown  
rc-update add killprocs shutdown  
rc-update add savecache shutdown  
rc-update add networking boot  
rc-update add urandom boot  
rc-update add acpid default  
rc-update add hwdrivers sysinit  
rc-update add crond default

apk add linux-vanilla syslinux \# 安装内核和引导  
dd bs=440 count=1 if=/usr/share/syslinux/mbr.bin of=/dev/sda \# 将mbr引导写到磁盘中  
extlinux -i /boot  
blkid /dev/sda  \# 查看硬盘uid  
sed -i -e "s:^root=.*:root=UUID=9d120ecb-8c8c-49f1-8e75-xxxxxxxxxx:" /etc/update-extlinux.conf \# 将硬盘 uuid 写入配置文件  
sed -i -e "s:^modules=.*:modules=sd-mod,usb-storage,ext3,ext4:" /etc/update-extlinux.conf \# 添加 ext4 支持  
update-extlinux \# 更新引导  
echo "UUID=9d120ecb-8c8c-49f1-8e75-xxxxxxxxxx / ext4 defaults 1 1" \> /etc/fstab  \# 将硬盘信息写到文件系统配置文件中

重启