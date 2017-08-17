---
layout: post
title: linux系统内核TCP性能优化
author: magic
date:   2016-11-15
categories: Linux
tags: linux
permalink: /archivers/linux-tcp-optimization
---
* 目录
{:toc}

## linux系统内核TCP性能优化

> /proc/sys/fs/file-max

指定了可以分配的文件句柄的最大数目（可以使用 /proc/sys/fs/file-nr 文件查看到当前已经使用的文件句柄和总句柄数。）
这个数值应大于hard limit
 
> /etc/security/limits.conf

* soft nofile 262140 #代表最大文件打开数 
* hard nofile 262140

在linux中这些限制是分为软限制(soft limit)和硬限制(hard limit)的。他们的区别就是软限制可以在程序的进程中自行改变(突破限制)，而硬限制则不行(除非程序进程有root权限)

> net.ipv4.tcp_syncookies=1

表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；

> net.ipv4.tcp_max_syn_backlog=81920

表示SYN队列的长度，默认为1024，加大队列长度为81920，可以容纳更多等待连接的网络连接数。

> net.ipv4.tcp_synack_retries=3

表示内核放弃连接之前发送SYN+ACK包的数量

> net.ipv4.tcp_syn_retries=3

表示内核放弃建立连接之前发送SYN包的数量

> net.ipv4.tcp_fin_timeout = 30

表示如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间。需要注意的是，即使一个负载很小的Web服务器，也会出现因为大量的死套接字而产生内存溢出的风险。FIN-WAIT-2的危险性比FIN-WAIT-1要小，因为它最多只能消耗1.5KB的内存，但是其生存期长些。

> net.ipv4.tcp_keepalive_time = 300

表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时

> net.ipv4.tcp_keepalive_probes = 30
> net.ipv4.tcp_keepalive_intvl = 3

上面两行意思是，如果probe 3次(每次30秒)不成功,内核才彻底放弃。
原来的默认值为，显然太大：

> net.ipv4.tcp_tw_reuse = 1

表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；

> net.ipv4.ip_local_port_range = 20000    65535

表示用于向外连接的端口范围。缺省情况下很小：32768到61000，改为20000到65535。

> net.ipv4.tcp_max_tw_buckets = 5000

表示系统同时保持TIME_WAIT套接字的最大数量，如果超过这个数字，TIME_WAIT套接字将立刻被清除并打印警告信息。默认为180000，改为 5000。对于Apache、Nginx等服务器，上几行的参数可以很好地减少TIME_WAIT套接字数量，但是对于Squid，效果却不大。此项参数 可以控制TIME_WAIT套接字的最大数量，避免Squid服务器被大量的TIME_WAIT套接字拖死。

> net.ipv4.route.max_size = 5242880

表示路由缓存最大值

> kernel.core_pattern = /data/core_files/core-%e-%p-%t

表示core dump的文件路径

> net.core.wmem_default=8388608

发送缓存区预留内存默认大小 默认值 16k

> net.core.rmem_default=8388608

接受缓存区预留内存默认大小 默认值 16k

> net.core.wmem_max=16777216

发送缓存区预留内存最大值 默认值 128k

> net.core.rmem_max=16777216

接受缓存区预留内存最大值 默认值 128k

> net.ipv4.tcp_tw_recycle=1

表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。

> net.ipv4.netfilter.ip_conntrack_max=204800

表示系统对最大跟踪的TCP连接数的限制


参考链接:

[http://www.jianshu.com/p/fb4dc2356582](http://www.jianshu.com/p/fb4dc2356582)
[https://help.aliyun.com/knowledge_detail/41334.html](https://help.aliyun.com/knowledge_detail/41334.html)
[http://colobu.com/2014/09/18/linux-tcpip-tuning](http://colobu.com/2014/09/18/linux-tcpip-tuning/)

