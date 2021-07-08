## docker容器中查看容器linux版本

https://blog.csdn.net/weixin_44191019/article/details/108732524



docker容器中查看容器linux版本
有时候需要登陆容器搞点事情，这时候需要看容器系统的版本，那么一条命令就能完成。
正确的姿势：

```
cat /etc/issue
```

错误的姿势:
cat /proc/version 或 uname -a ，这样查到的是宿主机的系统。

原文：https://blog.csdn.net/c461522756/article/details/70822234