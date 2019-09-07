---
title: Linux常用命令
date: 2019-05-21 14:04:59
tags: linux 
category: blog
---

1. tar
````
//解压
tar xf jdk-7u65-linux-x64.tar.gz -C /usr/local
````
2. netstat
````
netstat -anp|grep 5123
````
3. lsof -i 查看端口所属进程
````
lsof -i: 8080
````
4. df -h 查看硬盘占用
5. du -sh *  查当前文件夹下所有文件（夹）占用磁盘大小