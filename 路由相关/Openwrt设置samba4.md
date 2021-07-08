# Openwrt设置samba4

1.先进web,添加好要共享的目录
![请输入图片描述](images/samba4共享.png)
![请输入图片描述](images/1.png)
2.进入ssh命令行模式输入以下命令设置root的smb密码
smbpasswd -a root
3.重启下smaba4服务
service samba4 restart
4.设置好后,就可以用root和设置的smb密码访问强制root的目录比如/
![请输入图片描述](images/2.png)