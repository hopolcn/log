## 玩客云刷OP无障碍内网穿透和简易NAS小白操作

https://www.right.com.cn/forum/thread-4082815-1-1.html



目前很多操作都很复杂，这边介绍一个小白也可以操作的方法
工具介绍
易有云：详情可以查看 https://www.ddnsto.com/linkease/#/zh-cn/README
   手机相片越来越多，而公有云盘越来越慢
   私有云存储搭建太专业，外网访问设置很麻烦
   用 webdav、samba 等，有没有个好的全平台一致体验的客户端
   数据备份、容灾是个大问题，设置太专业，还容易设置错误造成数据丢失
   公司写的文档不方便同步到家里 
ddnsto：一个小白都可以操作的内网穿透，需要微信登陆确保安全 https://www.ddnsto.com/zh/guide/koolshare_merlin.html#获取token
    无需公网ip，不被网络环境限制。
    无需购买域名或服务器，省去了服务器年费和带宽要求以及域名购买、备案等等繁琐操作。
    全部的安装、配置都在浏览器完成，不需敲一行代码，对小白用户非常友好。
    支持http2，访问家庭内部网络速度更快！
我用玩客云刷了OP后的操作步骤：
    SSH ROOT 登陆玩客云:
        

#### 本帖隐藏的内容

安装易有云：

1. sh -c "$(curl -sSL http://firmware.koolshare.cn/binary/LinkEase/Openwrt/install_linkease.sh)"

*复制代码*

![img](images/forum.php)


          安装DDNSTO：

1. sh -c "$(curl -sSL http://firmware.koolshare.cn/binary/ddnsto/openwrt/install_ddnsto.sh)"

*复制代码*

![img](images/forum.php)


 安装结束后，刷新op管理界面，可以在服务里看到安装的新安装的
      

![img](images/forum.php)




   其他设备安装可以参照官网说明进行安装（威联通 群辉等设备都可以直接安装，非常简单）
   易有云
      其他设备客服端下载 https://www.ddnsto.com/linkease/download/#/
      其他设备服务端下载 https://www.ddnsto.com/linkease/download/#/disk
   DDNSTO https://www.ddnsto.com/zh/guide/koolshare_merlin.html#获取token 

  目前可以免费体验，过7天可以重新激活一下，如需稳定使用可以购买，费用也不贵，威联通和群辉等设备高手肯定有很多办法，但是还是有很多小白用户， 本人也是小白所以体验后不错来分享一下
DDNSTO: 

![img](images/forum.php)


  易有云： 

![img](images/forum.php)