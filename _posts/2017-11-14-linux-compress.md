---
layout: post
title: linux的解压缩
author: magic
date:   2017-11-02
categories: Linux
tags: linux
permalink: /archivers/linux-compress
---
* 目录
{:toc}

### 参考
[文件打包与解压缩](http://www.jianshu.com/p/013142267016)

[linux怎样解压.gz文件及解压其他](http://blog.sina.com.cn/s/blog_3edc5e2e0102vzsb.html)

<!--more-->

1、*.tar 用 tar –xvf 解压
2、*.gz 用 gzip -d或者gunzip 解压
3、*.tar.gz和*.tgz 用 tar –xzf 解压
4、*.bz2 用 bzip2 -d或者用bunzip2 解压
5、*.tar.bz2用tar –xjf 解压
6、*.Z 用 uncompress 解压
7、*.tar.Z 用tar –xZf 解压
8、*.rar 用 unrar e解压
9、*.zip 用 unzip 解压