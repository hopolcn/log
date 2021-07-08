# 申请新版Yandex免费域名邮箱
时隔多年，用了很多国内国外的免费域名邮箱，还是又换回来了，整体感觉还是Yandex更强大点。  
打开申请的界面，一脸懵逼，然后百度了下，网上大部分教程都是以前写的，配图都很老旧了。

下面简单介绍下Yandex域名邮箱的优势：  
可创建1000邮箱用户，每个用户10G的容量，支持 POP3、IMAP、SMTP，送达率高不易进垃圾箱，支持API。  
最亮点的是尽量支持很多收费企业邮箱才支持的Catch-all address routing。  
这个功能简单理解就是把abc@xxx.xxx，不管你abc前面是什么名称，不管在系统是否存在，都可以指定一个用户，系统会把不存在的用户收到的邮件全部转发到指定邮箱里面。拿来注册各种服务，不同的网站用不同的前缀，谁泄露了你的信息，一目了然。  
注册地址：[](https://mail.yandex.com/)[https://mail.yandex.com/](https://mail.yandex.com/)  
点击首页黄色的按钮：Create an account，注册账户。  
![Yandex注册](images/20200220023814585_27975.php "Yandex注册")  
注册完毕后，请登录：[](https://connect.yandex.com/pdd/)[https://connect.yandex.com/pdd/](https://connect.yandex.com/pdd/)  
![Yandex添加域名](images/20200220023810671_13903.php "Yandex添加域名")  
填写你的域名，点击Sign up free继续，就会进入Yandex Connect的服务页面[](https://connect.yandex.com/portal/services/webmaster)[https://connect.yandex.com/portal/services/webmaster](https://connect.yandex.com/portal/services/webmaster)  
![Yandex域名添加成功](images/20200220023809536_13406.php "Yandex域名添加成功")  
点击域名下面的Confirm your domain，开始验证域名所有权。  
目前，支持四种验证方式。  
DNS：在域名管理页面将文本框中的内容添加为TXT Record记录，最长需要72h才能完成验证。（其实一般几分钟就能完成）  
HTML File：在网站根目录下创建yandex_*.html文件 并将文本框中的代码复制进去，FTP权限就可以上传它。  
Meta Tag：在域名主页网站的<head>标签内增加Yandex所提供的代码行，只要能进后台编辑模板就可以加。  
WHOIS：不推荐这种，现在很多域名都强制开启隐私保护了  
![Yandex验证域名所属权](images/20200220023806222_13885.php "Yandex验证域名所属权")  
选择你觉得方便的一种，按照要求操作完成后，点一下Check，验证下。  
验证完成后，最后只需要配置CNAME, MX, SPF, DKIM记录，即可使用。  
配置DNS的界面只有CNAME、MX、SPF，DKIM需要在[DKIM signatures](https://connect.yandex.com/portal/admin/customization/mail)获取。  
特别注意：DKIM的主机名是：mail._domainkey，类型为：TXT。  
![YandexDNS修改后](images/20200220023804003_19525.php "YandexDNS修改后")  
最后，添加一个可用邮箱地址，[](https://connect.yandex.com/portal/admin)[https://connect.yandex.com/portal/admin](https://connect.yandex.com/portal/admin)  
下面，点一下add，选第一个Add a person.  
![Yandex创建用户](images/20200220023802392_16444.php "Yandex创建用户")

解析没生效前，你可以直接通过 [https://mail.yandex.com/?pdd_domain=test.com](https://mail.yandex.com/?pdd_domain=test.com)（把test.com换成你的）登录。  
新建账户必须登录一次后，同意一个协议，才能使用SMTP和IMAP收发信。

如果你需要Catch-all address routing，可以通过[](https://connect.yandex.com/portal/services/mail)[https://connect.yandex.com/portal/services/mail](https://connect.yandex.com/portal/services/mail)开启。  
选好用户，然后Save即可。

> POP3：pop.yandex.com 开启SSL 端口 995  
> SMTP：smtp.yandex.com 开启SSL 端口 465  
> IMAP：imap.yandex.com 开启SSL 端口 993