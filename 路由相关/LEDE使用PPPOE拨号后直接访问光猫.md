# LEDE使用PPPOE拨号后直接访问光猫

ifconfig eth1 192.168.1.3 netmask 255.255.255.0 broadcast 192.168.1.255 iptables -I forwarding_rule -d 192.168.1.1 -j ACCEPT iptables -t nat -I postrouting_rule -d 192.168.1.1 -j MASQUERADE