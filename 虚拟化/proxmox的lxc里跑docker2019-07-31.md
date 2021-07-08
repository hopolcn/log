# **proxmox的lxc里跑docker2019-07-31**

需要修改lxc配置文件：
 /etc/pve/lxc/ID.conf
 在最后加入



```go
lxc.aa_profile = unconfined
lxc.cgroup.devices.allow = a
lxc.cap.drop =
lxc.hook.mount =
lxc.hook.post-stop =
```

如果是通过模板创建的，配置文件中模板和当前的配置最后都需要加上上面那段。



作者：龙天ivan
链接：https://www.jianshu.com/p/965e3555f8d3
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。