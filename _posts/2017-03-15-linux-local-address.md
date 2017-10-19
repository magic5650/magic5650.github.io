---
layout: post
title: linux监听端口local address释义
author: magic
date:   2017-03-15
categories: Linux
tags: linux
permalink: /archivers/linux-local-address
---
As an example, apache has it's Listen option which tells it which address and port to listen on. Depending on how this is set, apache will listen on any IP address, a specific address:-
<!--more-->
作为一个例子，apache有它的Listen选项，这些选项会告知哪个地址和端口要监听。根据设置，apache将可以监听任何IP地址，有如下监听地址配置：

```shell
Listen *:80
Listen 0.0.0.0:80
Listen 127.0.0.1:80
Listen 192.168.0.5:80
```

>The above options show up as: 上面的选项在系统上显示为(可通过netstat -ltnp查看)

```shell
:::80
0.0.0.0:80
127.0.0.1:80
192.168.0.5:80
```

>and translate to: 转义过来就是

Listen on any IP address (IPv4 or IPv6)——在所有IP (IPv4 或者 IPv6)地址上监听

Listen on any IPv4 address on that server——在所有服务器的IPv4的地址上监听

Listen on IPv4 localhost only——只在本机的IPv4的回环地址上监听

Listen on external IPv4 address 192.68.0.5——只在IPv4地址192.68.0.5上监听

You could configure your service to listen on only the localhost interface if you don't want anyone external to access it. For example, if you're running a LAMP server you'd have apache listening on all IP addresses (so that your users can access it) while a mysql database could be configured to be accessible only from localhost (using it's bind=127.0.0.1 directive). This way php running on the same server will be able to access the database while external (and untrusted) users will not be able to access it.


如果你不想你的服务接收外部请求，你可以配置你的服务只在本机回环接口建立监听。例如，你运行了一个LANP的站点服务，你的apache在所有的IP地址上都建立了监听（以便于你的用户能够访问它），同时你的mysql数据库只需要接受来自本机的请求（监听器直接绑定了127.0.0.1）。这种方法能够使本机的php能够访问数据库，同时外部（和不受信任的）用户却无法访问数据库。