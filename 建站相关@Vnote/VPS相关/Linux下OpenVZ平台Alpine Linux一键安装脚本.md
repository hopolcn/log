# Linux下OpenVZ平台Alpine Linux一键安装脚本
Alpine Linux脚本应该出来好几个月了，一直没仔细研究，想着对我来说应该用处不大，但是昨晚128M内存的小鸡挂着ss突然就挂掉了，再也无法启动，后台查看到内存爆满，初步怀疑是ss连接数过大，导致了内存不足，直接小鸡就挂了，就想起这个Alpine Linux脚本。  
昨天试了一下，这个系统的确不错，其他的先不谈，仅仅开机占用7MB内存就值得一试，并且系统自带了netstat,ifconfig,wget这写脚本需要用到的组件，加上现在Docker官方的镜像底包都推荐使用Alpine来制作，官方推荐的docker.io/alpine:3.7只有4.15M，所以我现在把脚本搬运过来，整理一下写个文章，不然下次写Alpine一键脚本的时候，会不清楚如何安装Alpine。

废话不多，直接上一键脚本(OpenVZ平台，版本不限)：

wget https://d.3s.work/s/alpine.sh && bash alpine.sh

如果在安装过程中有错误提示，忽略即可，大多都是删除文件的是否提示文件不存在而已。安装完成会自动退出终端，我们使用原来的默认密码登陆即可。

如果服务器不停的修改/etc/inittab并添加一堆重新生成的getty，可运行命令：

apk add e2fsprogs-extra  
chattr +i /etc/inittab

安装后内存占用：

![](https://d.3s.work/wp-content/uploads/2018/08/alpine.png)

安装后优化操作：

1.如果发现更新或者安装程序非常慢，可以考虑更新apk源：

echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.8/main/" > /etc/apk/repositories  //清华大学源  
echo "https://mirrors.ustc.edu.cn/alpine/v3.8/main/" > /etc/apk/repositories  //中科大源

2.Alpine常用命令：

apk update //更新最新镜像源列表  
apk search //查找所以可用软件包  
apk search -v //查找所以可用软件包及其描述内容  
apk search -v 'acf*' //通过软件包名称查找软件包  
apk search -v -d 'docker' //通过描述文件查找特定的软件包  
apk add openssh //安装一个软件  
apk add openssh openntp vim //安装多个软件  
apk add --no-cache mysql-client //不使用本地镜像源缓存，相当于先执行update，再执行add  
apk info //列出所有已安装的软件包  
apk info -a zlib //显示完整的软件包信息  
apk info --who-owns /sbin/lbu //显示指定文件属于的包  
apk upgrade //升级所有软件  
apk upgrade openssh //升级指定软件  
apk upgrade openssh openntp vim //升级多个软件  
apk add --upgrade busybox //指定升级部分软件包  
apk del openssh //删除一个软件

3.Alpine服务管理：

alpine没有使用fedora的systemctl来进行服务管理，使用的是RC系列命令：  
rc-update     //主要用于不同运行级增加或者删除服务  
rc-status       //主要用于运行级的状态管理  
rc-service     //主用于管理服务的状态  
rc-status -a  //列出系统所有服务