# Typecho 如何自定义附件上传目录
在 config.inc.php 里新增一行下列代码即可，其中 your\_upload\_dir 是你要上传的目录：

```
define('__TYPECHO_UPLOAD_DIR__', 'your_upload_dir');

```

注意，这个your\_upload\_dir应该是绝对地址，你实在不知道，可以上传一个PHP探针可以看到探针的路径。

最后还是建议，小博客，图片外链，然后文件存储在第三方云盘上下载，慢点但是不容易丢，而且搬家方便。