# 雕牌LEDE个性化修改

## 修改主机名，设定时区，IP地址

修改位置：nano package/base-files/files/bin/config_generate

```
generate_static_system() {
uci -q batch <<-EOF
delete system.@system[0]
add system system
set system.@system[-1].hostname='LEDE'
set system.@system[-1].timezone='CST-8'                      #东八区
set system.@system[-1].zonename='Asia/Shanghai'       #这句话要加上，不然还是UTC

lan) ipad=${ipaddr:-"192.168.2.1"} ;;       #修改默认IP
```

## 修改默认语言主题：

修改 nano feeds/luci/modules/luci-base/root/etc/config/luci 文件

```
config internal themes
        option Argon  "/luci-static/argon"
```

## SSH/TELNET显示信息修改方式：

修改nano package/base-files/files/etc/banner文件。