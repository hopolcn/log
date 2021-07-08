# 一键更新Alpine到最新版
之前博客介绍了：[Alpine Linux一键安装脚本](https://www.sbblog.cn/archives/10.html)，安装的Alpine版本是3.9版，目前最新版本是3.11.2。  
下面的脚本，会一键更新alpine到3.11.2，并更新时区为上海，本脚本会持续更新。

```
apk add ca-certificates&& update-ca-certificates && apk --no-cache add openssl wget && wget --no-check-certificate -O alpine-update.sh https://git.io/JeDdM && chmod 755 alpine-update.sh &&./alpine-update.sh

```

更新完成后，会重启，重启一般几秒钟就可以完成，重新连上SSH后，可以通过

```
cat /etc/alpine-release.apk-new
或者
cat /etc/issue

```

查看版本。