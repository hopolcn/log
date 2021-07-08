## sish：一个开源、轻量且类似于Servo/Ngrok的内网穿透工具

https://www.moerats.com/archives/1002/



**说明：**`sish`是一个`SSH`服务器，仅用于远程端口转发，可以快速将本地端口暴露在外网，作者声称其为`Servo`/`Ngrok`替代方案，仅使用`SSH`的`HTTP(S)`、`WS(S)`、`TCP`隧道连接到他们的`localhost`服务器，该工具和[Servo](https://www.moerats.com/archives/990/)差不多一样，不同就是`Servo`官方提供了免费的`SSH`客户端，而`sish`作者提供的客户端貌似因为滥用关闭了，所以就需要我们自己搭建了，这里就水下`Docker`和手动安装。

## Docker安装

**Github地址：**https://github.com/antoniomika/sish

**1、安装Docker**

```
#CentOS 6
rpm -iUvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
yum update -y
yum -y install docker-io
service docker start
chkconfig docker on

#CentOS 7、Debian、Ubuntu
curl -sSL https://get.docker.com/ | sh
systemctl start docker
systemctl enable docker
```

**2、拉取镜像**
这里由于直接使用`ip`的话，只能用于转发`TCP`，`HTTP(S)`等就需要配置下域名了，所以以下全部默认使用域名。

先解析一个主/泛域名到服务器`ip`，比如解析`moerats.com`、`*.moerats.com`到服务器`ip`。

然后再参考下面的参数详解，再自行修改部分参数后，使用命令：

```
#配置http域名
docker run -d --name sish \
  --restart=always \
  -v ~/sish/keys:/keys \
  -v ~/sish/pubkeys:/pubkeys \
  --net=host antoniomika/sish \
  -sish.addr=:3333 \
  -sish.http=:80 \
  -sish.keysdir=/pubkeys \
  -sish.pkloc=/keys/ssh_key \
  -sish.forcerandomsubdomain=false \
  -sish.domain moerats.com \
  -sish.bindrandom=false \
  -sish.redirectrootlocation https://www.baidu.com 

#配置https域名，这里需要提供泛域名证书
docker run -d --name sish \
  --restart=always \
  -v ~/sish/ssl:/ssl \
  -v ~/sish/keys:/keys \
  -v ~/sish/pubkeys:/pubkeys \
  --net=host antoniomika/sish \
  -sish.addr=:3333 \
  -sish.https=:443 \
  -sish.http=:80 \
  -sish.httpsenabled=true \
  -sish.httpspems=/ssl \
  -sish.keysdir=/pubkeys \
  -sish.pkloc=/keys/ssh_key \
  -sish.forcerandomsubdomain=false \
  -sish.domain moerats.com \
  -sish.bindrandom=false \
  -sish.redirectrootlocation https://www.baidu.com
```

部分参数如下：

```
-sish.addr=:3333  #ssh监听地址
-sish.forcerandomsubdomain=false  #是否强制随机子域，这个建议关掉
-sish.bindrandom=false  #是否随机绑定端口，这个建议关掉
-sish.domain moerats.com  #使用的域名
-sish.redirectrootlocation https://www.baidu.com  #主域名(-sish.domain参数)强制跳转到该地址
-sish.httpspems=/ssl  #泛域名SSL证书路径，存放路径~/sish/ssl，证书命名格式fullchain.pem和privkey.pem
```

其他参数默认即可，也可以自行添加或修改其它参数。

全部参数如下：

```
Usage of sish:
  -sish.addr string
        The address to listen for SSH connections (default "localhost:2222")
  -sish.auth
        Whether or not to require auth on the SSH service
  -sish.bannedcountries string
        A comma separated list of banned countries
  -sish.bannedips string
        A comma separated list of banned ips
  -sish.bannedsubdomains string
        A comma separated list of banned subdomains (default "localhost")
  -sish.bindrandom
        Bind ports randomly (OS chooses) (default true)
  -sish.bindrange string
        Ports that are allowed to be bound (default "0,1024-65535")
  -sish.cleanupunbound
        Whether or not to cleanup unbound (forwarded) SSH connections (default true)
  -sish.debug
        Whether or not to print debug information
  -sish.domain string
        The domain for HTTP(S) multiplexing (default "ssi.sh")
  -sish.forcerandomsubdomain
        Whether or not to force a random subdomain (default true)
  -sish.http string
        The address to listen for HTTP connections (default "localhost:80")
  -sish.httpport int
        The port for HTTP connections. This is only for output messages (default 80)
  -sish.https string
        The address to listen for HTTPS connections (default "localhost:443")
  -sish.httpsenabled
        Whether or not to listen for HTTPS connections
  -sish.httpspems string
        The location of pem files for HTTPS (fullchain.pem and privkey.pem) (default "ssl/")
  -sish.httpsport int
        The port for HTTPS connections. This is only for output messages (default 443)
  -sish.keysdir string
        Directory for public keys for pubkey auth (default "pubkeys/")
  -sish.password string
        Password to use for password auth (default "S3Cr3tP4$$W0rD")
  -sish.pkloc string
        SSH server private key (default "keys/ssh_key")
  -sish.pkpass string
        Passphrase to use for the server private key (default "S3Cr3tP4$$phrAsE")
  -sish.proxyprotoenabled
        Whether or not to enable the use of the proxy protocol
  -sish.proxyprotoversion string
        What version of the proxy protocol to use. Can either be 1, 2, or userdefined. If userdefined, the user needs to add a command to SSH called proxy:version (ie proxy:1) (default "1")
  -sish.redirectroot
        Whether or not to redirect the root domain (default true)
  -sish.redirectrootlocation string
        Where to redirect the root domain to (default "https://github.com/antoniomika/sish")
  -sish.subdomainlen int
        The length of the random subdomain to generate (default 3)
  -sish.usegeodb
        Whether or not to use the maxmind geodb
  -sish.verifyorigin
        Whether or not to verify origin on websocket connection (default true)
  -sish.verifyssl
        Whether or not to verify SSL on proxy connection (default true)
  -sish.whitelistedcountries string
        A comma separated list of whitelisted countries
  -sish.whitelistedips string
        A comma separated list of whitelisted ips
```

看不懂的，可以使用下谷歌翻译。

最后`CentOS`系统建议关闭防火墙使用，或者打开部分端口也行，关闭命令：

```
#CentOS 6系统
service iptables stop
chkconfig iptables off

#CentOS 7系统
systemctl stop firewalld
systemctl disable firewalld
```

像阿里云等服务器，还需要去安全组那里开放下端口。

## 手动安装

`Docker`虽然方便很多，但也有人会喜欢手动安装，这里作者没直接给出二进制文件，所以就需要我们手动来构建二进制文件了。

**1、安装Go**
这里由于需要新版的`Go`环境，所以这里就使用`Go`二进制包安装环境，下载地址→[传送门](https://golang.org/dl/)。

然后根据自己的服务器架构下载对应的最新安装包，一般可以直接使用命令：

```
#32位系统下载
wget -O go.tar.gz https://dl.google.com/go/go1.13.3.linux-386.tar.gz
#64位系统下载
wget -O go.tar.gz https://dl.google.com/go/go1.13.3.linux-amd64.tar.gz

#解压压缩包
tar -zxvf go.tar.gz -C /usr/local
#设置环境变量，将以下一起复制进ssh客户端运行
mkdir $HOME/go
echo 'export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin' >> /etc/profile
source /etc/profile
#查看go版本，有输出即为安装成功
go version
```

**2、安装sish**

```
#下载源码到主目录
git clone https://github.com/antoniomika/sish
cd sish
#编译二进制文件
go install
```

这里提示`-bash: git: command not found`的，可以先使用命令：

```
#CentOS
yum -y install git

#Debian、Ubuntu
apt install git -y
```

**3、运行sish**
运行参数这里就不贴了，直接参考上面`Docker`安装最下面的全部参数就行了。

先解析一个主/泛域名到服务器`ip`，比如解析`moerats.com`、`*.moerats.com`到服务器`ip`。

这里就贴个大概需要使用的参数，其它的根据需求自行修改，使用命令：

```
#配置http域名
sish -sish.addr=:3333 -sish.http=:80 -sish.domain moerats.com -sish.forcerandomsubdomain=false -sish.bindrandom=false -sish.redirectrootlocation https://www.moerats.com -sish.keysdir=/sish/pubkeys -sish.pkloc=/sish/keys/ssh_key 

#配置https域名
sish -sish.addr=:3333 -sish.https=:443 -sish.http=:80 -sish.domain moerats.com -sish.forcerandomsubdomain=false -sish.bindrandom=false -sish.httpsenabled=true -sish.redirectrootlocation https://www.moerats.com -sish.keysdir=/sish/pubkeys -sish.pkloc=/sish/keys/ssh_key -sish.httpspems=/sish/ssl
```

部分参数详解：

```
-sish.addr=:3333  #ssh监听地址，这里为3333
-sish.forcerandomsubdomain=false  #是否强制随机子域，这个建议关掉
-sish.bindrandom=false  #是否随机绑定端口，这个建议关掉
-sish.domain moerats.com  #使用的域名
-sish.redirectrootlocation https://www.baidu.com  #主域名(-sish.domain参数)强制跳转到该地址
-sish.httpspems=/sish/ssl  #泛域名SSL证书存放路径，证书命名格式fullchain.pem和privkey.pem
-sish.keysdir=/sish/pubkeys  #pubkey auth的公共密钥存放文件夹
-sish.pkloc=/sish/keys/ssh_key  #SSH服务器私钥
```

这里`/sish/ssl`、`/sish/pubkeys`、`/sish/keys`目录需要自己提前创建下，使用命令：

```
mkdir -p /sish/ssl /sish/pubkeys /sish/keys
```

**4、开机自启**
如果你使用手动命令没问题了，先使用`Ctrl+C`断开命令。

再新建`systemd`配置文件，适用`CentOS 7`、`Debian 8+`、`Ubuntu 16+`。

```
#修改成你手动运行命令的全部参数
command="-sish.addr=:3333 -sish.http=:80 -sish.domain moerats.com -sish.forcerandomsubdomain=false -sish.bindrandom=false -sish.redirectrootlocation https://www.moerats.com -sish.keysdir=/sish/pubkeys -sish.pkloc=/sish/keys"
#将以下代码一起复制到SSH运行
cat > /etc/systemd/system/sish.service <<EOF
[Unit]
Description=sish
After=network.target

[Service]
Type=simple
ExecStart=$(command -v sish) ${command}
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

启动并设置开机自启：

```
systemctl start sish
systemctl enable sish
```

最后`CentOS`系统建议关闭防火墙使用，或者打开部分端口也行，关闭命令：

```
#CentOS 6系统
service iptables stop
chkconfig iptables off

#CentOS 7系统
systemctl stop firewalld
systemctl disable firewalld
```

像阿里云等服务器，还需要去安全组那里开放下端口。

## 使用

使用要求：可以使用`SSH`，并且能连接到互联网，`Linux`、`Windows`等系统都行。

以下所使用的的`moerats.com`为上面配置好的客户端域名地址，自行修改成自己的即可。

**1、转发HTTP(S)**
将本地`3000`端口穿透到公网中，使用命令：

```
#要转发其它端口的自行替换
ssh -p 3333 -R 80:localhost:3000 moerats.com
```

第一次如果有提示，选择`yes`即可，之后会为你随机生成一个`moerats.com`的二级域名，然后就可以使用浏览器间接访问本地的`localhost:3000`了。

如果要指定二级域名，可以使用命令：

```
#这里默认为no1.moerats.com，自行替换即可
ssh -p 3333 -R no1:80:localhost:3000 moerats.com
```

此时你就可以在外网使用`no1.moerats.com`访问你本地的`localhost:3000`了。

**2、转发TCP**
将本地`6789`端口穿透到公网的`9876`端口中，使用命令：

```
#可以自行设置公网端口，这里默认6789，如果你要转发SSH端口，那就改成你的SSH端口
ssh -p 3333 -R 9876:localhost:6789 moerats.com
```

这里只说了下简单用法，客户端我们还可以设置国家/地区、`IP`白名单等，使用参考→[传送门](https://github.com/antoniomika/sish#whitelisting-ips)。

最后没有泛域名证书的，可以查看该教程自己申请→[传送门](https://www.moerats.com/archives/900/)，或者等博主发码子→[传送门](https://www.moerats.com/archives/996/)。