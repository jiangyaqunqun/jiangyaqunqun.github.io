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

### 好文
1. [PHP之道](http://wulijun.github.io/php-the-right-way/)
2. [Cookie/Session机制详解](http://blog.csdn.net/fangaoxin/article/details/6952954)

### 优秀的类库
1. [PHP中文分词: 自动打标签功能](http://jingwentian.com/t-145)

### 判断是否为空
```
+--------------+-----------+---------+-----------+---------+--------+
| 真值表        | gettype() | empty() | is_null() | isset() | (bool) |
+--------------+-----------+---------+-----------+---------+--------+
| $x = ""      | string    | true    | false     | true    | false  |
| $x=null      | NULL      | true    | true      | false   | false  |
| var $x       | NULL      | true    | true      | false   | false  |
| $x = array() | array     | true    | false     | true    | false  |
| $x = false   | boolean   | true    | false     | true    | false  |
| $x = 15      | integer   | false   | false     | true    | true   |
| $x = 1       | integer   | false   | false     | true    | true   |
| $x = 0       | integer   | true    | false     | true    | false  |
| $x = -1      | integer   | false   | false     | true    | true   |
| $x = '15'    | string    | false   | false     | true    | true   |
| $x = '1'     | string    | false   | false     | true    | true   |
| $x = '0'     | string    | true    | false     | true    | false  |
| $x = '-1'    | string    | false   | false     | true    | true   |
| $x = 'foo'   | string    | false   | false     | true    | true   |
| $x = 'true'  | string    | false   | false     | true    | true   |
| $x = 'false' | string    | false   | false     | true    | true   |
+--------------+-----------+---------+-----------+---------+--------+
```

## 常用命令

### 修改phpunit内存限制    

    phpunit -d memory_limit=512M


## PHPStorm 常用快捷键

- Quick Command

    `Command + Shift + A`

- Quick File

    `Command + Shift + O`

- Quick Class

    `Command + O`

- Quick Symbol

    `Command + Option + O`
