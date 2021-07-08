# proxmox nginx反向代理配置2019-09-16

```php
server
{
    listen 80;
    listen 443 ssl http2;
    server_name www.xxx.com 10.5.1.209:8006;
    index index.php index.html index.htm default.php default.htm default.html;
    root /www/wwwroot/www.xxx.com;
    
    #SSL-START SSL相关配置，请勿删除或修改下一行带注释的404规则
    #error_page 404/404.html;
    ssl_certificate    /etc/letsencrypt/live/www.xxx.com/fullchain.pem;
    ssl_certificate_key    /etc/letsencrypt/live/www.xxx.com/privkey.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    error_page 497  https://$host$request_uri;


    #SSL-END
    #ssl on;
     location / {
        proxy_pass                https://10.5.1.209:8006;
        proxy_set_header          X-Real-IP $remote_addr;
        proxy_set_header          X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header          Upgrade $http_upgrade;
        #client_max_body_size      1m;
        #proxy_redirect               on;
        #proxy_set_header         Host        $host;
        #proxy_set_header         X-Forwarded-Proto https;
        proxy_ssl_verify          off;
        proxy_ssl_session_reuse   on;
        #proxy_http_version       1.1;
        #proxy_set_header referer  https://10.5.1.209:8006/;
     }
     
  
    #ERROR-PAGE-START  错误页配置，可以注释、删除或修改
    error_page 404 /404.html;
    error_page 502 /502.html;
    #ERROR-PAGE-END
    
    #PHP-INFO-START  PHP引用配置，可以注释或修改
    include enable-php-56.conf;
    #PHP-INFO-END
    
    #REWRITE-START URL重写规则引用,修改后将导致面板设置的伪静态规则失效
    include /www/server/panel/vhost/rewrite/www.xxx.com.conf;
    #REWRITE-END
    
    #禁止访问的文件或目录
    location ~ ^/(\.user.ini|\.htaccess|\.git|\.svn|\.project|LICENSE|README.md)
    {
        return 404;
    }
    
    #一键申请SSL证书验证目录相关设置
    location ~ \.well-known{
        allow all;
    }
    
    #location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    #{
    #    expires      30d;
    #    error_log off;
    #    access_log off;
    #}
    
    
    access_log  /www/wwwlogs/www.xxx.com.log;
    error_log  /www/wwwlogs/www.xxx.com.error.log;
}
```