# Alpine Linux一键安装脚本
Alpine Linux是一个社区开发的面向安全应用的轻量级Linux发行版操作系统，占用资源很少，  
初始状态基本只占用几M内存和几十兆硬盘，而且还很稳定，适合很多小型服务器和设备使用。

你可以通过如下脚本将VPS上现有的Linux系统一键转换为Alpine Linux。但是你需要注意的是，  
脚本只支持openvz6构架的vps，并且VPS上原有数据会全部丢失，安装成功后ssh端口号会变为22，密码不变。  
已经在gullo 德国nat机（openvz7）上试了下，重装后再也无法开机，通过vps面板重装系统也不能开机了。

```
wget --no-check-certificate -O alpine-install.sh https://git.io/JeD5I && bash alpine-install.sh

```

执行完毕后，几分钟即可重新连接SSH。

Alpine常用命令

```
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

```

Alpine服务管理

```
alpine没有使用fedora的systemctl来进行服务管理，使用的是RC系列命令：
rc-update     //主要用于不同运行级增加或者删除服务
rc-status       //主要用于运行级的状态管理
rc-service     //主用于管理服务的状态
rc-status -a  //列出系统所有服务

```

文章来源：[https://cikeblog.com/linux-to-apline.html](https://cikeblog.com/linux-to-apline.html)