## 自己动手完成Freenom动态域名解析

http://ocdman.github.io/2018/08/10/%E8%87%AA%E5%B7%B1%E5%8A%A8%E6%89%8B%E5%AE%8C%E6%88%90freenom%E5%8A%A8%E6%80%81%E5%9F%9F%E5%90%8D%E8%A7%A3%E6%9E%90/



freenom.com是一个免费的顶级域名注册商，包括了像.tk，.ml，.ga等域名。但是除了网站以外，freenom并没有提供更新DNS记录的API接口供开发者使用。因此，我到网上参考很多关于实现freenom更新DNS记录的代码。

由于freenom加入了token验证机制，导致以前一部分代码无法完成登录功能。我个人比较推荐[freenom-dns](https://github.com/dabendan2/freenom-dns)这个NodeJS项目，不仅功能完备，命令行的交互也很友好。

之前说过默认的光猫没有DDNS服务，为了让树莓派能调用一个DDNS服务，绑定动态外网IP和域名，我也是煞费苦心。要说为什么不选择dnspod，一来懒得再注册一个号，二来貌似freenom的延迟也还可以，虽然没有dnspod表现得好，总体还能接受。

### 安装Node

既然是NodeJS项目，首先需要给树莓派安装NodeJS。首先查看一下自己的树莓派是ARM V6还是V7的CPU架构。

```
uname -a
```

我的是`armv6l`，说明我是ARM V6架构的CPU，所以下载ARM V6版本的NodeJS。

```
wget https://nodejs.org/dist/v8.11.3/node-v8.11.3-linux-armv6l.tar.xz
```

我下的是稳定版本，不会频繁更新。

解压文件

```
tar -xvf node-v8.11.3-linux-armv6l.tar.xz
```

验证node是否能正常运行

```
cd node-v8.11.3-linux-armv6l/bin/
./node -v
v8.11.3
```

node运行正常，下面看看npm命令

```
./npm -v
/usr/bin/env: node: No such file or directory
```

输入以下命令即可解决

```
mv node-v8.11.3-linux-armv6l/ /usr/local/node
sudo ln -s /usr/local/node/bin/node /usr/bin/node
sudo ln -s /usr/local/node/bin/npm /usr/bin/npm
```

现在是不是能正常运行node以及npm命令了呢？单独执行npm命令，可能会受到警告

> npm update check failed

这也没什么大不了的，只是说npm无法更新，并不影响正常使用。

### 下载freenom-dns程序

下面安装freenom-dns程序

```
npm install -g freenom-dns
```

安装完毕后，既可以通过命令来操作freenom上的网站。

```
// 登录
freenom-dns login
email: your@email.com
password: ********

//查看域名
freenom-dns list

//查看二级域名
freenom=dns list subdomain

//新增或更新DNS记录
freenom-dns set www.domain1.tk 2.3.4.5

//清除某条DNS记录
freenom-dns clear www.domain1.tk
```

### 调用freenom-dns的API

调用API之前，先执行命令

```
npm install --save freenom-dns
```

这行命令会下载freenom-dns依赖的包。通过freenom-dns这个项目提供的API，可以写一个js脚本，方便树莓派调用，创建一个freenom.com.ddns.js文件，内容如下

```
var user = "[your email]";
var pass = "[your password]";
var freenom = require("freenom-dns").init(user, pass);
var http = require('http');

http.get({'host': 'api.ipify.org', 'port': 80, 'path': '/'}, function(resp) {
  resp.on('data', function(ip) {
    console.log(new Date().toUTCString());
    console.log("My public IP address is: " + ip);
    freenom.dns.listRecords("[your domain]")
    .then(function(ret){
        console.log(ret);
        ret.forEach(function(record){
          if(record.name.toLowerCase() === 'www') {
            if(record.value == ip) {
              console.log("nothing to do.");
            } else {
              console.log("update DNS record.");
              freenom.dns.setRecord("[your.subdomain.xyz]", "A", ip)
              .then((ret) => console.log(ret))
              .catch((err) => {
                  console.log(err);
              });
            }
          }
        });
    })
    .catch((err) => {
        console.log(err);
    });
  });
});
```

然后创建一个freenom.com.ddns.sh的脚本文件

```
#!/bin/sh
node /home/pi/freenom.com.ddns.js >>/home/pi/log.txt 2>&1
```

日志存储在log.txt文件中。最后使用crontab计划任务，每小时执行一次freenom.com.ddns.sh

```
crontab -e

// 新增一条每个整点检查ddns的脚本
0 * * * * /bin/sh /home/pi/freenom.com.ddns.sh
```

### 总结

freenom不提供DNS更新API，确实不方便。我的树莓派上会定时检查外网IP变化了没有，一旦发生变化，在下一个整点到来之时，更新DNS记录。这样的话，我就可以保持对家里网络的远程连接了。