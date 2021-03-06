# SnapRAID简介

https://zhuanlan.zhihu.com/p/32058088

最近在看SnapRAID，发现有点意思。

先说下优点：





- 支持1-6个盘的冗余级别，校验盘越多，可以承受损坏的硬盘越多。
- 支持完整性巡检，可以修复硬盘上静默错误（silent
       corruption）。
- 如果故障盘太多而无法恢复，则只会丢失故障盘上的数据，不影响其他硬盘中的数据。
- 可以在一定程度上恢复误删除的文件。
- 可以直接添加已有的数据盘，无需格式化，也不会丢失已有数据。
- 硬盘可以不同大小。
- 可以添加文件夹，而不一定是整块硬盘。
- 可以随时添加硬盘。
- 不会更改你的数据，可以随时停止使用SnapRAID，无需重新格式化或移动数据。
- 支持单个硬盘休眠。访问文件时，只需要唤醒文件所在的磁盘。（当然更新冗余数据时需要唤醒所有硬盘）
- 支持windows和linux



再来看看缺点：





- 快照式冗余，而非实时，需要手动更新冗余数据。（当然也可以写批处理，定时运行）
- 需要额外提供校验盘（当然，几乎所有的raid都需要），最好是空盘。（也可以不空）
- 各个盘的文件系统独立，管理起来稍繁琐。
- 不分条数据。而使用传统RAID，可以通过分条来增加读写速度。
- 不支持实时恢复。而使用传统RAID，当硬盘发生故障时，不必停止工作。
- 只能恢复有限数量的硬盘。多一个校验盘则增加一个冗余额度。
- 只保存文件，时间戳，符号链接和硬链接。权限，所有权和扩展属性不会保存。
- 最新版只有命令行，没有官方GUI界面。有第三方GUI界面Elucidate，更新速度较慢。





看起来，就像官方说明一样，比较适合存有大量大文件并很少改动的家庭媒体中心，比如说蓝光高清爱好者等。

具体使用方法和限制，请查看本人的其他SnapRAID文章。

还有两篇在路上，一篇是FAQ，一篇是配置说明。