# 刷新dns缓存命令

## 一、Linux 

重启nscd即可

```
/etc/init.d/nscd restart
```

较老的版本

```
/etc/rc.d/init.d/nscd restart
```

## 二、Windows 

  以管理员身份运行Cmd

```
ipconfig /flushdns
```

## 三、Mac OS X

  执行以命令

```
sudo killall -HUP mDNSResponder
```

  较老的版本

```
sudo dscacheutil -flushcache  |  lookupd -flushcache
```