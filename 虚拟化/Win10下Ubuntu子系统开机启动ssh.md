# Win10下Ubuntu子系统开机启动ssh

## new=====================================================

2019年11月14日
更新系统后发现原来旧的方法不能用,原因是开机没启动.修改一下
在ubuntu中新建一个启动ssh的脚本/home/jk/start.sh

```
#!/bin/sh
service ssh start
exit
```

并且给脚本赋运行权限

```
chmod +x /home/jk/start.sh
```

接着编辑一下sudoers免得输密码

```
nano /etc/sudoers
```

添加以下这行到该文件

```
%sudo ALL=NOPASSWD: /home/jk/start.sh
```

到这里我们切回win10.
先在运行>命令(win+r)中输入shell:startup打开启动文件夹
新建一个文件startssh.bat,内容如下

```
bash -c "sudo /home/jk/start.sh"
```

到此结束

重启win10,过一会,就可以利用secureCRT直接连接刚才配置的127.0.0.1的WSL了

## 

## old=====================================================

在ubuntu中新建一个启动ssh的脚本/home/jk/start.sh

```
#!/bin/sh
service ssh start
$SHELL
```

并且给脚本赋运行权限

```
chmod +x /home/jk/start.sh
```

接着编辑一下sudoers免得输密码

```
nano /etc/sudoers
```

添加以下这行到该文件

```
%sudo ALL=NOPASSWD: /home/jk/start.sh
```

到这里我们切回win10.
先在运行>命令(win+r)中输入shell:startup打开启动文件夹
新建一个文件startssh.vbs,内容如下

```
Set ws = CreateObject("Wscript.Shell")
ws.run "bash -c 'sudo /home/jk/start.sh'",vbhide
```

到此结束

重启win10,过一会,就可以利用secureCRT直接连接刚才配置的127.0.0.1的WSL了