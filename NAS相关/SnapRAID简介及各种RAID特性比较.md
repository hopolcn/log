# SnapRAID简介及各种RAID特性比较

http://www.songming.me/snapraid-compare.html



## SnapRAID简介

SnapRAID介绍：是磁盘整列的备份程序，可存储磁盘的奇偶校验信息，在两个磁盘损坏时也能恢复数据。

SnapRAID定位：家庭媒体中心，特别适合于文件较大较多且较少改变的系统。

SnapRAID特点：

1. 所有数据都有完整性校验，避免数据悄然损坏； 

2. 如果磁盘损坏的过多影响恢复，那么你损坏的也不是全部数据，未损害硬盘的数据不受影响，可以单独读取； 

3. 如果你不小心删掉了一些数据，你仍然可以恢复它们； 

4. 对于已有数据的磁盘，你无需格式化硬盘便可以加入整列，磁盘中的已有数据不受影响； 

5. 整列的磁盘可以容量不同； 

6. 随时添加磁盘； 

7. 每个磁盘数据相互独立，每个磁盘读写也独立，也就是说可以单盘读写，其余磁盘休眠，节能环保延长磁盘寿命； 

8. 基于上一点特性，你不会因为SnapRAID而提高整个整列的读写性能； 

9. 它不会锁定数据，可以随时停止使用SnapRAID而不需要移除数据或者格式化硬盘，各个磁盘中的数据不受影响，可以单独读写。 

   

   

## 各种RAID特性比较

除标准RAID磁盘部署解决方案外，还有众多解决方案。根据奇偶校验的实时性可以把各种冗余分为两大类：

1. 一类是实时（realtime）奇偶校验的冗余方案，这类方案的冗余不需要人为干预，实时更新，像RAID； 
2. 一类是快照（snapshot）奇偶校验的冗余方案，这类方案的冗余是在接收到人为指令后更新，像Backup。 

主要解决方案有：

1. [unRAID](http://www.lime-technology.com/)-商业和开源GPL2的解决方案，修改版可实现Linux下ReiserFS文件系统实时冗余，不支持任何完整性校验。 
2. [FlexRAID](http://www.flexraid.com/)-Windows下商业和专有C ++ / Java应用，可有限支持linux。它同时支持快照冗余和实施冗余，支持完整性校验。 
3. [disParity](http://www.vilett.com/disParity/forum/)-Windows下专有的. NET应用，支持快照冗余和完整性校验。 
4. [ZFS](http://en.wikipedia.org/wiki/ZFS)-开源文件系统（但与GPL不兼容），支持实时冗余和完整性校验。 
5. [Btrfs](http://en.wikipedia.org/wiki/btrfs)-开源GPL2授权的文件系统，支持实时冗余，Linux内核3.9以上开始支持RAID 5 / 6冗余和完整性校验。 
6. [Storage Spaces](http://blogs.msdn.com/b/b8/archive/2012/01/05/virtualizing-storage-for-scale-resiliency-and-efficiency.aspx)-最后是来自微软的方案，改方案已经集成到Win8了，支持专有的实时冗余，不支持完整性校验，但是在[ReFS](http://blogs.msdn.com/b/b8/archive/2012/01/16/building-the-next-generation-file-system-for-windows-refs.aspx)文件系统开始提供一些有限的支持。 

这些方案各有优缺点，综合各特性针对于家庭媒体中心的解决方案SnapRAID应运而生：

|                                                      | **SnapRAID**                                                 | **unRAID**                            | **FlexRAID**                     | **disParity**                | **ZFS**                                                      | **Btrfs**                         | **Storage Spaces**                |
| ---------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------- | -------------------------------- | ---------------------------- | ------------------------------------------------------------ | --------------------------------- | --------------------------------- |
| **冗余模式**                                         | 快照                                                         | 实时                                  | 快照和实时                       | 快照                         | 实时                                                         | 实时                              | 实时                              |
| **完整性校验****默认校验方式**                       | 支持SpookyHash             128 bit                           | 不支持                                | 支持Adler32             32 bit   | 支持MD5128 bit               | 支持fletcher4             256 bit                            | 支持CRC32C             32 bit     | 不支持                            |
| **如果发生silent errors可否通过校验盘计算修复**      | 支持                                                         | 不支持[1]                             | 不支持[2]                        | 不支持                       | 支持                                                         | 支持                              | 不支持                            |
| **允许磁盘故障数**                                   | 1 2 3 4 5 6                                                  | 1                                     | 1 2 3 4 5 6+                     | 1                            | 1 2 3                                                        | 1 2                               | 1                                 |
| **如果磁盘损坏块数超过冗余数可否恢复未损坏磁盘数据** | 支持                                                         | 支持                                  | 支持                             | 支持                         | 不支持                                                       | 不支持                            | 不支持                            |
| **读取单个文件时有几块硬盘运行**                     | 1                                                            | 1                                     | 1                                | 1                            | 全部                                                         | 全部                              | 全部                              |
| **可否添加已有数据的磁盘而不影响磁盘中的数据**       | 支持                                                         | 不支持                                | 支持                             | 支持                         | 不支持                                                       | 不支持                            | 不支持                            |
| **支持的操作系统**                                   | Linux             Windows              Mac OS X              OpenIndiana              Solaris              BSD | Linux                                 | Windows             Linux        | Windows                      | Mac OS X             OpenIndiana              Solaris              BSD | Linux                             | Windows                           |
| **首次正式支持RAID5冗余及更高冗余的时间**            | 2011                                                         | 2005                                  | 2008                             | 2009                         | 2006                                                         | 2013                              | 2012                              |
| **授权和价格**                                       | Open Source GPL3             Free                            | Open Source GPL2             69$/119$ | Proprietary             40$/100$ | Proprietary             Free | Open Source CDDL             Free                            | Open Source GPL2             Free | Proprietary             Windows 8 |
| **交互方式**                                         | 命令行或             [Elucidate](http://elucidate.codeplex.com/) | 图形                                  | 图形                             | 命令行                       | 命令行                                                       | 命令行                            | 图形                              |

[1]-unRAID doesn’t have any kind of checksum, and it just ignores silent errors. Even worse, if a parity error is detected as result of a silent error in the data, the parity is automatically recomputed, making impossible to recover the silent error, even manually.

[2]-Flexraid uses checksums to validate files, but such checksums are not verified when data is read to update the parity. This means that any silent error present will propagate into the parity, making impossible to fix it later, even if it can be still detected comparing the file checksum.

没有最好的方案，只有最适合的方案，SnapRAID定位于家庭多媒体中心个人认为还是有它独到之处，虽然采取快照冗余模式，但是可以设置快照的计划任务，实现定期自动快照，间隔时间根据自己的需求而定；再者其数据迁移的平滑性和硬盘独立性也是颇具特点。本文主要参考了[SnapRAID](http://snapraid.sourceforge.net/index.html)官网的介绍，详情请访问官网查阅，本文受制于个人水平难免有不确之处。