# Alpine Linux 全新安装 备份恢复 一键脚本及教程
之前博客介绍了OpenVZ构架下重装为Alpine Linux系统的方法，详情请参考：[Linux下OpenVZ平台Alpine Linux一键安装脚本](https://cikeblog.com/linux-to-apline.html "Linux下OpenVZ平台Alpine Linux一键安装脚本") 后面发现安装后，很多人可能需要重装或者数据备份到其他机器上，看到有人已经写出重装和备份脚本，所以在这里一并分享出来。

#### 全新安装：

为何要全新安装呢？举个例子，在Alpine Linux下，我误删了一些系统文件，并且无法恢复，但是SSH服务是正常的，或者说，SSH也挂了，但是可以通过VNC访问。

因为这些原因，我们就可以进机器去对系统进行完全重装为全新的Alpine Linux系统。

脚本：

wget  https://cikeblog.com/s/unalpine.sh && chmod 755 unalpine.sh &&./unalpine.sh

执行安装脚本后，会完全重装为全新系统。

#### 备份恢复：

备份恢复使用场景：在机器上，我们部署了很多程序，举个例子，安装了supervisord，并且配置了需要的监听程序，已经设置开机启动，那如果去其他机器上，还是得照样子写一次，显得非常麻烦，所以我们使用此脚本，即可自动对系统进行打包，去其他机器上执行恢复脚本即可。

先执行系统备份：

cd /
tar czf all.tar.gz * --exclude dev --exclude proc --exclude sys

备份后，会在根目录下出现all.tar.gz文件，把此文件拷贝到其他机器即可，可以利用scp，resyn，winscp等工具。

脚本：

wget  https://cikeblog.com/s/restorealpine.sh && chmod 755 restorealpine.sh

把all.tar.gz文件上传备份到服务器，比如/root/all.tar.gz，执行脚本：

./restore_alpine.sh /root/all.tar.gz

即可恢复成备份的系统。

如果执行过程中，wget脚本出现：wget: error getting response: Connection reset by peer

请参考：[Alpine Linux出现wget: error getting response: Connection reset by peer 解决办法](https://cikeblog.com/alpine-linux-wget-error-getting-response-connection-reset-by-pee.html "Alpine Linux出现wget: error getting response: Connection reset by peer 解决办法")

部分内容参考于此文章：[http://sent.iytc.net/blog/index.php/archives/4.html ](https://cikeblog.com/goto/n0nd)