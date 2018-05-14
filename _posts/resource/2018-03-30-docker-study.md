---
layout: post
title: Docker Practise学习随记
category: Docker
tags: Docker
keywords: Docker
description: 
---

## 常用资源

### 一次删除所有本地docker镜像
```
docker image rm $(docker image ls -q) 
```
### 组合查询你要的字段
```
docker image ls --format "{{.Repository}}: {{.Tag}}"
```
```
root@ubuntu14:/opt/down-image# docker image ls --format "{{.Repository}}: {{.Tag}}"
<none>: <none>
traefik: v1.5.4
mysql: 5.7.21
memcached: 1.5.6
```
### 组合查询且等距离显示
```
docker image ls --format "table {{.Repository}}\t{{.Tag}}"
```
```
root@ubuntu14:/opt/down-image# docker image ls --format "table {{.Repository}}\t{{.Tag}}"
REPOSITORY          TAG
<none>              <none>
traefik             v1.5.4
mysql               5.7.21
memcached           1.5.6
```
### 写DockerFile每一层后，记得清理掉无关文件，否则会使镜像臃肿
```
FROM debian:jessie
RUN buildDeps='gcc libc6-dev make' \
&& apt-get update \
&& apt-get install -y $buildDeps \
&& wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
&& mkdir -p /usr/src/redis \
&& tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
&& make -C /usr/src/redis \
&& make -C /usr/src/redis install \
&& rm -rf /var/lib/apt/lists/* \
&& rm redis.tar.gz \
&& rm -r /usr/src/redis \
&& apt-get purge -y --auto-remove $buildDeps
```

## 以构建nginx镜像为例
### DockerFile准备
```
root@ubuntu14:/opt/down-image# mkdir jyq
root@ubuntu14:/opt/down-image# mv Dockerfile jyq/
root@ubuntu14:/opt/down-image# cat jyq/Dockerfile 
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

### 构建镜像，加标签
```
root@ubuntu14:/opt/down-image# cd jyq/
root@ubuntu14:/opt/down-image/jyq# docker build -t nginx:v3 .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM nginx
 ---> c5c4e8fa2cf7
Step 2/2 : RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
 ---> Running in f8696fb38b12
Removing intermediate container f8696fb38b12
 ---> c2796dd83d40
Successfully built c2796dd83d40
Successfully tagged nginx:v3
root@ubuntu14:/opt/down-image/jyq# docker images | grep nginx
nginx                                    v3                  c2796dd83d40        35 seconds ago      109MB
nginx                                    latest              c5c4e8fa2cf7        4 days ago          109MB
root@ubuntu14:/opt/down-image/jyq# 

```

## 镜像迁移
### 从一台机器将镜像迁移至另一台机器
```
root@ubuntu14:/opt/down-image# docker save openvz/ubuntu:14.04 | bzip2 | pv | ssh root@172.18.4.41 'cat | docker load'
The authenticity of host '172.18.4.41 (172.18.4.41)' can't be established.
ECDSA key fingerprint is ad:a8:64:82:b6:a0:db:99:3f:d2:55:67:23:61:0f:27.
Are you sure you want to continue connecting (yes/no)?    0B 0:00:01 [   0B/s] [<=>                                                                                                   yes                                                   ]
Warning: Permanently added '172.18.4.41' (ECDSA) to the list of known hosts.

Authorized users only. All activities may be monitored and reported.
root@172.18.4.41's password:    0B 0:00:02 [   0B/s] [<=>                                                                                                                                                         ]
64.4MB 0:00:42 [1.53MB/s] [                                                                                                                  <=>                                     ]
root@ubuntu14:/opt/down-image# 
```

### 登录目标机器去检查
```
[root@host-172-18-4-41 ~]# docker images | grep ubuntu
openvz/ubuntu                                     14.04               e1d895170986        10 minutes ago      214.8 MB
[root@host-172-18-4-41 ~]# 
```
### 容器操作
```
docker container start
docker container stop
docker container restart
```
