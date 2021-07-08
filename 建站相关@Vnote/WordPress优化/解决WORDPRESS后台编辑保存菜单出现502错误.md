# 解决WORDPRESS后台编辑保存菜单出现502错误
由于Wordpress的菜单选项越来越多，导致在编辑和保存Wordpress菜单时出现了502错误，初步判定是PHP执行超时导致的，解决办法主要是调整php-fpm.conf配置的执行超时时长，增加pm.max\_requests和pm.max\_children，**加大request\_terminate\_timeout**。

优化前，配置如下：

```
pm = dynamic
pm.max_children = 33
pm.start_servers = 22
pm.min_spare_servers = 16
pm.max_spare_servers = 33
pm.max_requests = 2048
pm.process_idle_timeout = 10s
request_terminate_timeout = 120
request_slowlog_timeout = 0
```

优化后，配置如下：

```
pm = dynamic
pm.max_children = 43
pm.start_servers = 22
pm.min_spare_servers = 16
pm.max_spare_servers = 43
pm.max_requests = 6000
pm.process_idle_timeout = 10s
request_terminate_timeout = 600
request_slowlog_timeout = 0
```

有关于php-fpm优化心得，参考这里：[https://wzfou.com/php-fpm/](https://wzfou.com/php-fpm/)