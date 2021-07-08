# **PVE在LVM文件系统下利用snapshot进行备份与还原**

https://koolshare.cn/thread-166080-1-1.html



回馈坛友第二贴，PVE宿主机系统备份与还原教程
如果你的PVE是默认安装的，那么就是采用LVM文件系统的，并且如果你没有改动过硬盘分区参数的话（尤其是minfree这个参数别改），系统下还有free空间，那么就可以利用LVM文件系统的snapshot特性进行PVE宿主机的系统备份与快照，还原。

如果改动过硬盘分区参数，请自行查询攻略，修改自己的PVE宿主机分区情况，确保pv下有足够的free空间，即未分配空间。
例如我的情况，PVE系统默认安装下，我有16g的Pfree（显示10.84是因为我开了快照备份，占用了4g空间）

1. root@pve:~# pvs
2.  PV      VG Fmt Attr PSize  PFree
3.  /dev/sda3 pve lvm2 a-- <223.07g <10.84g
4. 

*复制代码*

那么，你就可以使用snapshot进行系统备份。

snapshot，即快照，顾名思义，就是保存下所备份的分析那一时刻的文件，之后只要有文件改动，旧文件就会保留一份备份。

废话不多说，这些都可以自己去学学看看。
第一部分，创建快照
那么先来创建快照，~~**首先请务必关闭所有的小鸡**~~ 其实也不必，认识加深一点后应该知道了，创建的是root分区的快照，pve的小鸡文件不在这里
然后根据你的系统的可用free空间来开启snapshot分区

1. lvcreate -L 2G -n snap-pve -s /dev/pve/root
2. 

*复制代码*

上面的2G是快照分区的大小，snap-pve是快照的名称，/dev/pve/root是备份的分区，也就是我生成了一个最大能备份2G的root分区的快照。
执行

1. root@pve:~# lvdisplay /dev/pve/snap-pve
2.  --- Logical volume ---
3.  LV Path          /dev/pve/snap-pve
4.  LV Name          snap-pve
5.  VG Name          pve
6.  LV UUID          izt3m6-p2v3-KwGx-HXL2-3CCB-OIgT-uNiuQs
7.  LV Write Access     read/write
8.  LV Creation host, time pve, 2019-07-30 21:25:33 +0800
9.  LV snapshot status   active destination for root
10.  LV Status         available
11.  \# open           0
12.  LV Size          32.00 GiB
13.  Current LE        8192
14.  COW-table size      <4.16 GiB
15.  COW-table LE       1064
16.  Allocated to snapshot 81.81%
17.  Snapshot chunk size  4.00 KiB
18.  Segments          1
19.  Allocation        inherit
20.  Read ahead sectors   auto
21.  \- currently set to   256
22.  Block device       253:4
23. 

*复制代码*

可以看到，我的root分区是32g，当前快照分区大小4.16g，使用了81.81%，注意，当快照满了的时候，会有严重问题，因为没有空间用来备份已修改的文件，快照将会报废，使用请务必初始的时候给一个较合适的大小，或者及时关注并给快照增加空间。

这里提供两个办法，一个是手动的，

1. lvextend -L +2G /dev/pve/snap-pve

*复制代码*

上面的命令的意思是给snap-pve快照增加2g的空间

另外一个是系统自动的，会自动根据你的快照空间的占用率自动调节快照区的大小。
修改/etc/lvm/lvm.conf，将以下参数改为如下（第一个参数默认是100，第二个似乎默认是20），含义就是，当快照区占用超过80%后，系统自动扩容20%的空间。

1. snapshot_autoextend_threshold = 80
2. snapshot_autoextend_percent = 20
3. 

*复制代码*

第二部分，还原快照以及系统备份

还原快照很简单，就是让快照分区的内容和当前的被备份的分区内容合并即可，合并是需要卸载分区的
命令如下：

1. lvconvert --merge

*复制代码*


由于我们快照的是root分区，是无法卸载的，因此系统会提示你在下一次重启后会自动执行合并。执行重启，重启完后，系统回复到创建快照的那一瞬间的状态。


系统备份也很简单，快照分区其实也就是一个分区，将其挂载到某个空目录即可读取，再使用tar打包或者rsync远程传输到其他备份的地方即可。

另外就是删除快照，有时候觉得快照已经没用了，例如更新系统后用了一段时间发现没问题，此时可以删除快照，方法也非常简单：



1. lvremove /dev/pve/snap-pve

*复制代码*





教程结束