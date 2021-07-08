## 【暂时保留】玩客云root成功，2021年，下迅雷、挂甜糖、smb，一个也不耽误

https://www.right.com.cn/forum/thread-4092171-1-1.html



先开坑，主要搞法已给出。考虑到可能会破坏生态圈，详细搞法暂时先不发，让我整理整理，过几天一定发，不放鸽子。
有把握的老司机们可以照着下面写的“主要流程”直接开搞。
（毕竟现在玩客云半死不活，全民root大概会导致玩客云直接停止提供下载服务？还请各位大佬指点，如果不用顾虑这个我就发）


教程已发布，参见新帖：[https://www.right.com.cn/forum/f ... ead&tid=4108325](https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=4108325)

主要目的：破解ROOT密码、删除挖矿程序以彻底绝育、保留迅雷下载、保留局域网分享、支持自己添加脚本或程序

我参考过的教程：
https://www.right.com.cn/forum/thread-4034559-1-1.html
https://www.right.com.cn/forum/thread-3526512-1-1.html
https://www.right.com.cn/forum/thread-403227-1-1.html
感谢各位大佬。

主要流程：
官方2.12.2固件 → 安卓 → 坛友1.4.0固件 → 官方2.4.2固件 → U盘启动十六进制编辑EMMC（或者用dd，替换/etc/init.d/里面任意一个启动文件的扇区数据。注意是emmc的不是u盘的）
然后就可以用root登录ttl/ssh/telnet，做任何想做的事情了。
如果不需要root，那也不用编辑emmc。插U盘启动就是u盘的系统，拔掉u盘启动就是玩客云官方系统。
（也可以升级到2.12.2，但还要再等一次凌晨升级，我懒得等了反正2.4也能云添加）

超级繁琐耗时，熟悉操作的老哥们估计也得搞俩小时。

总之，主旨就是用歪门邪道修改emmc，如果你有emmc芯片编程器可以跳过繁琐的刷机过程直接改，一打开你就会发现EMMC内所有数据无任何加密。
![img](images/forum.php) （图片来自玩客云官网）
在任意一个开机自动运行的脚本文件里加上：
sleep 10  （等待10秒）
echo root:2021march16|chpasswd  （修改root密码为2021march16）
/etc/init.d/S50dropbear restart （开启ssh）


顺带一提，玩客云这东西坏得很。
1、你找客服绝育完它只是不读盘了，但还会用你emmc继续挖。耗你的网，占你的RAM，毁你的emmc。root完首要事项就是删挖矿程序和自动升级程序。
2、开机联网，1分钟之内玩客云就会改掉你的root密码。所以需要用root登录的时候要断外网重启。或者强行锁定密码文件 chattr +i /etc/passwd 和 chattr +i /etc/shadow



让我想不通的是，它为啥更新固件的时候直接抛弃原有系统分区，重新开一个新分区？8个G的内部储存里，有用的不到1个G
![img](images/forum.php)
![img](images/forum.php)
![img](images/forum.php)


顺便再贴一段控制LED指示灯颜色的代码：
开启红色：echo 1 > /sys/class/gpio/gpio2/value
关闭红色：echo 0 > /sys/class/gpio/gpio2/value
开启绿色：echo 1 > /sys/class/gpio/gpio3/value
关闭绿色：echo 0 > /sys/class/gpio/gpio3/value
开启蓝色：echo 1 > /sys/class/gpio/gpio4/value
关闭蓝色：echo 0 > /sys/class/gpio/gpio4/value
通过搭配红绿蓝能显示更多颜色