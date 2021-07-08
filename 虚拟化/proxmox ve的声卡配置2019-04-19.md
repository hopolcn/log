# proxmox ve的声卡配置2019-04-19

## 一、安装alsautils



```csharp
sudo apt-get install alsa-utils
```

## 二、声卡设备查看



```undefined
aplay -l
```

![img](https:////upload-images.jianshu.io/upload_images/4171480-efb686a00c79b4ec.png?imageMogr2/auto-orient/strip|imageView2/2/w/488/format/webp)

aplay -l

## 三、配置默认声卡设备



```undefined
sudo vi /etc/asound.conf
```

添加如下内容



```bash
pcm.!default{
         type hw
         card 0
         device 0  #这里的设备号根据第二步显示的内容来设置如果要设置hdmi则改成上图的3或7
}
ctl.!default{
        type hw
        card 0
        device 0 #这里的设备号根据第二步显示的内容来设置如果要设置hdmi则改成上图的3或7
}
```



作者：龙天ivan
链接：https://www.jianshu.com/p/4f11bf2004bd
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。