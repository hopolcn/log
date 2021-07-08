# OpenWrt 编译步骤与命令详解

## [![img](images/2b1c3d1bf034562364aa5ce128d9b7e2.png)](https://wp.qiniu.gxnas.com/wp-content/uploads/2019/12/2b1c3d1bf034562364aa5ce128d9b7e2.png)

## 前言

编译 Open­Wrt 的过程就像是复读机，除了选择系统组件外，几乎每次编译都是复制粘贴相同的命令。而理解每一条命令的作用、什么时候该去执行，这样才能更好的去解决编译中遇到的问题，更顺利的编译出固件。

## 首次编译

- 克隆 Open­Wrt 源码

  ```none
  git clone https://github.com/coolsnowwolf/lede openwrt
  ```

  > 这里以 Lean 大佬的源码仓库为例子，毕竟很多人都在用它。命令末尾加了`openwrt`是指克隆代码到`openwrt`目录，目的是为了规范化，因为有时并不是编译这个的源码。

- 进入源码目录

  ```none
  cd openwrt
  ```

- 拉取 feeds 源码

  ```none
  ./scripts/feeds update -a
  ```

  > feeds 是扩展的软件包，独立于 Open­Wrt 源码之外，所以需要单独进行拉取和更新。

- 安装 feeds 中的软件包

  ```none
  ./scripts/feeds install -a
  ```

- 调整 Open­Wrt 系统组件

  ```none
  make menuconfig
  ```

  > 首次编译建议只选择架构，其它都不要动，这样编译成功率会更高。如果不打算调整组件则输入`make defconfig`，它会检测编译环境并生成默认的编译配置文件。

- 预下载编译所需的软件包

  ```none
  make download -j8 V=s
  ```

  > `-j8`是指使用8个线程下载，理论上是数字越大下载越快，但似乎有个上限，实测5线程以上其实速度相差不了多少，在网络好的情况下，基本在5分钟以内能下载完。

- 检查文件完整性

  ```none
  find dl -size -1024c -exec ls -l {} \;
  ```

  > 此命令可以列出下载不完整的文件（根据我多次编译的经验得出小于1k的文件属于下载不完整），如果存在这样的文件可以使用`find dl -size -1024c -exec rm -f {} \;`命令将它们删除，然后重新执行`make download`下载并反复检查，确认所有文件完整可大大提高编译成功率，避免浪费时间。

- 开始编译

  ```none
  make -j1 V=s
  ```

  > `-j1`：使用单线程编译。新手推荐单线程编译，一是因为玄学问题可能成功率高，二是方便查看错误日志，多线程的错误日志是交织在一起的，不方便排错。`V=s`：输出详细日志，用于编译失败时找出错误。

## 再次编译

- 进入源码目录（如果不在此目录）

  ```none
  cd openwrt
  ```

### 更新

> **TIPS：** 短期内再次编译可忽略更新这个步骤。

- 更新系统软件包

  ```none
  sudo sh -c "apt update && apt upgrade -y"
  ```

  > 主要作用是更新在编译环境搭建时所安装的编译组件

- 拉取 Open­Wrt 源码更新

  ```none
  git pull
  ```

- 拉取 feeds 源码

  ```none
  ./scripts/feeds update -a
  ```

- 安装 feeds 中的软件包

  ```none
  ./scripts/feeds install -a
  ```

### 文件清理

- 清除旧的编译产物（可选）

  ```none
  make clean
  ```

  > 在源码有大规模更新或者内核更新后执行，以保证编译质量。此操作会删除`/bin`和`/build_dir`目录中的文件。

- 清除旧的编译产物、交叉编译工具及工具链等目录（可选）

  ```none
  make dirclean
  ```

  > 更换架构编译前必须执行。此操作会删除`/bin`和`/build_dir`目录的中的文件(`make clean`)以及`/staging_dir`、`/toolchain`、`/tmp`和`/logs`中的文件。

- 清除 Open­Wrt 源码以外的文件（可选）

  ```none
  make distclean
  ```

  > 除非是做开发，并打算 push 到 GitHub 这样的远程仓库，否则几乎用不到。此操作相当于`make dirclean`外加删除`/dl`、`/feeds`目录和`.config`文件。

- 清除编译缓存

  ```none
  rm -rf tmp
  ```

  > 此操作据说可防止`make menuconfig`加载错误，暂时没遇到过，如有错误欢迎大佬指正。

- 删除配置文件（可选）

  ```none
  rm -f .config
  ```

  > 可以理解为恢复默认配置，建议切换架构编译前执行。

### 编译

- 调整 Open­Wrt 系统组件

  ```none
  make menuconfig
  ```

  > 如果不打算调整组件则输入`make defconfig`，它会检测编译环境并根据更新自动调整编译配置文件。

- 预下载编译所需的软件包

  ```none
  make download -j8 V=s
  ```

- 检查文件完整性

  ```none
  find dl -size -1024c -exec ls -l {} \;
  ```

  > 此命令可以列出下载不完整的文件（根据我多次编译的经验得出小于1k的文件属于下载不完整），如果存在这样的文件可以使用`find dl -size -1024c -exec rm -f {} \;`命令将它们删除，然后重新执行`make download`下载并反复检查，确认所有文件完整可大大提高编译成功率，避免浪费时间。

- 开始编译

  ```none
  make -j$(nproc) V=s
  ```

  > `-j$(nproc)`：自动获取CPU线程数，采用多线程编译。成功编译后的再次编译且没有进行`make clean`操作时使用。

 

------

原文地址：

https://p3terx.com/archives/openwrt-compilation-steps-and-commands.html