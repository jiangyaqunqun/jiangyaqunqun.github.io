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
以root权限登录进去修改时区，容器窗口保持打开，之所以一定要root用户登录，是因为容器进去默认是memcache用户，而该用户没有权限拷贝时区文件，必须要root用户才有权限
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
另外打开一个linux窗口，将本地写好的永久进程脚本拷贝至容器内
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
最初想法有点蠢，执意想要dockerfile中基于memcached:1.5.7来修改，于是就傻乎乎的将官方1.5.7打上其它标签，上面修改过的3.0打上1.5.7标签，现在看来完全没有必要，dockerfile直接引用memcached3.0就好了，给出修改标签的方法
```
root@ubuntu14:/home/huawei# docker images memcached
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
memcached           3.0                 524606192459        8 seconds ago       58.6MB
memcached           1.5.7               4f178e337fba        2 days ago          58.6MB
memcached           2.2.2               4f178e337fba        2 days ago          58.6MB
memcached           1.1.1               78988adb8ac3        2 weeks ago         58.6MB
root@ubuntu14:/home/huawei# docker tag memcached:1.1.1 memcached:1.1.2
root@ubuntu14:/home/huawei# docker images memcached
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
memcached           3.0                 524606192459        26 minutes ago      58.6MB
memcached           1.5.7               4f178e337fba        2 days ago          58.6MB
memcached           2.2.2               4f178e337fba        2 days ago          58.6MB
memcached           1.1.1               78988adb8ac3        2 weeks ago         58.6MB
memcached           1.1.2               78988adb8ac3        2 weeks ago         58.6MB
root@ubuntu14:/home/huawei# 
```

step 6
本地创建dfile夹，里面创建dockerfile镜像，dockerfile文件权限默认644，dockerfile内容如下，主要就是使窗口起来时，去执行已经拷贝进去的date.sh脚本，使其作为永久进程跑
```
root@ubuntu14:/opt/down-image/jyq# cd dfile/
root@ubuntu14:/opt/down-image/jyq/dfile# ls -al
total 12
drwxr-xr-x 2 root root 4096 May 14 09:24 .
drwxr-xr-x 3 root root 4096 May 14 10:27 ..
-rw-r--r-- 1 root root   90 May 11 14:49 dockerfile
root@ubuntu14:/opt/down-image/jyq/dfile# cat dockerfile 
FROM memcached:1.5.7
ENTRYPOINT [ "/etc/date.sh" ]
ONBUILD RUN rm -rf /etc/huawei-license
root@ubuntu14:/opt/down-image/jyq/dfile# 
```


step 7
全部准备好以后，开始构建镜像
```
root@ubuntu14:/opt/down-image/jyq# docker build -t memcached-huawei:1.5.7 dfile/
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM memcached:1.5.7
 ---> 4f178e337fba
Step 2/3 : ENTRYPOINT [ "/etc/date.sh" ]
 ---> Running in 52cea82c6ee6
Removing intermediate container 52cea82c6ee6
 ---> 76238d42d599
Step 3/3 : ONBUILD RUN rm -rf /etc/huawei-license
 ---> Running in e50b52f9c85b
Removing intermediate container e50b52f9c85b
 ---> 2792fd37e2fd
Successfully built 2792fd37e2fd
Successfully tagged memcached-huawei:1.5.7
root@ubuntu14:/opt/down-image/jyq# 
root@ubuntu14:/opt/down-image/jyq# 
root@ubuntu14:/opt/down-image/jyq# 
root@ubuntu14:/opt/down-image/jyq# docker images memcached
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
memcached           1.5.7               4f178e337fba        2 days ago          58.6MB
memcached           2.2.2               4f178e337fba        2 days ago          58.6MB
memcached           1.1.1               78988adb8ac3        2 weeks ago         58.6MB
root@ubuntu14:/opt/down-image/jyq# docker images memcached-huawei
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
memcached-huawei    1.5.7               2792fd37e2fd        15 seconds ago      58.6MB
```

step 8
将memcached-huawei镜像保存成tar文件，传至已有paas环境的镜像仓库，创建容器应用，可以成功一直运行


well done
人生第一次修改镜像，虽然很简单，但还是开心的~
