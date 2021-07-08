# Alpine Linux系统安装Docker以及Docker Compose环境
> Alpine Linux系统安装Docker环境，拓展更多玩法,从根本上杜绝内存浪费。

**安装社区存储库：**

echo "http://dl-cdn.alpinelinux.org/alpine/latest-stable/community" >>/etc/apk/repositories

**安装Docker:**

apk update;apk add docker

**设置开机自启：**

rc-update add docker boot

**启动Docker：**

service docker start

**查看Docker版本：**

docker -v

**安装Docker Compose：**

apk add py-pip;pip install docker-compose