# 在proxmox ve 5.3/5.4中安装Sensors传感器值显示界面2019-05-14

> This is a Sensors's interface install in proxmox ve 5.3/5.4 shell script.

## install guide

1. clone or download the code to your proxmox ve. `git clone https://github.com/ivanhao/PveSensors.git`
2. cd to the path
3. execute the `chmod +x ./*.sh` command to make it have right permission to execute the install or uninstall.
4. `./install.sh` to install.

> Don't forget to make a snapshot for rpool before run it.

> you must run this script as `root`.

## uninstall guide

`./uninstall.sh`
 or
 rollback your zfs rpool's snapshot and reboot.

------

> 这是一个在proxmox ve 5.3/5.4中安装Sensors传感器值显示界面的脚本

## 安装说明

#### 如果这个脚本帮到你了，麻烦点一下右上角的star小星星_

1. clone或下载安装包到proxmox ve里，例如在/root目录运行：`git clone https://github.com/ivanhao/PveSensors.git`。

> 如果你没安装git或提示找不到git命令，用`apt-get install git`安装一下再重新执行。

1. 命令行cd到安装包的路径里`cd PveSensors`。
2. 执行`chmod +x ./*.sh`来让程序有执行的权限。
3. `./install.sh`来安装，根据提示很快就完成了。然后刷新界面。

> 不要忘记执行前给rpool做个快照，以便恢复快照。

> 需要用root账号来运行

## 卸载

`./uninstall.sh` 来卸载。
 或者
 回滚你的rpool快照，重启。

## Preview 预览

![Preview](images/Preview.jpg)



作者：龙天ivan
链接：https://www.jianshu.com/p/4c97893a1693
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。