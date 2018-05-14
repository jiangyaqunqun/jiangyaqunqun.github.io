---
layout: post
title: 改造memcached镜像
category: Docker
tags: Docker
keywords: Docker
description: 
---

## 初衷
memcached镜像在公司paas平台，作为短job启动，并且启动后立即停止，尝试在其内部添加永久运行进程，使其不会自动停掉
同时修改容器时区为UTF-8

### 镜像改造步骤
step 1
从docker hub拿到memcached 1.5.7官方镜像

step 2
以root权限登录进去修改时区，容器窗口保持打开
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
root@faecfb2fd234:/# cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
root@faecfb2fd234:/# 
```

step 3
另外打开一个linux窗口，将本地写好的永久进程脚本拷贝至容器
```
root@ubuntu14:/home/huawei# cat date.sh 
#!/bin/bash
while true;do date;sleep 1;done

root@ubuntu14:/home/huawei# 
root@ubuntu14:/home/huawei# docker cp date.sh faecfb2fd234:/etc/
root@ubuntu14:/home/huawei# 
```

step 4
保存修改，随意给它打上标签，例如3.0，因为需要再基于它修改memcached镜像，这个标签仅用于过渡
```
root@ubuntu14:/home/huawei# docker commit faecfb2fd234 memcached:3.0
sha256:524606192459615d58d9bd96d3d74f4923208bb8a0f1b3eea4cc8a119e8263d2
root@ubuntu14:/home/huawei# docker images memcached
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
memcached           3.0                 524606192459        8 seconds ago       58.6MB
memcached           1.5.7               4f178e337fba        2 days ago          58.6MB
memcached           2.2.2               4f178e337fba        2 days ago          58.6MB
memcached           1.1.1               78988adb8ac3        2 weeks ago         58.6MB
root@ubuntu14:/home/huawei# 
```

step 5
