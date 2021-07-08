# 目录级别的冗余存储：SnapRAID

https://wzyboy.im/post/1186.html

*在数据存储领域，「备份」和「冗余」是两种常见的数据保护策略。两种策略各有不同的使用场景，对于重要数据，两者一起使用自然是最好了。本文介绍使用 [SnapRAID](https://www.snapraid.it/) 实现灵活的数据冗余存储。*



以下两篇文章与本文内容有联系，推荐阅读：



- [BorgBackup —— 增量备份方案](https://wzyboy.im/post/1106.html)
- [使用 mergerfs 合并多块硬盘的剩余空间](https://wzyboy.im/post/1148.html)



## 一、神奇的奇偶校验



小时候一直觉得 WinRAR 的「恢复卷」功能非常神奇。比如有一个 100 MiB 的压缩文件分成 10 个分卷，每个 10 MiB，创建者又创建了 3 个恢复卷，每个也是 10 MiB。当复制、分发这些卷的时候，如果因为数据传输、磁盘存储等各种原因，导致 10 个数据卷中有部分文件损坏或丢失，只要损坏或丢失的数据卷的数量小于等于恢复卷的数量，就可以用对应数量的恢复卷来修复压缩包。



小时候不理解为什么丢失一部分数据之后可以用现有的数据重新算出来，现在明白了，最简单的实现便是奇偶校验。



奇偶校验常见于数据传输中。比如 1 个字节（byte）由 8 个比特（bit）组成，但双方约定：只用 7 bit 来存储数据，剩下 1 bit 作为校验位（parity bit）。校验规则是：如果前面 7 bit 里 1 的数量为奇数（1, 3, 5, 7），则 parity bit 为 1；如果前面 7 bit 里 1 的数量为偶数（0, 2, 4, 6），则 parity bit 为 0。这样最终这个 byte 里 1 的数量一定是偶数，如果接收方发现 1 的数量不是偶数，那就说明出错了，一定是在传输过程中发生了比特翻转（bit flip），即本来是 0 的变成了 1，或本来是 1 的变成了 0。当然，如果这个 byte 在传输过程中发生了偶数个 bit flip，那校验倒也恰好能通过，但由于同一 byte 里有大于一个 bit flip 的概率非常低，所以奇偶校验在实际应用中还是非常简单有效的。



然而，上述奇偶校验只能知道「出错了」，但是无法知道「哪里出错了」，也无法修复出错的部分。但是在存储的时候，人们往往是知道哪里出错的。想像一下，如果上述 8 bit 不是存在同一 byte 里而是分散在 8 块磁盘上，这时候某一块磁盘突然挂了，你是明确地知道「哪里出错了」的（磁盘不转了），因此你根据其他 7 块磁盘里的 bit 值，来反推出坏掉的磁盘里存储的是 0 还是 1（把 1 的总数凑成偶数即可），也就是说，你可以利用 7 块健在的磁盘上的数据，修复坏掉的磁盘上的数据。



显然地，如果 8 块磁盘同时坏了 2 块或以上，那就有 00, 01, 10, 11 四种可能了，只有一个奇偶校验位的情况下是修复不了的。幸运的是，计算机科学家和数学家们早就研究出了其他更高级的[冗余算法](https://dl.acm.org/citation.cfm?doid=176979.176981)，可以使一组数据有两个或两个以上的校验位，用来在已知「哪里出错了」的情况下，修复出错的部分。SnapRAID 所用的[冗余算法](https://github.com/amadvance/snapraid/tree/master/raid)，用 *N* 个存储器用来存储数据，同时用 *P* 个存储器用来存放校验数据（*P ≤ 6, P ≤ N*），在总数 *P + N* 的存储器中，任意坏掉 *X* 个，只要 *X ≤ P*，就能用剩下存储器里的数据计算出坏掉的存储器里的数据。



这便是冗余存储的理论基础。



## 二、备份与冗余存储



虽说「备份」和「冗余」两大数据保护策略有各自的使用场景，冗余并不能取代备份，但如果要将这两者进行比较的话，冗余相比备份最大的优点，是可以相对节省存储空间。回顾上一节里提到的 7 + 1 块磁盘的场景，假设每块磁盘是 1 TB，则有效存储空间是 7 TB，剩下 1 TB 是校验数据。在磁盘使用过程中，任意一块磁盘坏掉，你都可以重新买一块新的替换磁盘，然后把坏掉磁盘的内容完整地推算出来，相当于你用 1 TB 的空间，换取了 7 TB 数据的「相对安全」（仅就对抗硬件故障而言），比完整备份 7 TB 数据省了很多空间。



然而，实际生产生活中，7 + 1 这样的组合风险比较高，因为[同时开始使用的磁盘，很可能一起坏掉](https://www.backblaze.com/b2/hard-drive-test-data.html)，一旦有两块磁盘同时坏掉，就会有数据无法被修复了。悲剧往往是这样发生的：一组磁盘里坏了一块，然后换了块新的上去，开始根据旧磁盘的数据推算出坏掉磁盘里原有的数据并写入新磁盘，由于这个推算过程需要将所有旧磁盘里的数据全部读一遍（视磁盘大小，这个过程可能会持续一两天），大量的读取操作成了最后一根稻草，还没等新磁盘填满，又一块旧磁盘挂了。



[根据存储及备份提供商 BackBlaze 公开的信息](https://www.backblaze.com/pod.html)，他们使用的是 13 + 2 的组合，即 15 块磁盘一组，最多能同时坏 2 块磁盘而不丢数据。



## 三、RAID 与 ZFS



在数据中心里，冗余存储最常见实现是 RAID。使用带冗余的 RAID 级别，如 RAID 5 和 RAID 6，可以在空间利用率和容错性之间达到一个平衡。存储服务器一般会安装硬件 RAID 卡，实现对操作系统透明的 RAID（操作系统看到的就是一个已经带冗余存储的磁盘，并不用关心下面是几块磁盘、怎么实现的）。在 GNU/Linux 中也可以使用 [mdadm](https://wiki.archlinux.org/index.php/Mdadm) 工具实现软件 RAID，无需专门的硬件即可实现冗余存储。



在《[使用 mergerfs 合并多块硬盘的剩余空间](https://wzyboy.im/post/1148.html)》一文的「读者来信」一节，有读者推荐了 ZFS。这套来自 Sun 的存储方案也可以实现类似 RAID 的功能，包括冗余存储。



然而，使用 RAID 和 ZFS 进行冗余存储都存在一个问题：data lock-in。即，要在已有数据的磁盘上使用 RAID / ZFS，需要把数据先导出，将磁盘清空，然后重新导入数据，并且你想要将某块磁盘脱离 RAID / ZFS 单独读写里面的数据也非常麻烦（如果不是不可能）。



这在数据中心看来也许不是什么问题，但是我希望能有一个更加灵活、自由的冗余存储方案。



## 四、似 RAID 而非 RAID 的 SnapRAID



SnapRAID 是一个目录级别的冗余存储方案，它与 RAID 的原理有相似的地方，但它并不是 RAID。SnapRAID 与 RAID 的主要区别有：



- SnapRAID 不会对数据进行条带化存储。RAID 通常会使用[数据条带化](https://en.wikipedia.org/wiki/Data_striping)，一个文件可能会被分散存储到多块磁盘上，这样的优点是读取的时候可以加速（多块磁盘同时读取），但条带化也是上节所说的 data lock-in 的根源——你不能拆出一块盘单独读写。
- SnapRAID 是工作于文件系统之上的。RAID 工作于文件系统之下，直接对磁盘区块进行操作，用磁盘区块上的比特计算校验数据，而 SnapRAID 是通过读取文件系统里的文件之后再进行计算的。
- SnapRAID 是非实时的。RAID 每时每刻都在工作，磁盘区块上的数据一旦发生变更就会重新计算校验数据，而 SnapRAID 可以在用户选择的时间进行重新计算。



SnapRAID 相比 RAID 的优点主要有：



- **数据独立**。不需要对磁盘做特殊处理，可以直接将已有数据的磁盘（甚至可以是不同文件系统的）加入 SnapRAID，SnapRAID 也不会改变这些已有的数据；一个文件不会被分散到多个磁盘，随时可以拆下来一块磁盘正常读写里面的数据；当磁盘阵列收到文件读写请求时，也只需要一块磁盘响应，而不是所有的磁盘全部从待机状态启动，开始寻道。
- **抗灾能力**。当磁盘列阵中同时损坏的磁盘数量超出预期而无法修复数据时，SnapRAID 的抗灾能力更强。例如：在 3 + 1 的 RAID 场景下，坏一块没事，如果同时坏了两块，所有的磁盘上的数据都将无法读取（因为条带化）；但如果是 3 + 1 的 SnapRAID，就算同时坏两块，剩下两块里的数据依然可以正常读取。
- **配置灵活**。标准的 RAID 等级中，RAID 5 最多承受 1 块磁盘同时损坏，RAID 6 最多承受 2 块磁盘同时损坏；而 SnapRAID 可以配置 1 到 6 块校验盘，最多承载 6 块磁盘同时损坏，因此可以组建更大的磁盘阵列而不提升风险（维持数据盘与校验盘的比例不变）。更重要的是，无论是增加还是减少磁盘，SnapRAID 都可以无痛完成，无需清空磁盘数据。
- 恢复误删文件。由于 RAID 是实时计算校验数据的，当文件被删除时，这一改动立刻就会被同步到校验数据里；而 SnapRAID 在用户请求的时候才进行同步，因此用户可以用 SnapRAID 从校验数据重新构建被误删除的文件。当然了，更可靠、更持久的的误删除防护还是应该用[增量备份](https://wzyboy.im/post/1106.html)来完成。
- 空间利用率高。在磁盘阵列中，校验盘的大小应大于等于数据盘中最大的那块。使用 SnapRAID 时，你可以「[超售](https://en.wikipedia.org/wiki/Resource_contention)」。比如数据盘是 6 TB 的但是只装了一半（3 TB），你把 4 TB 的磁盘作为校验盘也是可以的（因为此时校验数据最多只有 3 TB），只要在校验文件膨胀到接近 4 TB 的时候将校验文件挪到更大的磁盘里即可。同样的，校验盘里未被校验文件填满的剩余空间也可以用来存储一些「丢了也无所谓」的不重要数据。此外，由于 SnapRAID 工作于文件系统之上，你可以选择性地排除掉一些不想做冗余的目录和文件，以节省空间。



## 五、SnapRAID 的配置与使用



SnapRAID 提供了 Windows 版本的[二进制文件下载](https://www.snapraid.it/download)；GNU/Linux、macOS，以及各种 Unix-like 可以从源码编译或从软件仓库中安装。SnapRAID 的[配置文件](https://github.com/amadvance/snapraid/blob/master/snapraid.conf.example)简洁且注释详尽，读注释就能明白怎么配了。



目前我的 Gen8 里有三块 SATA 磁盘，容量分别是 2 TB, 4 TB, 4 TB。前两块服役多年，几乎满了，第三块是新买的，还是空的，我想把第三块磁盘作为校验盘。我的相关配置是这样的：



```
# 校验文件的位置
# 显然，校验文件不能放在数据盘上，否则就没有意义了
parity /media/disk3/snapraid.parity

# 如需添加更多的校验文件则继续添加
# 最多是 6 份校验，承受磁盘磁盘阵列中最多同时坏掉 6 块盘的情况
#2-parity /media/disk4/snapraid.2-parity
#3-parity /media/disk5/snapraid.3-parity

# 重要的索引文件，建议保存多份（内容是一样的）
# 我在系统盘（SSD）上存了一份，然后在三块磁盘上都各存一份
# 系统盘上的这份同时又会被 BorgBackup 异地备份
content /home/snapraid/snapraid.content
content /media/disk1/snapraid.content
content /media/disk2/snapraid.content
content /media/disk3/snapraid.content

# 指定数据盘及其挂载点
# 这里不一定要写确切的挂载点，可以是这块盘上的任意目录
# 目录以外的内容会被完全忽略
data d1 /media/disk1/
data d2 /media/disk2/

# 忽略所有隐藏文件和目录（不做冗余）
# 在 Unix-like 里就是 . 开头的文件和目录
# 在 Windows 里就是带隐藏属性的文件和目录
nohidden

# 排除列表与包含列表，注意顺序！下文详解
exclude *.unrecoverable
exclude *.nobackup
exclude *.nobackup/
exclude /tmp/
exclude /lost+found/
#include /foo
#include /bar/

# 生成校验数据时，每处理 10 GiB 数据自动保存一次，方便断点继续
autosave 10
```

写好配置文件之后，使用 `snapraid sync` 进行首次同步，也就是根据数据盘的内容生成校验盘的内容。我的第一次同步花了 24 小时：



```
Scanning disk d1...
Scanning disk d2...
Using 221 MiB of memory for the FileSystem.
Initializing...
Resizing...
Saving state to /home/snapraid/snapraid.content...
Saving state to /media/disk1/snapraid.content...
Saving state to /media/disk2/snapraid.content...
Saving state to /media/disk3/snapraid.content...
Verifying /home/snapraid/snapraid.content...
Verifying /media/disk1/snapraid.content...
Verifying /media/disk2/snapraid.content...
Verifying /media/disk3/snapraid.content...
Syncing...
Using 24 MiB of memory for 32 blocks of IO cache.
0%, 959 MB, 40 MB/s, CPU 1%, 24:08 ETA
```

常用的 SnapRAID 命令：



- `snapraid sync`：根据数据盘生成校验盘；
- `snapraid diff`：查看有哪些数据需要 sync；
- `snapraid status`：查看磁盘阵列的状态；
- `snapraid scrub`：进行[数据擦洗](https://en.wikipedia.org/wiki/Data_scrubbing)，提早发现磁盘阵列中的错误。



SnapRAID 首次同步完成之后，可以将 `snapraid sync` 和 `snapraid scrub` 加入 cron / systemd timer，定时运行。后者默认配置下每次运行擦洗全部数据的 8%，并且会挑选最近 10 天内没有被擦洗过的数据进行擦洗。如果每天运行一次 `snapraid scrub` 的话，每 12.5 天所有数据都会被擦洗一遍，形成一个健康的循环。



当擦洗发现有数据损坏，或是更糟糕地，某天整块磁盘挂了（不转了），就需要修复数据了。这时候应该做的是停掉所有的定时任务，然后换上新的磁盘，然后用 `snapraid fix -d name_of_disk` 命令，根据健在磁盘的内容，在新磁盘里重建坏掉磁盘里的内容。只要坏掉的磁盘数量小于等于校验盘的数量，SnapRAID 都能完整地修复数据。



由于 `snapraid sync` 是定期执行的，这意味着在下次同步之前，磁盘阵列是有机会恢复到上次同步的状态的，因此 `snapraid fix` 除了可以重建整个磁盘，也可以重建单个文件，也就是反删除。如果你误删除了文件，可以用 `snapraid fix -f path/to/file` 来恢复文件到上次同步时的状态。



## 六、SnapRAID 最佳实践



事实上我也就昨天才开始用 SnapRAID，所以这所谓的「最佳实践」，其实也只是我在阅读文档和配置使用中觉得需要注意的地方。



### 排除列表与包含列表



因为 SnapRAID 是工作在文件系统之上、基于目录的冗余存储方案，因此可以很方便选择哪些目录和文件需要做冗余，哪些不需要。在配置文件中 `include` 和 `exclude` 的规则如下：



- 可以使用 `* ? [1-3]` 这样的简单通配符；
- 以 `/` 开头的路径匹配的是数据盘的根目录，而不是系统的根目录；
- 以 `/` 结尾的路径只会匹配目录；
- 不以 `/` 结尾的路径只会匹配文件；
- 如果最后一条规则是**包含**（`include`），则所有未匹配的路径都会被**排除**；
- 如果最后一条规则是**排除**（`exclude`），则所有未匹配的路径都会被**包含**。



### 适合使用 SnapRAID 的文件



因为 SnapRAID 是定期运行的，在两次 `snapraid sync` 之间新增的数据是有一段时间没有冗余的，这时候如果磁盘挂了，那这些数据就丢失了。因此，**SnapRAID 并不适合用来对频繁变动的文件（如：系统盘）做冗余**。



SnapRAID 比较适用的场景是体积巨大、但是很少更改的文件。比如对摄影爱好者来说，磁盘中可能会有好几个 TiB 的 RAW 照片或是未剪辑的 4K 视频文件。这些原始文件因为体积巨大，很难通过[互联网](https://what-if.xkcd.com/31/)做异地备份，而它们本身几乎不会再发生变化，因此非常适合用 SnapRAID 做冗余。由于 SnapRAID 的灵活配置，用户可以方便地选择对哪些文件做冗余，也可以随时将单个磁盘从阵列中临时脱离出来，直接插到图形工作站上进行高速读写。



### 与 mergerfs 配合



SnapRAID 提供了类似 RAID 的冗余功能，但是 RAID 还能将磁盘阵列里的磁盘合并成一个大磁盘。SnapRAID 本身并不提供合并磁盘的功能，但是 mergerfs 可以达成这个目的：《[使用 mergerfs 合并多块硬盘的剩余空间](https://wzyboy.im/post/1148.html)》。



## 七、致谢



感谢 [Jimmy Xu](https://jimmyxu.org/) 在本文发表前，对本文奇偶校验算法相关的内容进行技术评测，并提出了宝贵的修改意见。