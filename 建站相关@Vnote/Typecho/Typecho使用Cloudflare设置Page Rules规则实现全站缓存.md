# Typecho使用Cloudflare设置Page Rules规则实现全站缓存
众所周知，免费版的Cloudflare可以设置3条Page Rules。  
我们要好好的利用这三条规则来给Typecho实现全站缓存的目的，  
当然这也是有弊端的，比如每次更新文章后，或者评论回复后，  
需要登录Cloudflare找到Caching，然后点击Purge Everything刷新全站缓存。  
特别是128m以内的小内存vps，设置缓存后，vps躲在Cloudflare后面相当的安全了。  
  
推荐不需要交互，和偶尔更新的Typecho博客使用，添加后访问效果显著。  
第一条：  
If the URL matches：`*sbblog.cn/admin*`  
`注意如果还需要用二级域名建站并不需要缓存，sbblog.cn前面的*号就要换成主域名`  
Security Level：High  
Cache Level：Bypass  
Disable Apps  
Disable Performance

第二条：  
If the URL matches：`*sbblog.cn/action*`  
`我看到过很多人，只添加了admin这个后缀，登录无限跳转的情况。`  
Security Level：High  
Cache Level:Bypass  
Disable Apps  
Disable Performance

第三条：  
If the URL matches：`*sbblog.cn/*`  
`全站缓存，包含所有二级三级域名。`  
Browser Cache TTL：a month  
Cache Level：Cache Everything  
Edge Cache TTL：a month