# 继续使用Debian 7，让小内存vps焕发生机
大部分主机商都提供了Debian10，CentOS8，但是新系统对内存的要求越来越高。  
目前的根据实际情况，小内存vps它还是有它存在的必要。  
小内存vps在面对最新的系统，运行一些web服务或者$$或者v2服务就显得心有余而力不足。  
装上主机商提供的Debian7模板，修改默认源，让小内存vps焕发生机。  
推荐使用 Debian 7 x86 minimal，或者 Debian 7 x64 minimal。

编辑 /etc/apt/sources.list  
把里面的源删掉，替换为

```
deb http://archive.debian.org/debian wheezy main
deb-src http://archive.debian.org/debian wheezy main

```

然后依次执行下面的命令，精简下系统，精简完成后，输入reboot重启下。

```
apt-get -y update&&apt-get -y upgrade #升级系统
apt-get -y purge apache2-* bind9-* xinetd samba-* nscd-* portmap sendmail-* sasl2-bin #移除多余的软件
apt-get -y purge lynx memtester unixodbc python-* odbcinst-* sudo tcpdump ttf-* #移除多余的组件
apt-get -y autoremove && apt-get clean #清理缓存文件
```