# Typecho不同环境伪静态的支持
这里面收录了大部分规则，所有信息来自互联网，后面我会标明出处。  
开启方法：进入后台–》设置–》永久链接–》强制启用地址重写，除了Apache有几率成功，其他环境都需要手动添加。  
2019年03月10日 目前收录了Apache、Nginx、IIS、Lighttpd、Caddy。

Linux Apache 环境 (.htaccess)：  
理论上主机可以直接通过后台设置，自动创建，如果无法自动创建，请先上传一个空白的1.txt到主机，再重命名为.htaccess，然后覆盖下面的规则。

```
<IfModule mod_rewrite.c>
RewriteEngine On
# 下面是在根目录，文件夹要修改路径
RewriteBase /
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ /index.php/$1 [L]
</IfModule>

```

Linux Nginx环境：  
在nginx.conf里面server 里面添加如下

```
if (!-e $request_filename) {
            rewrite ^(.*)$ /index.php$1 last;
        }

```

Windows IIS 伪静态 (httpd.ini)：  
修改网站根目录的httpd.ini，然后添加如下内容。

```
[ISAPI_Rewrite]
# 3600 = 1 hour
CacheClockRate 3600
RepeatLimit 32
# 中文tag解决
RewriteRule /tag/(.*) /index\.php\?tag=$1
# sitemapxml
RewriteRule /sitemap.xml /sitemap.xml [L]
RewriteRule /favicon.ico /favicon.ico [L]
# 内容页
RewriteRule /(.*).html /index.php/$1.html [L]
# 评论
RewriteRule /(.*)/comment /index.php/$1/comment [L]
# 分类页
RewriteRule /category/(.*) /index.php/category/$1 [L]
# 分页
RewriteRule /page/(.*) /index.php/page/$1 [L]
# 搜索页
RewriteRule /search/(.*) /index.php/search/$1 [L]
# feed
RewriteRule /feed/(.*) /index.php/feed/$1 [L]
# 日期归档
RewriteRule /2(.*) /index.php/2$1 [L]
# 上传图片等
RewriteRule /action(.*) /index.php/action$1 [L]

```

Linux Lighttpd环境  
在 /etc/lighttpd/lighttpd.conf 下新增

```
url.rewrite = (
"^/(admin|usr)/(.*)"  => "/$1/$2",
"^/(.*).html$" => "/index.php/$1.html",
"^/archives/(.*)" => "/index.php/archives/$1",
"^/category/(.*)" => "/index.php/category/$1",
"^/([0-9]+)/([0-9]+)/$" => "/index.php/$1/$2/",
"^/tag/(.*)/$" => "/index.php/tag/$1",
"^/search/(.*)/$" => "/index.php/search/$1",
"^/(.*)page/(.*)" => "/index.php/$1page/$2",
"^/(feed.*)" => "/index.php/$1",
"^/action/(.*)" => "/index.php/action/$1",
"^/(.*)comment" => "/index.php/$1/comment"
)

```

Linux Caddy环境  
在括号内，添加下面的规则

```
rewrite / {
                if {path} not_match (/usr/|/admin/)
                to /index.php{uri}
        }

```

引用：[](http://docs.typecho.org/servers)[http://docs.typecho.org/servers](http://docs.typecho.org/servers)  
[](https://qqdie.com/archives/typecho-open-pseudo-static-get-rid-of-that-pesky-index-the-php.html)[https://qqdie.com/archives/typecho-open-pseudo-static-get-rid-of-that-pesky-index-the-php.html](https://qqdie.com/archives/typecho-open-pseudo-static-get-rid-of-that-pesky-index-the-php.html)  
[](http://www.zzfly.net/lighttpd-typecho-rewirte/)[http://www.zzfly.net/lighttpd-typecho-rewirte/](http://www.zzfly.net/lighttpd-typecho-rewirte/)