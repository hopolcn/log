## FRP架设在 Docker

https://www.jianshu.com/p/1272fbbecfeb/



参考：
 [1.内网穿透服务器](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fqq_34178764%2Farticle%2Fdetails%2F103451117)
 [2.https://github.com/fatedier/frp#admin-ui](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Ffatedier%2Ffrp%23admin-ui)
 [3.https://github.com/fatedier/frp#admin-ui](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Ffatedier%2Ffrp%23admin-ui)
 [4.https://www.jianshu.com/p/f3fae889bc9c](https://www.jianshu.com/p/f3fae889bc9c)
 [5.http://auan.cn/server/1680.html](https://links.jianshu.com/go?to=http%3A%2F%2Fauan.cn%2Fserver%2F1680.html)

将frps.ini服务器的配置文件放到/var/frp/conf中，也可以根据自己的情况进行修改

![img](images/21643744-cea37e6071764c02.png)

```
docker pull cloverzrg/frps-docker
//conf的配置文件预先放在了/var/frp/conf，下面命令会将其进行拷贝。实际可以根据自己的进行修改

docker run -d --name frp-server --restart=always -p 41004:41004 -p 41003:41003  -p 41005:41005 -p 41002:41002/udp -p 43000-43010:43000-43010/udp -p 43000-43010:43000-43010/tcp  -v /var/frp/conf:/conf --restart=always cloverzrg/frps-docker
//端口根据自己的情况进行修改
```

//查看是否成功

```
[root@rbtnode1 ~]# docker ps -a
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                                                                                                                                        NAMES

46731feed5f1        cloverzrg/frps-docker   "./frps -c /conf/frp…"   2 days ago          Up 41 hours         0.0.0.0:41003-41005->41003-41005/tcp, 0.0.0.0:41002->41002/udp, 0.0.0.0:43000-43010->43000-43010/udp, 0.0.0.0:43000-43010->43000-43010/tcp   frp-server
```

确保防火墙已经允许这些端口的使用

> 完整的服务端配置[https://github.com/fatedier/frp/blob/master/conf/frps_full.ini](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Ffatedier%2Ffrp%2Fblob%2Fmaster%2Fconf%2Ffrps_full.ini)

```json
# [common] is integral section
[common]
# A literal address or host name for IPv6 must be enclosed
# in square brackets, as in "[::1]:80", "[ipv6-host]:http" or "[ipv6-host%zone]:80"
bind_addr = 0.0.0.0
bind_port = 7000

# udp port to help make udp hole to penetrate nat
bind_udp_port = 7001

# udp port used for kcp protocol, it can be same with 'bind_port'
# if not set, kcp is disabled in frps
kcp_bind_port = 7000

# specify which address proxy will listen for, default value is same with bind_addr
# proxy_bind_addr = 127.0.0.1

# if you want to support virtual host, you must set the http port for listening (optional)
# Note: http port and https port can be same with bind_port
vhost_http_port = 80
vhost_https_port = 443

# response header timeout(seconds) for vhost http server, default is 60s
# vhost_http_timeout = 60

# set dashboard_addr and dashboard_port to view dashboard of frps
# dashboard_addr's default value is same with bind_addr
# dashboard is available only if dashboard_port is set
dashboard_addr = 0.0.0.0
dashboard_port = 7500

# dashboard user and passwd for basic auth protect, if not set, both default value is admin
dashboard_user = admin
dashboard_pwd = admin

# dashboard assets directory(only for debug mode)
# assets_dir = ./static
# console or real logFile path like ./frps.log
log_file = ./frps.log

# trace, debug, info, warn, error
log_level = info

log_max_days = 3

# disable log colors when log_file is console, default is false
disable_log_color = false

# auth token
token = 12345678

# heartbeat configure, it's not recommended to modify the default value
# the default value of heartbeat_timeout is 90
# heartbeat_timeout = 90

# only allow frpc to bind ports you list, if you set nothing, there won't be any limit
allow_ports = 2000-3000,3001,3003,4000-50000

# pool_count in each proxy will change to max_pool_count if they exceed the maximum value
max_pool_count = 5

# max ports can be used for each client, default value is 0 means no limit
max_ports_per_client = 0

# if subdomain_host is not empty, you can set subdomain when type is http or https in frpc's configure file
# when subdomain is test, the host used by routing is test.frps.com
subdomain_host = frps.com

# if tcp stream multiplexing is used, default is true
tcp_mux = true

# custom 404 page for HTTP requests
# custom_404_page = /path/to/404.html

[plugin.user-manager]
addr = 127.0.0.1:9000
path = /handler
ops = Login

[plugin.port-manager]
addr = 127.0.0.1:9001
path = /handler
ops = NewProxy
```

## 安装frp客户端

安装客户端：

下载客户端软件：

Mac版本

[https://github.com/fatedier/frp/releases/download/v0.20.0/frp_0.20.0_darwin_amd64.tar.gz](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Ffatedier%2Ffrp%2Freleases%2Fdownload%2Fv0.20.0%2Ffrp_0.20.0_darwin_amd64.tar.gz)

Win版本

[https://github.com/fatedier/frp/releases/download/v0.20.0/frp_0.20.0_windows_386.zip](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Ffatedier%2Ffrp%2Freleases%2Fdownload%2Fv0.20.0%2Ffrp_0.20.0_windows_386.zip)

下载完成后解压，找到 frpc.ini 文件

修改 frpc.ini 文件，假设 frps 所在的服务器的 IP 为 180.28.83.22，local_port 为本地机器上 web 服务对应的端口, 绑定自定义域名 web.frp.auan.cn或者直接写公网IP地址，根据实际情况修改
 根据实际要映射的端口和方式进行配置

![img](images/21643744-ecd22b427eae7b1c.png)

完整的客户端配置可参考

```json
# [common] is integral section
[common]
# A literal address or host name for IPv6 must be enclosed
# in square brackets, as in "[::1]:80", "[ipv6-host]:http" or "[ipv6-host%zone]:80"
server_addr = 0.0.0.0
server_port = 7000

# if you want to connect frps by http proxy or socks5 proxy, you can set http_proxy here or in global environment variables
# it only works when protocol is tcp
# http_proxy = http://user:passwd@192.168.1.128:8080
# http_proxy = socks5://user:passwd@192.168.1.128:1080

# console or real logFile path like ./frpc.log
log_file = ./frpc.log

# trace, debug, info, warn, error
log_level = info

log_max_days = 3

# disable log colors when log_file is console, default is false
disable_log_color = false

# for authentication
token = 12345678

# set admin address for control frpc's action by http api such as reload
admin_addr = 127.0.0.1
admin_port = 7400
admin_user = admin
admin_pwd = admin
# Admin assets directory. By default, these assets are bundled with frpc.
# assets_dir = ./static

# connections will be established in advance, default value is zero
pool_count = 5

# if tcp stream multiplexing is used, default is true, it must be same with frps
tcp_mux = true

# your proxy name will be changed to {user}.{proxy}
user = your_name

# decide if exit program when first login failed, otherwise continuous relogin to frps
# default is true
login_fail_exit = true

# communication protocol used to connect to server
# now it supports tcp and kcp and websocket, default is tcp
protocol = tcp

# if tls_enable is true, frpc will connect frps by tls
tls_enable = true

# specify a dns server, so frpc will use this instead of default one
# dns_server = 8.8.8.8

# proxy names you want to start seperated by ','
# default is empty, means all proxies
# start = ssh,dns

# heartbeat configure, it's not recommended to modify the default value
# the default value of heartbeat_interval is 10 and heartbeat_timeout is 90
# heartbeat_interval = 30
# heartbeat_timeout = 90

# additional meta info for client
meta_var1 = 123
meta_var2 = 234

# 'ssh' is the unique proxy name
# if user in [common] section is not empty, it will be changed to {user}.{proxy} such as 'your_name.ssh'
[ssh]
# tcp | udp | http | https | stcp | xtcp, default is tcp
type = tcp
local_ip = 127.0.0.1
local_port = 22
# limit bandwidth for this proxy, unit is KB and MB
bandwidth_limit = 1MB
# true or false, if true, messages between frps and frpc will be encrypted, default is false
use_encryption = false
# if true, message will be compressed
use_compression = false
# remote port listen by frps
remote_port = 6001
# frps will load balancing connections for proxies in same group
group = test_group
# group should have same group key
group_key = 123456
# enable health check for the backend service, it support 'tcp' and 'http' now
# frpc will connect local service's port to detect it's healthy status
health_check_type = tcp
# health check connection timeout
health_check_timeout_s = 3
# if continuous failed in 3 times, the proxy will be removed from frps
health_check_max_failed = 3
# every 10 seconds will do a health check
health_check_interval_s = 10
# additional meta info for each proxy
meta_var1 = 123
meta_var2 = 234

[ssh_random]
type = tcp
local_ip = 127.0.0.1
local_port = 22
# if remote_port is 0, frps will assign a random port for you
remote_port = 0

# if you want to expose multiple ports, add 'range:' prefix to the section name
# frpc will generate multiple proxies such as 'tcp_port_6010', 'tcp_port_6011' and so on.
[range:tcp_port]
type = tcp
local_ip = 127.0.0.1
local_port = 6010-6020,6022,6024-6028
remote_port = 6010-6020,6022,6024-6028
use_encryption = false
use_compression = false

[dns]
type = udp
local_ip = 114.114.114.114
local_port = 53
remote_port = 6002
use_encryption = false
use_compression = false

[range:udp_port]
type = udp
local_ip = 127.0.0.1
local_port = 6010-6020
remote_port = 6010-6020
use_encryption = false
use_compression = false

# Resolve your domain names to [server_addr] so you can use http://web01.yourdomain.com to browse web01 and http://web02.yourdomain.com to browse web02
[web01]
type = http
local_ip = 127.0.0.1
local_port = 80
use_encryption = false
use_compression = true
# http username and password are safety certification for http protocol
# if not set, you can access this custom_domains without certification
http_user = admin
http_pwd = admin
# if domain for frps is frps.com, then you can access [web01] proxy by URL http://test.frps.com
subdomain = web01
custom_domains = web02.yourdomain.com
# locations is only available for http type
locations = /,/pic
host_header_rewrite = example.com
# params with prefix "header_" will be used to update http request headers
header_X-From-Where = frp
health_check_type = http
# frpc will send a GET http request '/status' to local http service
# http service is alive when it return 2xx http response code
health_check_url = /status
health_check_interval_s = 10
health_check_max_failed = 3
health_check_timeout_s = 3

[web02]
type = https
local_ip = 127.0.0.1
local_port = 8000
use_encryption = false
use_compression = false
subdomain = web01
custom_domains = web02.yourdomain.com
# if not empty, frpc will use proxy protocol to transfer connection info to your local service
# v1 or v2 or empty
proxy_protocol_version = v2

[plugin_unix_domain_socket]
type = tcp
remote_port = 6003
# if plugin is defined, local_ip and local_port is useless
# plugin will handle connections got from frps
plugin = unix_domain_socket
# params with prefix "plugin_" that plugin needed
plugin_unix_path = /var/run/docker.sock

[plugin_http_proxy]
type = tcp
remote_port = 6004
plugin = http_proxy
plugin_http_user = abc
plugin_http_passwd = abc

[plugin_socks5]
type = tcp
remote_port = 6005
plugin = socks5
plugin_user = abc
plugin_passwd = abc

[plugin_static_file]
type = tcp
remote_port = 6006
plugin = static_file
plugin_local_path = /var/www/blog
plugin_strip_prefix = static
plugin_http_user = abc
plugin_http_passwd = abc

[plugin_https2http]
type = https
custom_domains = test.yourdomain.com
plugin = https2http
plugin_local_addr = 127.0.0.1:80
plugin_crt_path = ./server.crt
plugin_key_path = ./server.key
plugin_host_header_rewrite = 127.0.0.1
plugin_header_X-From-Where = frp

[plugin_http2https]
type = http
custom_domains = test.yourdomain.com
plugin = http2https
plugin_local_addr = 127.0.0.1:443
plugin_host_header_rewrite = 127.0.0.1
plugin_header_X-From-Where = frp

[secret_tcp]
# If the type is secret tcp, remote_port is useless
# Who want to connect local port should deploy another frpc with stcp proxy and role is visitor
type = stcp
# sk used for authentication for visitors
sk = abcdefg
local_ip = 127.0.0.1
local_port = 22
use_encryption = false
use_compression = false

# user of frpc should be same in both stcp server and stcp visitor
[secret_tcp_visitor]
# frpc role visitor -> frps -> frpc role server
role = visitor
type = stcp
# the server name you want to visitor
server_name = secret_tcp
sk = abcdefg
# connect this address to visitor stcp server
bind_addr = 127.0.0.1
bind_port = 9000
use_encryption = false
use_compression = false

[p2p_tcp]
type = xtcp
sk = abcdefg
local_ip = 127.0.0.1
local_port = 22
use_encryption = false
use_compression = false

[p2p_tcp_visitor]
role = visitor
type = xtcp
server_name = p2p_tcp
sk = abcdefg
bind_addr = 127.0.0.1
bind_port = 9001
use_encryption = false
use_compression = false
```

修改完成后，保存。然后启动 frpc

我是Mac系统 所以执行 ./frpc -c ./frpc.ini

如果是Win的话，要用命令行进入到 客户端解压的文件夹，执行 frpc.exe -c frpc.ini

将 web.frp.auan.cn 的域名 A 记录解析到 IP 180.28.83.22（此处更改为你的服务器公网IP），如果服务器已经有对应的域名，也可以将 CNAME 记录解析到服务器原先的域名。

通过浏览器访问 [http://web.frp.auan.cn:9080](https://links.jianshu.com/go?to=http%3A%2F%2Fweb.frp.auan.cn%3A9080) 即可访问到处于内网机器上的 web 服务。

我们通过服务器控制台就可以看到有客户端已经连接了。



作者：Rafaelhuang
链接：https://www.jianshu.com/p/1272fbbecfeb/
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。