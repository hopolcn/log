# Proxmox VE配置嵌套虚拟化

Proxmox VE（PVE）虚拟出来的主机CPU默认不支持嵌套虚拟化（vmx），参照KVM支持嵌套虚拟化的方法开启nested。



```c
cat /sys/module/kvm_intel/parameters/nested
N
```

显示默认状态下是未开启。
 打开嵌套虚拟化需要关闭所有虚拟机。
 列出所有虚拟机：



```c
qm list
```

关闭虚拟机：



```xml
qm stop <vmid>
```

批量关闭：



```bash
for i in {100..115};do qm stop $i;done   #{}号中间填vmid的起止数字
```

开启内核支持：



```bash
modprobe -r kvm_intel  
modprobe kvm_intel nested=1  #如果报错Module kvm_intel is in use，请检查你的虚拟机是否全部关闭
```

现在再看看nested是否已开启：



```ruby
cat /sys/module/kvm_intel/parameters/nested
Y
```

编辑配置文件：



```bash
echo "options kvm_intel nested=1" >> /etc/modprobe.d/modprobe.conf
```

让系统重启自动加载netsted

最后，查看虚拟机启动命令配置：



```xml
qm showcmd <vmid>
```

找到-cpu kvm64,+lahf_lm,+sep,+kvm_pv_unhalt,+kvm_pv_eoi,enforce
 在后面加上+vmx，表示开启vmx
 加入配置文件



```jsx
vi /etc/pve/qemu-server/<vmid>.conf
```

可以加到第一行，添加一个args参数，示例：



```c
args: -cpu kvm64,+lahf_lm,+sep,+kvm_pv_unhalt,+kvm_pv_eoi,enforce,+vmx
bootdisk: scsi0
cores: 1
ide2: d02:iso/proxmox-ve_5.4-1.iso,media=cdrom
memory: 512
name: pve
net0: virtio=7E:89:5B:25:10:59,bridge=vmbr0,firewall=1
numa: 0
ostype: l26
scsi0: vms:vm-103-disk-0,size=8G
scsihw: virtio-scsi-pci
smbios1: uuid=094fdd5d-ae0b-4af6-aa08-c23c445f9da0
sockets: 1
startup: down=0
vmgenid: 8813ce4c-1248-4964-86da-83a3f1cd6738
```

------

为了简化操作，我写了个pve工具：
 [https://github.com/ivanhao/pvetools.git](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fivanhao%2Fpvetools.git)
 其中就包含上面的内容而且是自动化配置，很方便实用。
 如果好用请去github页面点星点赞，

# qq交流群: 878510703



作者：龙天ivan
链接：https://www.jianshu.com/p/159345fc5372
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。