# proxmox lxc alpine安装docker 2019-09-06

\>准备工作是先用模板安装好alpine，比如3.10，勾选 **特权容器**。
 lxc的配置文件：



```css
lxc.apparmor.profile: unconfined
lxc.mount.auto: cgroup:rw
lxc.mount.auto: proc:rw
lxc.mount.auto: sys:rw
```

启动后：



```c
echo "http://mirrors.aliyun.com/alpine/latest-stable/main/" > /etc/apk/repositories \
 && echo "http://mirrors.aliyun.com/alpine/latest-stable/community/"  >> /etc/apk/repositories
apk add docker
service docker start
rc-update add docker boot
```



作者：龙天ivan
链接：https://www.jianshu.com/p/abd94d9a7916
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。