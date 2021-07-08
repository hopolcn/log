# ubuntu-18.04 设置开机启动脚本

ubuntu-18.04 设置开机启动脚本

参阅下列链接

https://askubuntu.com/questions/886620/how-can-i-execute-command-on-startup-rc-local-alternative-on-ubuntu-16-10

ubuntu-18.04不能像ubuntu14一样通过编辑rc.local来设置开机启动脚本，通过下列简单设置后，可以使rc.local重新发挥作用。

1、建立rc-local.service文件

sudo vi /etc/systemd/system/rc-local.service
2、将下列内容复制进rc-local.service文件

[Unit]
Description=/etc/rc.local Compatibility
ConditionPathExists=/etc/rc.local

[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99

[Install]
WantedBy=multi-user.target
3、创建文件rc.local　　

sudo vi /etc/rc.local
4、将下列内容复制进rc.local文件

#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
echo "看到这行字，说明添加自启动脚本成功。" > /usr/local/test.log
exit 0
5、给rc.local加上权限

sudo chmod +x /etc/rc.local
6、启用服务

sudo systemctl enable rc-local
7、启动服务并检查状态

sudo systemctl start rc-local.service
sudo systemctl status rc-local.service
8、重启并检查test.log文件

cat /usr/local/test.log
如果能看到内容，说明设置成功，你就可以通过编辑rc.local文件来设置启动脚本了