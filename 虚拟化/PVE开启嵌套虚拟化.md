# PVE开启嵌套虚拟化

所谓嵌套虚拟化，就是说你可以在PVE的虚拟机中再开启虚拟机，也就是平时俗话说的套娃。

## 检测pve虚拟系统是否支持虚拟化

PVE虚拟出来的vm系统的cpu,默认不支持vmx，即不支持嵌套虚拟化，在**虚拟机**中使用命令来验证：

```bash
 egrep --color 'vmx|svm' /proc/cpuinfo
没有输出即不支持，否则会高亮显示vmx或者svm。
```

## 开启嵌套虚拟化步骤：

#### 1.开启pve主机的nested,关闭所有虚拟机

检查pve系统是否开启nested，运行命令：

```bash
 cat /sys/module/kvm_intel/parameters/nested
Y
输出N，表示未开启，输出Y，表示已开启
```

检查结果未开启，必须关闭所有的虚拟机系统，否则不能开启内核支持。

```bash
modprobe -r kvm_intel
modprobe kvm_intel nested=1
 # 再次检查nested,输出Y，即为开启成功。
cat /sys/module/kvm_intel/parameters/nested 
```

如果报错Module kvm_intel is in use，请检查你的虚拟机是否全部关闭。

#### 2.设置系统启动后自动开启nested

```bash
echo "options kvm_intel nested=1" >> /etc/modprobe.d/modprobe.conf
```

这样系统重启会自动加载netsted，支持嵌套虚拟了。

#### 3.设置虚拟系统vm的cpu类型为host

```bash
qm set <vmid> --cpu cputype=host
```

也可以在图形界面设置：选择vm,“硬件”–“处理器”–“类型”–“host"