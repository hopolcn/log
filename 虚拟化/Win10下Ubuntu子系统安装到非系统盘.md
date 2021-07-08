# Win10下Ubuntu子系统安装到非系统盘

Windows下的Linux子系统对于工作在Linux系统下的人而言是十分方便的，但是用久了就会发现子系统对C盘的占用空间越来越大，原因在于Linux默认安装在用户目录下的 AppData\Local\Packages 下。

本文介绍在Windows下对Linux子系统迁移的方法。

## 工具

mklink : 本质上是一个创建链接的工具，这里使用mklink 欺骗系统，使系统误以为还是安装在了C盘

首先，我们需要找到子系统安装的文件系统在哪个位置，根据以往的经验，系统位置在：

C:\Users\xxxx\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows

注意 AppData 文件夹是隐藏文件夹，你需要打开查看隐藏文件的选项。

但实际上这个根据你自己安装的子系统需要自己另外确定，我上次安装的文件夹是上面哪个，第二次安装又变成了下面这个：

```
C:\Users\xxxx\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc
```

# 定位Linux子系统的文件系统位置

简单直接的方法：

首先安装一遍Linux子系统，在

```
C:\Users\xxxx\AppData\Local\Packages\
```

下查看带有类似 CanonicalGroupLimited.UbuntuonWindows 字眼的新文件夹，记下它的名字。

# 开始安装

卸载Linux子系统。
卸载的原因在于Linux子系统下的文件系统的权限更改十分复杂，这里面的一些文件不属于Windows下的管理员用户所有，也不属于你的用户，它就是Linux下用户所有的，使用一般的修改权限文件方法很容易出问题。因此还是推荐先备份后再卸载。

## 创造软链接

使用管理员打开cmd， 输入下面的命令：

```
mklink /j C:\Users\XXXX\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc  D:\Linux-share\WSL\
```

后面那个 D:\Linux-share\WSL\ 是非系统盘的位置。

创建成功后再打开应用商店，安装Linux子系统。