# proxmox控制cpu频率实现降频降温省电2019-08-19

# 一、首先修改启动项



```c
root@pve:~# vi /etc/default/grub
```

找到quiet那个位置，后面加空格，再加`intel_pstate=disable`
 加上参数后保存，并更新grub



```c
root@pve:~# update-grub
```

保存完后重启生效。

# 二、安装cpufrequtils



```c
root@pve:~# apt-get install cpufrequtils
```

# 三、配置

先查看一下信息



```c
root@pve:~# cpufreq-info
cpufrequtils 008: cpufreq-info (C) Dominik Brodowski 2004-2009
Report errors and bugs to cpufreq@vger.kernel.org, please.
analyzing CPU 0:
  driver: acpi-cpufreq
  CPUs which run at the same hardware frequency: 0
  CPUs which need to have their frequency coordinated by software: 0
  maximum transition latency: 10.0 us.
  hardware limits: 1.60 GHz - 2.30 GHz
  available frequency steps: 2.30 GHz, 2.20 GHz, 2.10 GHz, 2.00 GHz, 1.90 GHz, 1.80 GHz, 1.70 GHz, 1.60 GHz
  available cpufreq governors: conservative, ondemand, userspace, powersave, performance, schedutil
  current policy: frequency should be within 1.60 GHz and 1.60 GHz.
                  The governor "performance" may decide which speed to use
                  within this range.
  current CPU frequency is 2.30 GHz (asserted by call to hardware).
  cpufreq stats: 2.30 GHz:12.35%, 2.20 GHz:0.00%, 2.10 GHz:0.00%, 2.00 GHz:5.16%, 1.90 GHz:0.00%, 1.80 GHz:0.00%, 1.70 GHz:0.00%, 1.60 GHz:82.49%  (4)
analyzing CPU 1:
  driver: acpi-cpufreq
  CPUs which run at the same hardware frequency: 1
  CPUs which need to have their frequency coordinated by software: 1
  maximum transition latency: 10.0 us.
  hardware limits: 1.60 GHz - 2.30 GHz
  available frequency steps: 2.30 GHz, 2.20 GHz, 2.10 GHz, 2.00 GHz, 1.90 GHz, 1.80 GHz, 1.70 GHz, 1.60 GHz
  available cpufreq governors: conservative, ondemand, userspace, powersave, performance, schedutil
  current policy: frequency should be within 1.60 GHz and 1.60 GHz.
                  The governor "performance" may decide which speed to use
                  within this range.
  current CPU frequency is 2.30 GHz (asserted by call to hardware).
  cpufreq stats: 2.30 GHz:12.35%, 2.20 GHz:0.00%, 2.10 GHz:0.00%, 2.00 GHz:2.45%, 1.90 GHz:0.00%, 1.80 GHz:0.00%, 1.70 GHz:0.00%, 1.60 GHz:85.21%  (4)
```

可以看到目前是performance模式，主频是2.30 GHz ，下面就配置一下，改成省电模式（powersave) 以最低允许的主频运行



```c
root@pve:~# vi /etc/init.d/cpufrequtils
```

找到如下：



```c
ENABLE="true"
GOVERNOR="performance"
MAX_SPEED="2300000"
MIN_SPEED="1600000"
```

改成：



```c
ENABLE="true"
GOVERNOR="powersave"
MAX_SPEED="1600000"
MIN_SPEED="1600000"
```

保存后，重启服务：



```c
root@pve:~# systemctl daemon-reload
root@pve:~# /etc/init.d/cpufrequtils restart
```

> 如果不生效，查看一下是否有/etc/default/cpufrequtils文件，如果有，在这里修改就可以了。

# 四、查看



```c
root@pve:~# cpufreq-info
cpufrequtils 008: cpufreq-info (C) Dominik Brodowski 2004-2009
Report errors and bugs to cpufreq@vger.kernel.org, please.
analyzing CPU 0:
  driver: acpi-cpufreq
  CPUs which run at the same hardware frequency: 0
  CPUs which need to have their frequency coordinated by software: 0
  maximum transition latency: 10.0 us.
  hardware limits: 1.60 GHz - 2.30 GHz
  available frequency steps: 2.30 GHz, 2.20 GHz, 2.10 GHz, 2.00 GHz, 1.90 GHz, 1.80 GHz, 1.70 GHz, 1.60 GHz
  available cpufreq governors: conservative, ondemand, userspace, powersave, performance, schedutil
  current policy: frequency should be within 1.60 GHz and 1.60 GHz.
                  The governor "powersave" may decide which speed to use
                  within this range.
  current CPU frequency is 1.60 GHz (asserted by call to hardware).
  cpufreq stats: 2.30 GHz:12.35%, 2.20 GHz:0.00%, 2.10 GHz:0.00%, 2.00 GHz:5.16%, 1.90 GHz:0.00%, 1.80 GHz:0.00%, 1.70 GHz:0.00%, 1.60 GHz:82.49%  (4)
analyzing CPU 1:
  driver: acpi-cpufreq
  CPUs which run at the same hardware frequency: 1
  CPUs which need to have their frequency coordinated by software: 1
  maximum transition latency: 10.0 us.
  hardware limits: 1.60 GHz - 2.30 GHz
  available frequency steps: 2.30 GHz, 2.20 GHz, 2.10 GHz, 2.00 GHz, 1.90 GHz, 1.80 GHz, 1.70 GHz, 1.60 GHz
  available cpufreq governors: conservative, ondemand, userspace, powersave, performance, schedutil
  current policy: frequency should be within 1.60 GHz and 1.60 GHz.
                  The governor "powersave" may decide which speed to use
                  within this range.
  current CPU frequency is 1.60 GHz (asserted by call to hardware).
  cpufreq stats: 2.30 GHz:12.35%, 2.20 GHz:0.00%, 2.10 GHz:0.00%, 2.00 GHz:2.45%, 1.90 GHz:0.00%, 1.80 GHz:0.00%, 1.70 GHz:0.00%, 1.60 GHz:85.21%  (4)
```

看到已经修改生效了。

经过测试，省电模式下，待机会保持在最低功耗，温度也最低。



作者：龙天ivan
链接：https://www.jianshu.com/p/b407baa9cb27
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。