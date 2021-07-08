# proxmox使用chroot运行alpine（proxmox下容器的另类使用）2019-10-25

> chroot命令用来在指定的根目录下运行指令。chroot，即 change root directory （更改 root 目录）。在 linux 系统中，系统默认的目录结构都是以/，即是以根 (root) 开始的。而在使用 chroot 之后，系统的目录结构将以指定的位置作为/位置。
>  chroot可以算是容器的始祖了。

## 一、准备工作

下载alpine的chroot包，别害怕，只有2.6M



![img](https:////upload-images.jianshu.io/upload_images/4171480-e531bbb230408c40.png?imageMogr2/auto-orient/strip|imageView2/2/w/1131/format/webp)

images



pve肯定是选x86_64了
 为了省事，直接在terminal里wget好了：



```ruby
mkdir /alpine   #我这里放到/alpine好了
cd /alpine
wget http://dl-cdn.alpinelinux.org/alpine/v3.10/releases/x86_64/alpine-minirootfs-3.10.3-x86_64.tar.gz
```

下载后，解压



```css
tar -xvzf alpine-minirootfs-3.10.3-x86_64.tar.gz
```

完事了切回/目录操作：



```bash
cd /
```

## 二、chroot操作

刚才切到/目录了，操作如下：



```objectivec
mount -t proc /proc /alpine/proc
mount --rbind /dev /alpine/dev
mount --rbind /sys /alpine/sys
#如果需要卸载rbind，需要先mount --make-rslave /alpine/sys，再umount -R /alpine/sys , dev也是这样
#schroot --list --all-sessions
#schroot -e -c precise-ca6c72e4-0e9f-4721-8a0e-cca359e2c2fd
#批量删除：for i in `schroot --list --all-sessions|awk -F ":" '{print $2}'`;do schroot -e -c $i;done
```

chroot:



```sh
chroot /alpine /bin/sh   #这里注意，要用/bin/sh，alpine里默认没有bash
```

然后，会发现前面的提示符变了。说明已经进chroot了。

## 三、安装和使用软件

1. 为alpine添加阿里的源，加速啊，速度明显加快：



```sh
echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories \
 && echo "http://mirrors.aliyun.com/alpine/latest-stable/community/"  >> /etc/apk/repositories
```

1. 安装lm_sensors验证是否可以调用传感器（这里要注意包名，是下划线，不是debian的中横线）：



```csharp
#安装
apk add lm_sensors
#执行
sensors
```

如果没有意外，就已经看到结果了。具体apk的使用，请`apk --help`或自行查找资料。

1. 查看硬盘信息：



```csharp
ls -alh /dev/disk/by-id
```

## 四、退出



```bash
exit
```

回到主系统。

------

## 五、如何不进主系统自动执行chroot中的程序

待续...



作者：龙天ivan
链接：https://www.jianshu.com/p/0291cb935e50
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。