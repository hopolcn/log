# proxmox导入ESXI的虚拟机2019-08-15

# 一、esxi导出ova或ovf的方式

#### 1、首先从esxi导出ova或ovf

如果是ova可以改名iso后缀，从网页上传到proxmox中
 然后解压，如：



```css
tar xvf vm-lede.ova.iso
```

解压成功之后会得到几个文件：
 1.一个ovf文件，这个文件包含虚拟机的硬件配置，例如cpu、内存、硬盘等。
 2.一个或多个vmdk文件，是虚拟机的硬盘镜像，数量取决于虚拟机有多少个硬盘。

- 如果是 ovf就把ovf和vmdk文件用scp或sftp等你熟悉的方式导入proxmox

#### 2、使用命令把ovf导入到proxmox中。

命令如下：



```bash
qm importovf 100 vm-lede.ovf local --format qcow2
```

说明：



```xml
qm importovf <vmid> <manifest> <storage> [OPTIONS]
```

对照上面的，100是虚拟机id 然后跟ovf的路径，最后指定存储，上面的存储名称是local，根据你实际情况，最后 --format指定格式

# 二、直接拷贝esxi里vm的vmdk文件进行转换

#### 1、先转换vmdk文件：



```css
qemu-img convert -f vmdk -O qcow2 xxx.vmdk xxx.qcow2
```

基实-O参数后，也是可以为vmdk的，也就是把esxi的vmdk转成proxmox可用的vmdk

> 附：镜像格式的转换
>  VMDK–>qcow2
>  `qemu-img convert -f vmdk -O qcow2 xxx.vmdk xxx.qcow2`
>  qcow2->VMDK
>  `qemu-img convert -f qcow2 xxx.qcow2 -O vmdk xxx.vmdk`
>  qcow2–>raw
>  `qemu-img convert -O qcow2 xxx.raw xxx.qcow`
>  raw->qcow2
>  `qemu-img convert -f raw -O qcow2 xxx.img xxx.qcow2`

#### 2、在proxmox中添加好虚拟机，然后把转好的硬盘加入

例如：



```bash
qm set 100 --ide1  local:vm-100-disk-0.qcow2
```

其中100是虚拟机的id ， --ide1是指定为ide1，也可以改成sata或scsi,  最后那个local是你存储的名字，根据实际位置，比如是/root/lede.qcow2等。
 ova和ovf的方式简单，还是建议用第一种方式。



作者：龙天ivan
链接：https://www.jianshu.com/p/418295bff320
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。