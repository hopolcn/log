# Alpine Linux下搭建$$R一键脚本,内存占用极少
现阶段有很多小内存的小鸡鸡出现，就算做个mini的镜像，开机占用都不算太好。
所以let有人写了一个openvz构架重装Alpine Linux的教程，试了下，的确不错。
开机占用8M内存，最主要的是，麻雀虽小，五脏俱全。今天花了点时间搞了Alpine Linux的$$R脚本。有喜欢的继续看：

开机占用：


安装$$后的占用：


废话不多，直接上脚本：

apk update
apk add py3-lxml
apk add python3
pip3 install pip==10.0.0
pip3 freeze
apk add --no-cache --virtual .build-deps tar
apk add --no-cache --virtual .build-deps wget
apk add --no-cache --virtual .build-deps openssl
apk add --no-cache --virtual .build-depslibsodium-dev
wget -O /tmp/shadowsocksr-3.2.2.tar.gz https://github.com/shadowsocksrr/shadowsocksr/archive/3.2.2.tar.gz
tar zxf /tmp/shadowsocksr-3.2.2.tar.gz -C /tmp
mv /tmp/shadowsocksr-3.2.2/shadowsocks /usr/local/
rm -fr /tmp/shadowsocksr-3.2.2
rm -f /tmp/shadowsocksr-3.2.2.tar.gz
cd /root
echo '{
"server":"0.0.0.0",
"server_ipv6":"::",
"server_port":9000,
"local_address":"127.0.0.1",
"local_port":1080,
"password":"password0",
"timeout":120,
"method":"aes-256-cfb",
"protocol":"origin",
"protocol_param":"",
"obfs":"plain",
"obfs_param":"",
"redirect":"",
"dns_ipv6":false,
"fast_open":true,
"workers":1
}' >1.json
nohup python3 /usr/local/shadowsocks/server.py -c /root/1.json &
echo "nohup python3 /usr/local/shadowsocks/server.py -c /root/1.json & " >/etc/local.d/ss.start
chmod +x /etc/local.d/ss.start
rc-update add local

直接复制到终端运行即可，如果需要详细教程，看这里：https://4ker.cc/alpine-install-shadowsocksr.html

有人问了，如何重装为Alpine Linux呢？ 看这里：https://4ker.cc/linux-to-apline.html  （不敢用我的脚本文章底部有let链接，用作者的最好）


有人说整合到Docker？
答：个人觉得没这个必要，直接docker pull alpine之后，运行docker，留出一个映射端口，然后Docker内运行上面的脚本，Docker保留，或者重新commit一下，不就ojbk啦！
不喜勿喷，免费分享的


## 二次更新说明:

由于ssrr项目被移除，更新安装程序。替换为博客备份版本。

安装Alpine Linux: Linux下OpenVZ平台Alpine Linux一键安装脚本

重装Alpine Linux后的内存占用：



下面上安装脚本：
（建议使用上面的来重装，如果是Docker pull的Alpine的话，没有包含rc-update服务，无法实现开机自启，需要手动启动。）

apk update
apk add py3-lxml
apk add python3
pip3 install pip==10.0.0
pip3 freeze
apk add --no-cache --virtual .build-deps tar
apk add --no-cache --virtual .build-deps wget
apk add --no-cache --virtual .build-deps openssl
apk add --no-cache --virtual .build-depslibsodium-dev
apk add curl unzip
cd /usr/local/
curl -O  https://cikeblog.com/s/shadowsocksr.zip
unzip shadowsocksr.zip
cd /usr/local/shadowsocks/shadowsocks
echo '{
"server":"0.0.0.0",
"server_ipv6":"::",
"server_port":9000,
"local_address":"127.0.0.1",
"local_port":1080,
"password":"password0",
"timeout":120,
"method":"aes-256-cfb",
"protocol":"origin",
"protocol_param":"",
"obfs":"plain",
"obfs_param":"",
"redirect":"",
"dns_ipv6":false,
"fast_open":true,
"workers":1
}' &gt;1.json
nohup python3 /usr/local/shadowsocks/shadowsocks/server.py -c /usr/local/shadowsocks/shadowsocks/1.json &amp;
echo "nohup python3 /usr/local/shadowsocks/shadowsocks/server.py -c  /usr/local/shadowsocks/shadowsocks/1.json &amp; " &gt;/etc/local.d/ss.start
chmod +x /etc/local.d/ss.start
rc-update add local
把上面的代码复制到终端执行即可，最好一行行复制执行。

安装完成后，我们执行：

netstat -ntlp
查看shadowsocksr运行状态，如果需要修改端口和加密方式的话，请修改/root/1.json即可，由于Alpine的运行方式，我们修改后需要kill掉ss的进程，再执行：

nohup python3 /usr/local/shadowsocks/shadowsocks/server.py -c  /usr/local/shadowsocks/shadowsocks/1.json &amp;
来进行后台守护，如果觉着麻烦，直接写进/bin目录下，作为程序运行也可以。

安装完成后重启，包含ss和系统的内存占用如下：



因为ShadowsocksR的进程会递增，如果小内存的vps的话，可以考虑下更换为Shadowsocks-go等占用小的程序运行。后面说不定我会发布ss版本的Alpine。