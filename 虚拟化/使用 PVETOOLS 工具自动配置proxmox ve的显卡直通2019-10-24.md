# 使用 PVETOOLS 工具自动配置proxmox ve的显卡直通2019-10-24

# 一、工具准备

链接：[https://github.com/ivanhao/pvetools](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fivanhao%2Fpvetools)

# 二、安装

按github链接的说明进行安装，如果安装有问题，请再看一遍说明，并查看wiki中的FAQ。

# 三、使用

1. 按说明安装好并运行后，主界面：

![img](https:////upload-images.jianshu.io/upload_images/4171480-ecf44e6cd2a294e4.png?imageMogr2/auto-orient/strip|imageView2/2/w/707/format/webp)

main

1. 先择`i`项“配置PCI硬件直通

![img](https:////upload-images.jianshu.io/upload_images/4171480-911ee036b80acfcf.png?imageMogr2/auto-orient/strip|imageView2/2/w/709/format/webp)

pci passthrough

这里要说明一下，`a`和`b`是一对，一个开启一个关闭，开启物理机硬件直通支持是大前提，如果这个不开启，直接使用`c`的功能，是不管用的，而且在`c`里，也会提示你来`a`先配置。这里只需要给物理主机配置一次就可以了，如果误点会提示已经配置过了。

1. 选`a`进入配置

![img](https:////upload-images.jianshu.io/upload_images/4171480-34fc24f98aa6ebfc.png?imageMogr2/auto-orient/strip|imageView2/2/w/680/format/webp)

enable

这里肯定是yes了，除非你点错了

![img](https:////upload-images.jianshu.io/upload_images/4171480-4280a1103674eb0e.png?imageMogr2/auto-orient/strip|imageView2/2/w/752/format/webp)

configed

如果配置过了，会有提示

1. 安装好后提示重启，这里重启物理机是必要的

![img](https:////upload-images.jianshu.io/upload_images/4171480-d06462a3cf5f2401.png?imageMogr2/auto-orient/strip|imageView2/2/w/777/format/webp)

done

------

## 到这里开启硬件直通的配置就完成了，以下是显卡直通的说明。

> 需要配置什么硬件直通给虚拟机，可以通过pve的web界面进行配置，添加pci就可以了
>
> ![img](https:////upload-images.jianshu.io/upload_images/4171480-f4a48bb54979afe8.png?imageMogr2/auto-orient/strip|imageView2/2/w/324/format/webp)
>
> pci
>
> 例如你的硬盘直通（sata控制器）啊，什么的

------

1. 重启后，还按上面步骤回到这界面，这回给现有的虚拟机进行配置直通（注意，是现有的虚拟机，工具不会帮你自动创建，它是个傻子）

![img](https:////upload-images.jianshu.io/upload_images/4171480-0228e7c9070a5ee8.png?imageMogr2/auto-orient/strip|imageView2/2/w/736/format/webp)

reconfig

6.这里要看一下

![img](https:////upload-images.jianshu.io/upload_images/4171480-2d84c01699b07ef9.png?imageMogr2/auto-orient/strip|imageView2/2/w/823/format/webp)

video card

这里`a`是配置显卡直通的前提，会加上一些必要的参数，并且需要重启，这里只需要第一次配置好以后，就不需要每次都配置了，如果你误点了，也会提示你已经配置过了。

1. 这里工具会帮你过滤出来显卡设备，不需要从一堆里面自己找了

![img](https:////upload-images.jianshu.io/upload_images/4171480-2f0267f47f39131c.png?imageMogr2/auto-orient/strip|imageView2/2/w/909/format/webp)

config



标好星号后点OK下一步

1. 按提示确认后，提示安装成功

![img](https:////upload-images.jianshu.io/upload_images/4171480-1fe8d119f2d9c59c.png?imageMogr2/auto-orient/strip|imageView2/2/w/773/format/webp)

success

这一步，也是配置一次就可以了。

![img](https:////upload-images.jianshu.io/upload_images/4171480-48c6f0489e010df5.png?imageMogr2/auto-orient/strip|imageView2/2/w/656/format/webp)

comfirm

1. 显卡直通最后一步：

![img](https:////upload-images.jianshu.io/upload_images/4171480-c8915b3edd64b79d.png?imageMogr2/auto-orient/strip|imageView2/2/w/727/format/webp)

video card1

选`b`

![img](https:////upload-images.jianshu.io/upload_images/4171480-cf29dbe1077d7946.png?imageMogr2/auto-orient/strip|imageView2/2/w/893/format/webp)

choose card

勾上你要直通的显卡，支持多选

![img](https:////upload-images.jianshu.io/upload_images/4171480-d5e2f78480094e99.png?imageMogr2/auto-orient/strip|imageView2/2/w/736/format/webp)

vms

这里上下左右键，tab键，用你知道的方式把要直通的虚拟机前面标上星号，已经配置过的会自动标上星号，需要调整就按自己的需求，标好后OK会自动配置或切换直通。

![img](https:////upload-images.jianshu.io/upload_images/4171480-aade021c41b65a46.png?imageMogr2/auto-orient/strip|imageView2/2/w/757/format/webp)

comfirm

确认就是了

![img](https:////upload-images.jianshu.io/upload_images/4171480-fd3661eb3fd5d7f0.png?imageMogr2/auto-orient/strip|imageView2/2/w/761/format/webp)

options

这里会有选项，如果不是gpu核显直通，直接默认就可以，按ok下一步

![img](https:////upload-images.jianshu.io/upload_images/4171480-ed1146461f0bb49e.png?imageMogr2/auto-orient/strip|imageView2/2/w/757/format/webp)

success

这就成了。

![img](https:////upload-images.jianshu.io/upload_images/4171480-0b79a2875a7766a3.png?imageMogr2/auto-orient/strip|imageView2/2/w/776/format/webp)

check

等下，还有个功能，如果检测到你之前有配置直通给其他虚拟机了，会提示你要不要自动帮你切换，否则你需要手工重启虚拟机。有自动的功能，为什么不用？？？

> 这个逻辑是这样的，你第一台直通设备，Usb键盘鼠标是要手工添加的，只有切换的时候，才会自动过去）

![img](https:////upload-images.jianshu.io/upload_images/4171480-896ec1dc825f8ff5.png?imageMogr2/auto-orient/strip|imageView2/2/w/827/format/webp)

check

还有提示？是的，你需要把你直通的虚拟机里的键盘鼠标带过去不啊？还是你不怕麻烦要自己搞啊？有自动的功能为什么不用？？？

![img](https:////upload-images.jianshu.io/upload_images/4171480-e1590c98f0ad79a2.png?imageMogr2/auto-orient/strip|imageView2/2/w/791/format/webp)

almost end



![img](https:////upload-images.jianshu.io/upload_images/4171480-5e5bc503e2711cc4.png?imageMogr2/auto-orient/strip|imageView2/2/w/782/format/webp)

end

好了，看看你的显示器。

如果好用请去github页面点星点赞，



作者：龙天ivan
链接：https://www.jianshu.com/p/871fcc955f3b
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

