# 雕牌LEDE编译小记

## 1.可在win10下的ubuntu16.04编译,先安装ubuntu

## 2.更新软件包:

```
sudo apt-get update
sudo apt-get upgrade
```

## 3.安装基础环境:

```
sudo apt-get install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint u-boot-tools device-tree-compiler
```

## 4.下载源代码,建立工作目录,进入工作目录,下载好源代码

```
cd
git clone https://github.com/coolsnowwolf/lede
cd lede
```

## 5.更新软件包(feeds都是一些插件):

```
./scripts/feeds update -a
./scripts/feeds install -a
```

## 6.测试编译环境:

```
make defconfig
```

## 7.配置固件菜单：

```
make menuconfig
```

## 8.预先下载dl库，可以避免下载造成的编译失败。

```
make download V=s
```

## 9.开始编译:(首次用j1,后面可以根据cpu自己加如j8)

```
make V=99 -j1
```

## 10.更新代码

```
cd
cd lede
git pull;./scripts/feeds update -a;./scripts/feeds install -a
```