# Typecho开启Cloudflare切换 IP 来源获取
如果你的网站使用了 CloudFlare 一类的 CDN 服务使得部分插件无法正常记录用户 IP 地址的话，  
可以在 config.inc.php 声明这个静态变量，替换成服务商对应的用户 IP 头就可以了！  
CloudFlare 现在默认提供的是 HTTP\_X\_FORWARDED_FOR 头传送用户真实 IP 地址，所以我直接填入头的名称就可以了！

```
define('__TYPECHO_IP_SOURCE__', 'HTTP_X_FORWARDED_FOR');

```

来源：[https://paugram.com/coding/typecho-secret-usage.html](https://paugram.com/coding/typecho-secret-usage.html)