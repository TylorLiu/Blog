---
title: Linux服务器性能参数
date: 2018/2/10 14:42
tags: [Linux,性能]
category: tips
---
### 防火墙
service iptables start
iptables -F

### 网络连接相关
sysctl -w netfilter.ip_conntrack_tcp_timeout_time_wait=10  
sysctl -w net.ipv4.netfilter.ip_conntrack_tcp_timeout_close=10
netstat -n |grep 8846 | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'  端口使用状态统计

  


#### 统一修改配置文件 /etc/sysctl.conf
 
加入以下内容：
net.ipv4.tcp_syncookies = 1
表示开启SYN Cookies。；
net.ipv4.tcp_tw_reuse = 1
表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 1
表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
net.ipv4.tcp_fin_timeout = 30
表示如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间。
net.ipv4.ip_local_port_range = 1024 65000
修改端口范围，默认是32768 61000
net.ipv4.tcp_max_tw_buckets = 3000

当time_out达到多少以后，系统自动释放链接


执行 /sbin/sysctl -p 让参数生效
 
### 修改文件配置
/proc/sys/net/ipv4/netfilter/ip_conntrack_tcp_timeout_time_wait  值从原来的120改为10
这是tcp连接中断后系统等待的释放的时间, 否则在高并发下面产生大量的time_out，导致无法连接
 
1. 查看网络TCP连接状态命令
#netstat -n | awk '/^tcp/ {++state[$NF]} END {for(key in state) print key,"\t",state[key]}'
 
2. 查看Timeout配置命令
#sysctl -a | grep time | grep wait

文件打开数设置
ulimit -n
1. 修改/etc/security/limits.conf文件，在文件中添加如下行：
* soft nofile 10240
* hard nofile 10240
2. 修改/etc/pam.d/login文件，在文件中添加如下行：
session required /lib/security/pam_limits.so
3. 查看Linux系统级的最大打开文件数限制，使用如下命令：
cat /proc/sys/fs/file-max
  修改  ： echo 22158 > /proc/sys/fs/file-max
4. 修改用户打开数限制
echo ulimit -HSn 65536 >> /etc/rc.local
echo ulimit -HSn 65536 >>/root/.bash_profile
ulimit -HSn 65536

### 数据库相关
alter database clear unarchived logfile group 7;

alter system set DB_RECOVERY_FILE_DEST_SIZE=20g;


