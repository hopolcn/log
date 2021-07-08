## docker Alpine(简洁型)镜像的使用

https://blog.csdn.net/weixin_44266650/article/details/89539463



使用Apk不支持yum
登录alpine

Docker run –it alpine sh

apk 安装软件

apk add --no-cache bash
apk add --no-cache curl
apk add --no-cache python3

查看当前任务列表

Jobs –l 

-9强制kill(被杀死)如果任务列表存在

Kill -9 11    

查看当前的容器编号

Cat /proc/self/cgroup | head -1 

清除缓存

docker ps -a | grep "Exited" | awk '{print $1 }'|xargs docker stop
docker ps -a | grep "Exited" | awk '{print $1 }'|xargs docker rm

清除内存

rm -rf /var/cache/apk/*     

alpine 配置django

apk更新

apk update

apk 安装软件

apk add --no-cache bash
apk add --no-cache curl
apk add --no-cache python3

wget --no-check-certificate https://bootstrap.pypa.io/get-pip.py
python3 get-pip.py