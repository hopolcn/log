# Yandex搭配RainLoop，构建web邮件服务器系统
RainLoop用PHP开发，正常情况下在PHP环境中就可以运行，有时候并不需要数据库，应用程序不存储电子邮件，应用直接访问邮件服务器和RainLoop显示邮件而已，简单实用！性能方面完全取决于你的服务器内存和网速了。  
默认的SQLite数据库，无需特别配置，能对联系人很简单的支持。  
如果有大量的活跃用户,请不要选择SQLite数据库类型，建议启用MySQL数据库。

Rainloop支持IMAP和SMTP协议，因此我们可以用Rainloop来管理Yandex邮箱，你可以用Rainloop来收发Yandex邮箱。  
你可以通过绑定自己的二级域名来使用，避免了本地打开Yandex卡顿的问题。

RainLoop下载：[](http://www.rainloop.net/downloads/)[http://www.rainloop.net/downloads/](http://www.rainloop.net/downloads/)  
里面有2个选项，Community edition 和 Standard edition，建议选 Standard edition，毕竟个人使用还带在线更新。  
其实我两个都装了下，除了在线更新，没什么区别，下面是对比官方给的对比：  
Community edition就是社区版，可以用于个人或者商业，没有在线更新。  
Standard edition是标准版，没给钱就只能个人使用，并多了一个在线更新。

> 对系统的要求也不是很高，一般的虚拟主机或者小内存vps都能胜任。  
> Web server: Apache, NGINX, lighttpd or other with PHP support PHP: 5.4  
> and above PHP extensions: cURL, iconv, json, libxml, dom, openssl,  
> DateTime, PCRE, SPL Browser: Google Chrome, Firefox, Opera 10+, Safari  
> 3+, Internet Explorer 11 or EDGE Optional: PDO  
> (MySQL/PostgreSQL/SQLite) PHP extension (for contacts)

下载完成后，无需安装，直接解压到指定文件夹即可，你觉得太难还是上宝塔吧。  
然后通过安装位置+?admin登录，比如：我放在mail.test.com这个域名下，  
直接输入：mail.test.com/?admin即可访问。默认账户admin 默认密码12345。  
进去后第一件事就是改语言，再去改密码，如果你就去改不了东西，你可能是vps需要给文件权限。  
![RainLoop更改语言](images/20200220024001634_1472.php "RainLoop更改语言")  
最后在域名里面新加你添加域名，填写Yandex验证过的域名、IMAP、SMTP。  
![RainLoop添加域名设置](images/20200220023959521_13000.php "RainLoop添加域名设置")  
填写完毕后，点测试，当IMAP、SMTP字体由黑色变成绿色，就代表连接成功了。  
失败了，如果是VPS，你可以先ping一下这2个域名看通不通，如果是虚拟主机，放弃吧。

配置完成后，删掉地址?admin，直接输入完整的邮箱地址和密码就可以登录了。