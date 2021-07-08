# ESXi 直通板载SATA控制器

首先，在直通SATA控制器之前，请确保板载SATA接口已经连接硬盘。如果没有连接硬盘，ESXi会彻底忽略掉这个设备（也就是在web client下的主机-管理-硬件-pci设备看不到）。另外由于要直通SATA控制器，所以ESXi的系统盘不能接在板载SATA控制器上，否则ESXI的存储将无法正常使用。具体步骤如下：

1、shell下执行如下命令

```
lspci -v | grep "Class 0106" -B 1
```

查看是否有如下显示

```
0000:00:1f.2 SATAcontroller Mass storage controller: Intel Corporation Lynx Point AHCIController [vmhba0]

         Class 0106: 8086:8c02
```

2、手工配置直通。

```
vi /etc/vmware/passthru.map
```

添加如下：

```
#Intel Corporation Lynx Point AHCI Controller

8086   8c02    d3d0    fasle
```

3、保存，重启即可。

注意：不同的芯片组可能会有不同。***8c02\*** 更改为第一步在终端看到的硬件ID即可。