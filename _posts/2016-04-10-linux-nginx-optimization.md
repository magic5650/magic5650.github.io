---
layout: post
title: nginx、fastcgi优化
author: magic
date:   2016-04-10
categories: Linux
tags: linux nginx
permalink: /archivers/linux-nginx-optimization
---
* 目录
{:toc}

## nginx、fastcgi优化

从nginx的内部来看，一个HTTP Request的处理过程涉及到以下几个阶段。
 
>1. 初始化HTTP Request（读取来自客户端的数据，生成HTTP Request对象，该对象含有该请求所有的信息）。
>2. 处理请求头。
>3. 处理请求体。
>4. 如果有的话，调用与此请求（URL或者Location）关联的handler。
>5. 依次调用各phase handler进行处理。
<!--more-->
在这里，我们需要了解一下phase handler这个概念。phase字面的意思，就是阶段。所以phase handlers也就好理解了，就是包含若干个处理阶段的一些handler。

在每一个阶段，包含有若干个handler，再处理到某个阶段的时候，依次调用该阶段的handler对HTTP Request进行处理。

通常情况下，一个phase handler对这个request进行处理，并产生一些输出。通常phase handler是与定义在配置文件中的某个location相关联的。

一个phase handler通常执行以下几项任务：

1. 获取location配置。
2. 产生适当的响应。
3. 发送response header。
4. 发送response body。


当nginx读取到一个HTTP Request的header的时候，nginx首先查找与这个请求关联的虚拟主机的配置。如果找到了这个虚拟主机的配置，那么通常情况下，这个HTTP Request将会经过以下几个阶段的处理（phase handlers）：

- NGX_HTTP_POST_READ_PHASE:
 读取请求内容阶段

- NGX_HTTP_SERVER_REWRITE_PHASE:
 Server请求地址重写阶段

- NGX_HTTP_FIND_CONFIG_PHASE:
 配置查找阶段

- NGX_HTTP_REWRITE_PHASE:
 Location请求地址重写阶段

- NGX_HTTP_POST_REWRITE_PHASE:
 请求地址重写提交阶段

- NGX_HTTP_PREACCESS_PHASE:
 访问权限检查准备阶段

- NGX_HTTP_ACCESS_PHASE:
 访问权限检查阶段

- NGX_HTTP_POST_ACCESS_PHASE:
 访问权限检查提交阶段

- NGX_HTTP_TRY_FILES_PHASE:
 配置项try_files处理阶段

- NGX_HTTP_CONTENT_PHASE:
 内容产生阶段

- NGX_HTTP_LOG_PHASE:
 日志模块处理阶段

在内容产生阶段，为了给一个request产生正确的响应，nginx必须把这个request交给一个合适的content handler去处理。如果这个request对应的location在配置文件中被明确指定了一个content handler，那么nginx就可以通过对location的匹配，直接找到这个对应的handler，并把这个request交给这个content handler去处理。这样的配置指令包括像，perl，flv，proxy_pass，mp4等。