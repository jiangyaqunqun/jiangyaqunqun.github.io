---
layout: post
title: 改造memcached镜像
category: Docker
tags: Docker
keywords: Docker
description: 
---

## 初衷
memcached镜像在公司paas平台,作为短job启动,并且启动后立即停止,尝试在其内部添加永久运行进程,使其不会自动停掉
同时修改容器时区为UTF-8

### 镜像改造步骤
step 1
从docker hub拿到memcached 1.5.7官方镜像
step 2
以root权限登录进去修改时区,本地永久进程脚本拷贝进容器
```
root@ubuntu14:/opt/down-image/jyq# docker images memcached
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
memcached           1.5.7               4f178e337fba        2 days ago          58.6MB
memcached           2.2.2               4f178e337fba        2 days ago          58.6MB
memcached           1.1.1               78988adb8ac3        2 weeks ago         58.6MB
root@ubuntu14:/opt/down-image/jyq# 
root@ubuntu14:/opt/down-image/jyq# 
root@ubuntu14:/opt/down-image/jyq# 
root@ubuntu14:/opt/down-image/jyq# docker run -u 0 -it 4f178e337fba /bin/bash
root@faecfb2fd234:/# 
root@faecfb2fd234:/# 
root@faecfb2fd234:/# 
```
