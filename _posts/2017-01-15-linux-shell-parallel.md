---
layout: post
title: shell并发执行示例
author: magic
date:   2017-01-15
categories: Linux
tags: linux
permalink: /archivers/linux-shell-parallel
---


## shell并发执行示例

**背景：从服务器上抽取出来的IP需要调用接口查询IP地址所在地，直接for循环使用&后台并行查询接口时，有可能受到接口的限制，会导致获取失败；
故采用批量查询的方法，一次查询n个，查询完后在执行下一次查询**
<!--more-->
```
#!/bin/bash

exec 200<>$0
flock -n -e 200 || { echo "already Running, exit!";exit 1; }
trap 'rm -f ipinfo.txt;exit' 0 1 2 3 15
nums="$1"
[ -z "$nums" ] && nums=20

function get_ip_info(){
	ip_array=("$1")
	for ipaddr in ${ip_array[@]}
	do
		ipaddr=$(echo "$ipaddr"|tr -d '\r')
		ip_info=$(curl -m 3 --retry 1 http://ip.sysop.duowan.com/ipDesc.php?ip=${ipaddr} 2>/dev/null|grep "info: " | sed 's/info: //g')
		if [ "$ip_info" == "" ];then
			[ "$ip_info" == "" ] && ip_info="未确定地址信息"
		fi
		echo "${ipaddr};${ip_info}" >> ipinfo.txt
	done
}

ip_count=$(wc -l iplist.txt|awk '{print $1}')
for(( i=1;i<="$ip_count";i= i + "$nums" ))
do
	j=$(( $i + $nums -1 ))
	ip_array=$(sed -n ""$i","$j"p" iplist.txt)
	get_ip_info "$ip_array" &
done
wait
```

nums为并行查询数量

iplist.txt为存放IP的文件
例：

```
1.1.1.1
2.2.2.2
……
```

ipinfo.txt为存放获取的IP信息文件，例

```
1.1.1.1 中国，广东，广州 电信
2.2.2.2 中国，广东，广州 联通
……
```