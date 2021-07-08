# ESXI使用系统U盘做存储

1.前言
本人虚拟机比较偏爱esxi，但esxi通常需要一个数据存储器，client manager上只支持hdd。据说玩黑群直通（非RDM）比较合适，本着省钱省功耗的原则，直通板载achi给黑裙，esxi存储使用系统u盘空闲的空间。这样省了一张hba卡，省了一个盘位，降了功耗。
2.适用场景
本文档适用于
a.必须esxi6及以上版本
b.esxi系统u盘（tf）建议8G以上
b.熟悉esxi ssh人士。
3.操作步骤
a.打开esxi ssh并root登录
b.进入/vmfs/devices/disks目录。shell:cd /vmfs/devices/disks
 c.列出磁盘 shell:ls
   6.0通常是mpx.vmhba32:C0:T0:L0，但6.5不不同，可能是以naa开始。通常规律是有一个前缀想同，后面带有：1,5,6,7,8类似的:数字很可能就是
   esxi的系统盘（不带":数字"的那个）。本文以mpx.vmhba32:C0:T0:L0为例
   还可通过partedUtil getptbl mpx.vmhba32:C0:T0:L0 查看分区信息以确认设备
  d.查看分区
    shell:partedUtil getptbl mpx.vmhba32:C0:T0:L0
    显示输出：
    gpt
  2088 255 63 33554432
  1 64 8191 C12A7328F81F11D2BA4B00A0C93EC93B systemPartition 128
  5 8224 520191 EBD0A0A2B9E5443387C068B6B72699C7 linuxNative 0
  6 520224 1032191 EBD0A0A2B9E5443387C068B6B72699C7 linuxNative 0
  7 1032224 1257471 9D27538040AD11DBBF97000C2911D1B8 vmkDiagnostic 0
  8 1257504 1843199 EBD0A0A2B9E5443387C068B6B72699C7 linuxNative 0
  9 1843200 7086079 9D27538040AD11DBBF97000C2911D1B8 vmkDiagnostic 0
c.获取上文中红色部分值（红色部分每个人是不同的），将其-34 (减34) 替换如下shell中的红色的数字（复制出shell部分到写字板，然后将上面的数字减去34以后的值，复制到esxi shell中执行）
    为什么是34，我也不清楚，我是试出来的最小值，还有个-48出现的也比较频繁。网上大都说是-2048.这都没问题。
partedUtil setptbl mpx.vmhba32:C0:T0:L0 gpt \
"1 64 8191 C12A7328F81F11D2BA4B00A0C93EC93B 128" \
"5 8224 520191 EBD0A0A2B9E5443387C068B6B72699C7 0" \
"6 520224 1032191 EBD0A0A2B9E5443387C068B6B72699C7 0" \
"7 1032224 1257471 9D27538040AD11DBBF97000C2911D1B8 0" \
"8 1257504 1843199 EBD0A0A2B9E5443387C068B6B72699C7 0" \
"9 1843200 7086079 9D27538040AD11DBBF97000C2911D1B8 0" \
"2 7086080 15472639 EBD0A0A2B9E5443387C068B6B72699C7 0" \
"3 15472640 33554398 AA31E02A400F11DB9590000C2911D1B8 0"
d.创建存储：
shell:vmkfstools -C vmfs5 -b 1m -S UsbDatastore mpx.vmhba32:C0:T0:L0:3
完成后即可在client里看到一个UsbDatastore的存储了，可在上面建虚拟机。
4.总结
    u盘由于速度原因，建议不要其建立大的虚拟机文件，否则经常会导致存储丢失（也可能是我u盘有点问题）。比合适的是做黑裙虚拟机，通过iso启动，当然像ros，openwrt这类小的系统也是没问题的，总之就是减少u盘操作。