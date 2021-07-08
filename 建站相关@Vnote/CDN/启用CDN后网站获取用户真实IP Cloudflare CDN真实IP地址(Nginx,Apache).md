# 启用CDN后网站获取用户真实IP Cloudflare CDN真实IP地址(Nginx,Apache)
近期在[其云否](https://wzfou.com/qyfou-cheap-vps/)维护客户的网站时，客户要求屏蔽国外IP的访问，因为从日志来看攻击的IP大部分都是来自国外，并且自己的目标用户为国内，所以只允许国内的IP访问网站可阻止绝大多数的CC和DDoS攻击。实际测试后，发现效果还是不错，攻击想要再次攻击成本增加了不少。

不过，随后发现了一个问题，就是使用了Cloudflare CDN后，网站获取到的IP地址都是Cloudflare的CDN节点的，不能得到真实用户的IP地址，防御效果大大折扣。好在Cloudflare已经为我们想到这一点了，将访问者的 IP 地址包含在  X-Forwarded-For 标头和 CF-Connecting-IP 标头。

有了 X-Forwarded-For 标头，如果是Nginx可以使用[ngx\_http\_realip_module](https://wzfou.com/tag/ngx_http_realip_module/)模块，如果是Apache，则可以使用[mod_remoteip](https://wzfou.com/tag/mod_remoteip/)模块来获取用户的真实IP。本篇文章就来分享一下如何编译和启用ngx\_http\_realip\_module模块和mod\_remoteip模块来获取用户的真实IP地址。

[![启用CDN后网站获取用户真实IP地址：Cloudflare CDN真实IP地址(Nginx,Apache和宝塔)](images/20200219125523654_27255.jpg)](https://wzfou.com/wp-content/uploads/2019/01/cf-block-ip_00.jpg)

一般来说CDN厂商都采用了X-Forwarded-For和X-Real_IP等标准协议，所以本文介绍了获取用户真实IP的访问基本上适用于其它的CDN厂商。更多的关于CDN加速和服务器优化加速的方法，这里有：

1. [Cloudflare Partner接入管理Cloudflare CDN-启用Railgun动态加速](https://wzfou.com/cloudflare-railgun/)
2. [又拍云CDN加速申请使用教程-一键镜像,静态动态CDN和免费SSL](https://wzfou.com/upyun/)
3. [WordPress开启Nginx fastcgi_cache缓存加速方法-Nginx配置实例](https://wzfou.com/nginx-fastcgi-cache/)

## 一、Nginx编译ngx\_http\_realip_module

### 1.1  Oneinstack编译

如果用的是[Oneinstack一键包](https://wzfou.com/oneinstack/)，则可以用以下命令来编译ngx\_http\_realip_module：

1. #下编译安装nginx的时候，都编译安装的哪些模块
2. \[root@wzfoume ~\]# nginx -V
3. nginx version: nginx/1.14.2
4. built by gcc 4.4.7 20120313 (Red Hat 4.4.7-23) (GCC)
5. built with OpenSSL 1.1.1a 20 Nov 2018
6. TLS SNI support enabled
7. configure arguments: --prefix=/usr/local/nginx --user=www --group=www --with-http\_stub\_status\_module --with-http\_v2\_module --with-http\_ssl\_module --with-http\_gzip\_static\_module --with-http\_realip\_module --with-http\_flv\_module --with-http\_mp4\_module --with-openssl=../openssl-1.1.1a --with-pcre=../pcre-8.42 --with-pcre-jit --with-ld-opt=-ljemalloc

9. #进入到oneinstack的nginx安装目录下，如果没有请先解压
10. \[root@wzfoume src\]# cd /root/oneinstack/src
11. \[root@wzfoume src\]# tar xzf nginx-1.14.2.tar.gz
12. \[root@wzfoume src\]# cd /root/oneinstack/src/nginx-1.14.2
13. \[root@wzfoume nginx-1.14.2\]# ./configure --prefix=/usr/local/nginx --user=www --group=www --with-http\_stub\_status\_module --with-http\_v2\_module --with-http\_ssl\_module --with-http\_gzip\_static\_module --with-http\_realip\_module --with-http\_flv\_module --with-http\_mp4\_module --with-openssl=../openssl-1.1.1a --with-pcre=../pcre-8.42 --with-pcre-jit --with-ld-opt=-ljemalloc --with-http\_realip\_module

15. make

17. #如果出现错误，应该是依赖路径不对，请cd ..到上一个目录解压相应的软件
18. tar xzf pcre-8.42.tar.gz
19. tar xzf openssl-1.0.2q.tar.gz
20. tar xzf openssl-1.1.1a.tar.gz

22. #编译完成，备份原先配置，然后替换nginx二进制文件
23. mv /usr/local/nginx/sbin/nginx{,_\`date +%F\`} #备份nginx
24. cp objs/nginx /usr/local/nginx/sbin

26. #查看是否已经把http\_realip\_module模块加入进去
27. nginx -V

### 1.2  LNMP编译

如果你用的是[LNMP一键包](https://wzfou.com/lnmp-1-4/)，在lnmp安装目录下找到lnmp.conf编辑它，在Nginx\_Modules\_Options里加上realip，保存后执行./upgrade.sh nginx来升级下Nginx就可以了。命令如下：

1. Nginx\_Modules\_Options='--with-http\_realip\_module'

### 1.3  BT宝塔面板

如果你用的是[BT宝塔面板](https://wzfou.com/bt-cn/)，可以使用以下命令来编译ngx\_http\_realip_module：

1. #宝塔面板安装模块

3. #先查看一下本机的Nginx配置情况
4. \[root@cs ~\]# nginx -V
5. nginx version: nginx/1.14.2
6. built by gcc 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC)
7. built with OpenSSL 1.0.2l 25 May 2017
8. TLS SNI support enabled
9. configure arguments: --user=www --group=www --prefix=/www/server/nginx --with-openssl=/www/server/nginx/src/openssl --add-module=/www/server/nginx/src/ngx\_devel\_kit --add-module=/www/server/nginx/src/lua\_nginx\_module --add-module=/www/server/nginx/src/ngx\_cache\_purge --add-module=/www/server/nginx/src/nginx-sticky-module --add-module=/www/server/nginx/src/nginx-http-concat --with-http\_stub\_status\_module --with-http\_ssl\_module --with-http\_v2\_module --with-http\_image\_filter\_module --with-http\_gzip\_static\_module --with-http\_gunzip\_module --with-stream --with-stream\_ssl\_module --with-ipv6 --with-http\_sub\_module --with-http\_flv\_module --with-http\_addition\_module --with-http\_realip\_module --with-http\_mp4_module --with-ld-opt=-Wl,-E --with-pcre=pcre-8.40 --with-ld-opt=-ljemalloc
10. #开始下载Nginx，这里用的是1.15.1，你也可以下载其它的版本
11. wget http://nginx.org/download/nginx-1.15.1.tar.gz
12. tar -xzvf nginx-1.15.1.tar.gz
13. cd nginx-1.15.1
14. #下面的命令只是在上面的Nginx -v得到的配置详情后加上了--with-http\_realip\_module，目的是为了保持原来的配置不变同时又增加新的模块
15. ./configure --user=www --group=www --prefix=/www/server/nginx --with-openssl=/www/server/nginx/src/openssl --add-module=/www/server/nginx/src/ngx\_devel\_kit --add-module=/www/server/nginx/src/lua\_nginx\_module --add-module=/www/server/nginx/src/ngx\_cache\_purge --add-module=/www/server/nginx/src/nginx-sticky-module --add-module=/www/server/nginx/src/nginx-http-concat --with-http\_stub\_status\_module --with-http\_ssl\_module --with-http\_v2\_module --with-http\_image\_filter\_module --with-http\_gzip\_static\_module --with-http\_gunzip\_module --with-stream --with-stream\_ssl\_module --with-ipv6 --with-http\_sub\_module --with-http\_flv\_module --with-http\_addition\_module --with-http\_realip\_module --with-http\_mp4\_module --with-ld-opt=-Wl,-E --with-pcre=pcre-8.40 --with-ld-opt=-ljemalloc --with-http\_realip_module
16. #只编译不安装
17. make

19. #先停用Nginx，然后替换新的Nginx并查看模块是否已经加载。命令如下：
20. mv /www/server/nginx/sbin/nginx /www/server/nginx/sbin/nginx-wzfou.backup
21. cp objs/nginx /www/server/nginx/sbin/nginx
22. nginx -V

24. #重启Nginx

## 二、Nginx设置set\_real\_ip_from

编译好了ngx\_http\_realip\_module，现在我们只需要在Nginx配置文件中添加set\_real\_ip\_from代码，示例如下：

1. set\_real\_ip_from 222.222.222.222; #这里是需要填写具体的CDN服务器IP地址,可添加多个
2. set\_real\_ip_from 222.222.111.111;
3. real\_ip\_header X-Forwarded-For;
4. real\_ip\_recursive on;

如果你用的是CloudFlare免费CDN，请将以下代码加入到你的Nginx配置文件当中。

1. location / {
2. set\_real\_ip_from 103.21.244.0/22;
3. set\_real\_ip_from 103.22.200.0/22;
4. set\_real\_ip_from 103.31.4.0/22;
5. set\_real\_ip_from 104.16.0.0/12;
6. set\_real\_ip_from 108.162.192.0/18;
7. set\_real\_ip_from 131.0.72.0/22;
8. set\_real\_ip_from 141.101.64.0/18;
9. set\_real\_ip_from 162.158.0.0/15;
10. set\_real\_ip_from 172.64.0.0/13;
11. set\_real\_ip_from 173.245.48.0/20;
12. set\_real\_ip_from 188.114.96.0/20;
13. set\_real\_ip_from 190.93.240.0/20;
14. set\_real\_ip_from 197.234.240.0/22;
15. set\_real\_ip_from 198.41.128.0/17;
16. set\_real\_ip_from 199.27.128.0/21;
17. set\_real\_ip_from 2400:cb00::/32;
18. set\_real\_ip_from 2606:4700::/32;
19. set\_real\_ip_from 2803:f800::/32;
20. set\_real\_ip_from 2405:b500::/32;
21. set\_real\_ip_from 2405:8100::/32;
22. set\_real\_ip_from 2c0f:f248::/32;
23. set\_real\_ip_from 2a06:98c0::/29;
24. \# use any of the following two
25. real\_ip\_header CF-Connecting-IP;
26. #real\_ip\_header X-Forwarded-For;
27. }

29. #不要忘记重启nginx
30. service nginx restart

一般来说CloudFlare的IP地址是不会变的，你可以在这里找到：https://www.cloudflare.com/ips/，但是为了以防万一，wzfou.com建议设置一个自动更新CloudFlare的IP的定时任务，自动将最新的IP添加到Nginx的配置文件当中。代码如下：

1. #在nginx配置目录创建cloudflare_ip.conf文件
2. touch /usr/local/nginx/conf/cloudflare_ip.conf

4. #修改原有的vhost配置，将原来第五步配置的信息改为
5. include cloudflare_ip.conf;

7. #创建自更新脚本update\_cloudflare\_ip.sh（假定该文件放在 /root 目录下），内容如下：

9. #!/bin/bash
10. echo "#Cloudflare" \> /usr/local/nginx/conf/cloudflare_ip.conf;
11. **for** i **in**  \`curl https://www.cloudflare.com/ips-v4\`; **do**
12. echo "set\_real\_ip_from $i;" >\> /usr/local/nginx/conf/cloudflare_ip.conf;
13. **done**
14. **for** i **in**  \`curl https://www.cloudflare.com/ips-v6\`; **do**
15. echo "set\_real\_ip_from $i;" >\> /usr/local/nginx/conf/cloudflare_ip.conf;
16. **done**
17. echo "" >\> /usr/local/nginx/conf/cloudflare_ip.conf;
18. echo "# use any of the following two" >\> /usr/local/nginx/conf/cloudflare_ip.conf;
19. echo "real\_ip\_header CF-Connecting-IP;" >\> /usr/local/nginx/conf/cloudflare_ip.conf;
20. echo "#real\_ip\_header X-Forwarded-For;" >\> /usr/local/nginx/conf/cloudflare_ip.conf;

23. #配置crontab 每周一的上午5点更新
24. 0 5 * * 1 /bin/bash /root/update\_cloudflare\_ip.sh

## 三、Apache配置mod_remoteip模块

### 3.1  apache 2.4

apache 2.4自带mod_remoteip模块不需要安装，按照下文操作：

1. #启用模块

3. vim /usr/local/apache/conf/httpd.conf

5. Include conf/extra/httpd-remoteip.conf

7. #添加如下内容
8. vim /usr/local/apache/conf/extra/httpd-remoteip.conf

10. LoadModule remoteip\_module modules/mod\_remoteip.so
11. RemoteIPHeader X-Forwarded-For
12. RemoteIPInternalProxy 127.0.0.1/24
13. #CloudFlare IP Ranges
14. RemoteIPInternalProxy 103.21.244.0/22
15. RemoteIPInternalProxy 103.22.200.0/22
16. RemoteIPInternalProxy 103.31.4.0/22
17. RemoteIPInternalProxy 104.16.0.0/12
18. RemoteIPInternalProxy 108.162.192.0/18
19. RemoteIPInternalProxy 131.0.72.0/22
20. RemoteIPInternalProxy 141.101.64.0/18
21. RemoteIPInternalProxy 162.158.0.0/15
22. RemoteIPInternalProxy 172.64.0.0/13
23. RemoteIPInternalProxy 173.245.48.0/20
24. RemoteIPInternalProxy 188.114.96.0/20
25. RemoteIPInternalProxy 190.93.240.0/20
26. RemoteIPInternalProxy 197.234.240.0/22
27. RemoteIPInternalProxy 198.41.128.0/17 #你的CDN的IP，可以重复添加

29. #修改日志格式，在日志格式中加上%a，然后重启apache即可

31. LogFormat "%h %a %l %u %t \\"%r\\" %>s %b \\"%{Referer}i\\" \\"%{User-Agent}i\\"" combined
32. LogFormat "%h %a %l %u %t \\"%r\\" %>s %b" common
33. LogFormat "%h %l %u %t \\"%r\\" %>s %b \\"%{Referer}i\\" \\"%{User-Agent}i\\" %I %O" combined

### 3.2  apache 2.2

apache 2.2需要安装mod_remoteip模块，方法如下：

1. wget https://github.com/ttkzw/mod\_remoteip-httpd22/raw/master/mod\_remoteip.c
2. /usr/local/apache/bin/apxs -i -c -n mod\_remoteip.so mod\_remoteip.c

4. #启用模块
5. vim /usr/local/apache/conf/httpd.conf

7. Include conf/extra/httpd-remoteip.conf

9. #添加如下内容，然后重启apache即可
10. vim /usr/local/apache/conf/extra/httpd-remoteip.conf

12. LoadModule remoteip\_module modules/mod\_remoteip.so
13. RemoteIPHeader X-Forwarded-For
14. RemoteIPInternalProxy 127.0.0.1 #你的CDN的IP，可以重复添加

## 四、网站仅允许Cloudflare CDN的IP访问

上面我们是通过安装ngx\_http\_realip\_module和mod\_remoteip模块获取到了用户真实的IP地址，但是有的时候我们需要借助Cloudflare 的安全防护功能来防止CC或者DDoS攻击，即仅允许Cloudflare CDN的IP访问我们的访问。

Nginx直接拒绝和允许IP访问代码示例如下：

1. location / {
2. deny 192.168.1.1;
3. allow 192.168.1.0/24;
4. allow 10.1.1.0/16;
5. allow 2001:0db8::/32;
6. #Railgun IP
7. deny all;
8. }

如果我们仅允许Cloudflare CDN的IP访问网站，我们可以直接在nginx配置中将Cloudflare CDN的IP添加到允许的范围内。

1. #直接加入
2. \# https://www.cloudflare.com/ips
3. \# IPv4
4. allow 103.21.244.0/22;
5. allow 103.22.200.0/22;
6. allow 103.31.4.0/22;
7. allow 104.16.0.0/12;
8. allow 108.162.192.0/18;
9. allow 131.0.72.0/22;
10. allow 141.101.64.0/18;
11. allow 162.158.0.0/15;
12. allow 172.64.0.0/13;
13. allow 173.245.48.0/20;
14. allow 188.114.96.0/20;
15. allow 190.93.240.0/20;
16. allow 197.234.240.0/22;
17. allow 198.41.128.0/17;

19. \# IPv6
20. allow 2400:cb00::/32;
21. allow 2405:8100::/32;
22. allow 2405:b500::/32;
23. allow 2606:4700::/32;
24. allow 2803:f800::/32;
25. allow 2c0f:f248::/32;
26. allow 2a06:98c0::/29;

**自动更新Cloudflare CDN的IP。**手动添加Cloudflare CDN的IP到Nginx配置当中简单方便，但是一旦Cloudflare CDN的IP有变化时还得自己手动处理，我们可以创建一个脚本，定时去更新Cloudflare CDN的IP，自动添加到Nginx配置中，代码如下：

1. touch /usr/local/nginx/conf/allow_ip.conf
2. #修改网站nginx配置，加入以下代码：
3. include /usr/local/nginx/conf/allow_ip.conf;

5. vim /data/script/allow\_cf\_ip.sh
6. #!/bin/bash
7. echo "#Cloudflare" \> /usr/local/nginx/conf/allow_ip.conf;
8. **for** i **in**  \`curl https://www.cloudflare.com/ips-v4\`; **do**
9. echo "allow $i;" >\> /usr/local/nginx/conf/allow_ip.conf;
10. **done**
11. **for** i **in**  \`curl https://www.cloudflare.com/ips-v6\`; **do**
12. echo "allow $i;" >\> /usr/local/nginx/conf/allow_ip.conf;
13. **done**

16. #添加定时任务
17. 0 5 * * 1 /bin/bash /data/script/allow\_cf\_ip.sh

## 五、总结

使用了CDN加速后，我们的网站获取到的用户IP变成了CDN的IP了，想要获取到用户的真实IP就得利用Nginx和Apache的模式功能。当然，如果你用的是PHP,例如Wordpress，直接将以下代码加入到你的Wordpress配置文件当中即可。

1. **if**(**isset**($_SERVER\['HTTP\_X\_FORWARDED_FOR'\]))
2. {
3. $list  =  explode(‘,’,$_SERVER\['HTTP\_X\_FORWARDED_FOR'\]);
4. $_SERVER\['REMOTE_ADDR'\]  =  $list\[0\];
5. }

这里还要特别提醒一下，如果你启用了Cloudflare Railgun动态加速（[挖站否的Cloudflare Partner接入管理](https://wzfou.com/cloudflare-railgun/)就提供此免费服务）,记得将Railgun的服务器IP加入到配置当中，因为启用了Railgun后网站获取到的IP地址都来自Railgun服务器上的。