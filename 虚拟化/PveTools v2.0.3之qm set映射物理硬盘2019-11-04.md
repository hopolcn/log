# PveTools v2.0.3之qm set映射物理硬盘2019-11-04

# 一、工具准备

链接：[https://github.com/ivanhao/pvetools](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fivanhao%2Fpvetools)

# 二、安装

按github链接的说明进行安装，如果安装有问题，请再看一遍说明，并查看wiki中的FAQ。

# 三、使用

1. 按说明安装好并运行后，主界面：

![img](https:////upload-images.jianshu.io/upload_images/4171480-e6f615cef5c243ec.png?imageMogr2/auto-orient/strip|imageView2/2/w/766/format/webp)

main

由此进入

![img](https:////upload-images.jianshu.io/upload_images/4171480-6d7928f425b9a402.png?imageMogr2/auto-orient/strip|imageView2/2/w/736/format/webp)

qm

选d

![img](https:////upload-images.jianshu.io/upload_images/4171480-ad01e6627aece6dd.png?imageMogr2/auto-orient/strip|imageView2/2/w/750/format/webp)

choose

### 1. 添加

![img](https:////upload-images.jianshu.io/upload_images/4171480-d80dabcfe265447d.png?imageMogr2/auto-orient/strip|imageView2/2/w/742/format/webp)

vms

选择一个需要添加的vm

![img](https:////upload-images.jianshu.io/upload_images/4171480-2814139faac335c2.png?imageMogr2/auto-orient/strip|imageView2/2/w/765/format/webp)

choose

会列出目前已添加的硬盘设备，在选择列表中选择需要添加的设备，标记`*`号

![img](https:////upload-images.jianshu.io/upload_images/4171480-38b47da9d139c7f7.png?imageMogr2/auto-orient/strip|imageView2/2/w/783/format/webp)

type

选择设备类型，sata scsi 或ide，注意，ide只能加到4个设备，也就是序号ide3。有些os，比如windows，会需要添加类型为ide

![img](https:////upload-images.jianshu.io/upload_images/4171480-8be0fbaf5fa4ff1c.png?imageMogr2/auto-orient/strip|imageView2/2/w/786/format/webp)

done

提示完成。

### 2. 删除

还是以刚才的vm为例子

![img](https:////upload-images.jianshu.io/upload_images/4171480-73732330cfc35e58.png?imageMogr2/auto-orient/strip|imageView2/2/w/786/format/webp)

choose

![img](https:////upload-images.jianshu.io/upload_images/4171480-e0642611b3e4c6a3.png?imageMogr2/auto-orient/strip|imageView2/2/w/795/format/webp)

vms

![img](https:////upload-images.jianshu.io/upload_images/4171480-f0f2c1e1873940f3.png?imageMogr2/auto-orient/strip|imageView2/2/w/824/format/webp)

disk

这里显示刚才添加的硬盘已经成功了，下面演示删除

![img](https:////upload-images.jianshu.io/upload_images/4171480-fd4bb42059c14008.png?imageMogr2/auto-orient/strip|imageView2/2/w/800/format/webp)

done

添加和删除后，都可以在web界面中看到变化。

如果好用请去github页面点星点赞，



作者：龙天ivan
链接：https://www.jianshu.com/p/c44770dcfd20
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。