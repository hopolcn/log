# alpine装酸酸奶教程
刚接触alpine，发现真挺小巧的。装了一堆软件了，硬盘占用才100M，内存占用10M。  
命令也上手，apk search, apk add 就可以找软件安装了，各种版本都是很新很高的版本，足够使用。  
当然，市面上的各种一键脚本几乎都不能用，毕竟命令都不一样。  
当然，装啥软件也要先装个酸酸奶，官方没有SS打包，所以还需要手动安装，  
  
所以，优先当然考虑是Go版本的酸酸奶，gayhub找了下，我找的是这个 https://github.com/riobard/go-shadowsocks2  
顺便问下，Go版本还有那个好用的。  
  
1、下载：wget https://github.com/riobard/go-shadowsocks2/releases/download/v0.1.0/shadowsocks2-linux.gz  
  
解压缩并复制到 /usr/sbin/ 共用目录，我把它改名成 ss-go，你不改也可以， chmod +x 给运行权限  
  
  
2、测试，用官方给的命令格式测试，看成功没，加密我这选择了aes-256-gcm，端口选了8080，这些可以自己改，密码就自己设置，然后打开**不可描述**之网站测试  
  
  

1. ss-go -s ss://aes-256-gcm:your-password@:8080 -verbose

*复制代码*

  
  
  
3、测试成功没问题，直接放后台运行就可以了  

1. nohup ss-go -s ss://aes-256-gcm:your-password@:8080 -verbose &

*复制代码*

  
  
4、剩下就是弄成自启，不然不方便。  
  
nano /etc/local.d/ss-go.start  添加下面的内容，  
  

1. #!/bin/sh  
    
2.   
    
3. nohup ss-go -s ss://aes-256-gcm:your-password@:8080 -verbose &

*复制代码*

  
  
同样要记得给权限 chmod +x /etc/local.d/ss-go.start , 最后， rc-update add local  就完成，reboot 检查一下  
  
Note: alpine 新装是裸系统，所以如果你是新装的，需要先

1. apk add nano wget openssl

*复制代码*

  
  
PS：随便写的，将就看吧。