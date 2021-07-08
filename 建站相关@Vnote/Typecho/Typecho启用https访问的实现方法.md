# Typecho启用https访问的实现方法
登录Typecho后台 -> 设置 -> 基本设置 -> 站点地址改成https的域名是必须的。

编辑Typecho站点根目录下的文件config.inc.php加入下面一行配置，否则网站后台还是会调用HTTP资源。

```
/** 开启HTTPS支持 */
define('__TYPECHO_SECURE__',true);

```

由于Chrome浏览器对HTTPS要求较高，Firefox已经显示小绿锁，可是Chrome还是有警告提示，F12查看，

评论表单的action地址还是http，找到站点主题目录下的 comments.php 文件，

并搜索 $this->commentUrl()，将其替换为：echo str_replace("http","https",$this->commentUrl()); 最后保存。

来源：[](https://feixiang.eu.org/archives/7.html)[https://feixiang.eu.org/archives/7.html](https://feixiang.eu.org/archives/7.html)