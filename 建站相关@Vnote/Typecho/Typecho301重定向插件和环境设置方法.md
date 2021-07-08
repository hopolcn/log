# Typecho301重定向插件和环境设置方法
绑定域名后有这样一个困惑，主域一个域名有带www或者不带www的，不管你选带www还是不带，  
建议把另外一个重定向到常用的，不一定带www就权重高，根据自己喜好调整即可，一旦确定，尽量不要改变。  
Typecho有插件可以实现301重定向，如果你不想根据环境来添加规则，用插件实现的方法最简单。  
Typecho官方Docs里面有下载地址，你可以直接下载它，上传到/var/plugins/即可，然后在控制台激活插件，进行设置。

下载地址：[](http://docs.typecho.org/_media/plugins/redirect301_v1.0.0.zip)[http://docs.typecho.org/\_media/plugins/redirect301\_v1.0.0.zip](http://docs.typecho.org/_media/plugins/redirect301_v1.0.0.zip)  
来源：[](http://docs.typecho.org/plugins/download)[http://docs.typecho.org/plugins/download](http://docs.typecho.org/plugins/download)

***

如果你不喜欢插件，在环境里面加规则也可以实现。  
不同环境的方法也不一样，参考下这篇文章：[](https://www.sbblog.cn/archives/6.html)[https://www.sbblog.cn/archives/6.html](https://www.sbblog.cn/archives/6.html)  
Linux Apache 环境：在</IfModule>之前加，简单来说就是不带www跳转到www

```
RewriteCond %{HTTP} !=on
RewriteRule ^(.*) http://www.sbblog.cn/$1 [L,R=301]

```

Linux Nginx环境：加在server里面，也是不带www跳转到www  
if ($host ~* sbblog.cn) {

```
rewrite ^/(.*)$ http://www.sbblog.cn/$1 permanent; 

```

IIS Windows：  
看图：[](https://jingyan.baidu.com/article/3a2f7c2e3b7f3726aed61155.html)[https://jingyan.baidu.com/article/3a2f7c2e3b7f3726aed61155.html](https://jingyan.baidu.com/article/3a2f7c2e3b7f3726aed61155.html)

Linux Lighttpd环境：

```
$HTTP["host"] =~ "(sbblog.cn)" {  
    url.redirect = ( "^/(.*)" => "http://www.sbblog.cn/$1" )  
}

```

Linux Caddy环境：

```
sbblog.cn {
  redir http://www.moerats.com{url}
}

```

***

Cloudflare 通过 Page Rules 也可以实现，搜索：**Cloudflare 301** 即可