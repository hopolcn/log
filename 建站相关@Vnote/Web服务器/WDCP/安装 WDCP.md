# 安装 WDCP
https://wdcp.net/install.html

运行环境及安装说明
系统要求：全新安装的系统或最小安装(CentOS,RedHat,Ubuntu)

内存要求：最低128MB，推荐1024MB以上

硬盘要求：10G以上

其它说明：确保是纯净系统，即没有安装过其它环境及面板，可最小安装更佳

## 默认账户密码
admin
wdlinux.cn


## 下载安装
使用SSH终端工具登录系统

```
wget http://dl.wdlinux.cn/files/lanmp_v3.3.tar.gz
tar zxvf lanmp_v3.3.tar.gz
sh lanmp.sh
```


## 可加参数cus来选择版本自定义安装,如下
`sh lanmp.sh cus`
 

## 多版本PHP支持，可选安装，安装后可在后台切换所使用的版本
`sh lib/phps.sh`


## tomcat安装，可选安装，默认版本为8.5
`sh lib/tomcat.sh`


## nodejs应用环境，可选安装,默认版本为v10.13
`sh lib/nodejs.sh`


## 升级说明
wdcp面板的升级直接在后台上点击操作即可升级

环境的升级，可根据需求使用脚本升级相应软件或版本

更多文档与说明，可看论坛 https://www.wdlinux.cn/bbs/forum-23-1.html