系统版本：Alpine 3.10.3  
如果你的系统版本低于此版本，你可以通过，[一键更新Alpine到最新版](https://www.sbblog.cn/archives/11.html)来一键更新。  
你可用在执行一键安装之前，通过 cat /etc/alpine-release.apk-new 查看下当前系统版本。

```
wget --no-check-certificate -O alnp.sh https://git.io/Jeyks && chmod 755 alnp.sh && ./alnp.sh

```

安装完成后，每次修改nginx配置文件，都需要重启nginx才能生效。/etc/init.d/nginx restart  
重启php7的命令是：/etc/init.d/php-fpm7 restart  
默认路径：/home/wwwroot/default，你可以把程序放在这里面。# 基于Alpine一键安装 Nginx+PHP7+Sqlite3 环境
