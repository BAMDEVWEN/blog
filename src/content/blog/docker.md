---
title: 'Docker基础使用'
description: 'docker 的基本使用'
pubDate: 'Nov 15 2024'
heroImage: '/docker.png'
tags: ['docker','docker-compose']
---
# Docker
##  Dockerfile
- From 指定基础镜像pm
- MAINTAINER 指定维护者信息
- Run zai镜像中执行的指令
- CMD 启动容器默认执行的指令   CMD ["nginx", "-g", "daemon off;"]
- EXPOSR 声明容器指定的端口号
- ENV 设定环境变量
- ADD 将本地文件或目录复制到镜像当中 ADD app.jar /var
- COPY 将本地文件或目录复制到镜像当中
- WORKDIR 设置工作目录
- USER 设置容器运行时得分用户 USER nginx
- volume 声明匿名卷
- ARG 定义构建时可以传递的参数
- LABEL 添加元数据标签

## DOCKERCOMPOSE
1. service
   ```yaml
    services:
     db:
    image: mysql
    environment: 
      MYSQL_ROOT_PASSWORD: password
   
    web:
     build: 
      contxt: . //构建上下文
      dockerfile: DOCKFILE 指定文件的名称
      ports:
       - "8080:80" //主机端口:容器端口
     depends_on:
       - db
   ```
    - 参数：
        - image 指定镜像名称
        - build
        - container_name 容器的名称
        - ports
        - volumes 设置容器内的环境变量
        - commad 启动时要执行的命令
        - depends_on 服务依赖关系
        - networks 指定所属的网络
        -  environment:      设置北京时间
           - - TZ=Asia/Shanghai

```yaml
version: "3"
services:
  mysql:
    image: mysql:8.0
    container_name: mysql_public
    networks:
      mynet:
        ipv4_address: 192.168.0.5 #指定IP地址
    ports:
      - "9907:4406"
    environment:
      MYSQL_TCP_PORT: 4406
      MYSQL_ROOT_PASSWORD: yuan0601
      MYSQL_DATABASE: simulator
    volumes:
      - db_public:/var/lib/mysql
volumes:
  db_public:
  
networks:
  mynet: #网络名
    driver: bridge #网络模式桥接
    ipam: #配置网络管理的配置
      driver: default 
      config:
       - subnet: 192.168.0.0/16 #子网CIDR
         gateway: 192.168.0.1 # 网关

#主服务
version: '3.0'
services:
  app:
    image: "lottery:v1"
    networks:
     mynet:
        ipv4_address: 192.168.0.6
    ports:
      - "18077:3000"
networks:
  mynet:
    external:
      name: test_mynet #指定网络

```

```yaml

#使用存在的网络
networks:
  persist:
    external:
      name: bridge2
```

## 健康性检查

```yaml
services:
  db:
    image: mysql
    # 其他配置项...
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 3s
      retries: 10

services:
  web:
    build: .
    depends_on:
      db:
        condition: service_healthy  #当服务健康时为正常时才能启动
```



# Docker启动

- docker run
    - -p 配置端口
    - -d 后台启动
    - -name 指定容器名称
    - -e 指定环境变量
    - 删除全部 $(docker ps -aq)

##  DOCKER 网络

###  理解docker网络

```shell
#docker 是如何处理容器网络范围跟

```

![image-20231113120602459](/image-20231113120602459.png)

```shell
docker run -d -P --name tomcat01 tomcat #启动一个tomcat容器
#linux宿主机可以ping docker 容器内部
```

#### 容器可能缺少相关指令

```shell
docker exec -it <container-name> bash
apt update
apt install inetutils-ping
apt install iproute2
```



#### 原理

1、 启动一个容器，docker会给docker 容器分配一个ip，就会有一个网卡docker0

桥接模式，使用enth-pair技术

连接虚拟网络设备

2、 tomcat01与tomcat能ping通

```shell
#容器之间可以ping通
#tomacat02容器
ping 172.17.0.2
```

![image-20231113132617422](/image-20231113132617422.png)

结论：通过docker0路由转发，不指定网络的情况下，都是dockers的可用网络

Docker使用linux的虚拟网络连接

### --link

通过服务名ping

```shell
#docker exex -it tomcat02 ping tomcat01

root@wens:/home/wen# docker exec -it tomcat02 ping tomcat01
ping: tomcat01: Name or service not known\
# 如何解决
docker run -d -P --link tomcat02 --name tomcat03 tomcat
#方向能ping通？
```

#### 原理

```shell
root@wens:/home/wen# docker exec -it  tomcat03 bash
root@337a61c48174:/usr/local/tomcat# cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.3      tomcat02 a0f782a551e5
172.17.0.4      337a61c48174
#查看host配置 --link在host中加了一个 ip的映射 如：
172.17.0.3      tomcat02 a0f782a551e5

```

summary：不建议使用--link

docker0：不支持容器名访问

## 自定义网络

> 查看所有的网络

**网络模式**

brige：桥接

none：不配置网络

host： 和宿主机共用网络

container：容器网络联通

```shell
#默认启动 --net brige,默认的docker0
docker run -d -P tomcat02 --net brige tomcat
#docker0 特点，域名不能访问
#创建网络
root@wens:/home/wen# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
# 查看所有网络
root@wens:/home/wen# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
2eceded35eda   bridge    bridge    local
db49fdfd6bc4   host      host      local
3512783a968f   mynet     bridge    local
301947d93638   none      null      local
# 查看容器启动后的网络
root@wens:/home/wen# docker  inspect mynet
[
    {
        "Name": "mynet",
        "Id": "3512783a968f954b6f64ec51d2b29c4950f7fa613c093b305e207a0acc252a39",
        "Created": "2023-11-13T06:24:45.78766367Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "003516a6532b689788016f0ae169012232b44a114f9a87fc7efa27d1edf7d085": {
                "Name": "tomcat-net-02",
                "EndpointID": "3eb8b9bb4b927da5617a31b26badb126e8b9c65f3760f188f41215dd82a05017",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "64c93c4b311b791b567a27edf1dd768662bc033ae7d6fb1e8f2fb92ff396acd9": {
                "Name": "tomcat-net-01",
                "EndpointID": "4b907c89937f8d297ff6d94da1784f25c1fdac4fddc1cf7e02e65f6f06a02f59",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

```

结论：自定义网络维护好相关网络，推荐（不设置--link也可以通过容器名连通）

## 网络联通

```shell
docker network connect mynet tomcat
#联通将tomcat01放到mynet网络下
# 一个容器两个ip

root@wens:/home/wen# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "3512783a968f954b6f64ec51d2b29c4950f7fa613c093b305e207a0acc252a39",
        "Created": "2023-11-13T06:24:45.78766367Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "003516a6532b689788016f0ae169012232b44a114f9a87fc7efa27d1edf7d085": {
                "Name": "tomcat-net-02",
                "EndpointID": "3eb8b9bb4b927da5617a31b26badb126e8b9c65f3760f188f41215dd82a05017",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "5d5376da09b4db2b821053e438a924e5fdfdc2c13a8730d22c26e8ce688d5687": {
                "Name": "tomcat01",
                "EndpointID": "fceb446493379e80dd0aaac76880f331eb8c666013b223e79c878d01a14f1173",
                "MacAddress": "02:42:c0:a8:00:04",
                "IPv4Address": "192.168.0.4/16",
                "IPv6Address": ""
            },
            "64c93c4b311b791b567a27edf1dd768662bc033ae7d6fb1e8f2fb92ff396acd9": {
                "Name": "tomcat-net-01",
                "EndpointID": "4b907c89937f8d297ff6d94da1784f25c1fdac4fddc1cf7e02e65f6f06a02f59",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

```

结论：跨网络联通



## 实战redis部署

```shell
#清除docker
 docker rm -f $(docker ps -aq)
 #创建网络
 docker network create redis --subnet 172.38.0.0/16
 #脚本创建redis
for port in $(seq 1 6); \
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
port 6379 
bind 0.0.0.0
cluster-enabled yes 
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done

docker run -p 637${port}:6379 -p 1637${port}:16379 --name redis-${port} \
-v /mydata/redis/node-${port}/data:/data \
-v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf 
#1
docker run -p 6371:6379 -p 16371:16379 --name redis-1 \
-v /mydata/redis/node-1/data:/data \
-v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf 
#2
docker run -p 6372:6379 -p 16372:16379 --name redis-2 \
-v /mydata/redis/node-2/data:/data \
-v /mydata/redis/node-2/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.12 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf 
#3
docker run -p 6373:6379 -p 16373:16379 --name redis-3 \
-v /mydata/redis/node-3/data:/data \
-v /mydata/redis/node-3/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.13 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf 
#4
docker run -p 6374:6379 -p 16374:16379 --name redis-4 \
-v /mydata/redis/node-4/data:/data \
-v /mydata/redis/node-4/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.14 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf 
#5
docker run -p 6375:6379 -p 16375:16379 --name redis-5 \
-v /mydata/redis/node-5/data:/data \
-v /mydata/redis/node-5/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.15 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf 
#6
docker run -p 6376:6379 -p 16376:16379 --name redis-6 \
-v /mydata/redis/node-6/data:/data \
-v /mydata/redis/node-6/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.16 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf 
#没学过，待续

```

[38、Redis集群部署实战_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1og4y1q7M4?p=38&spm_id_from=pageDriver&vd_source=d61d9dd65ed60a0e3707c806dcaf906c)