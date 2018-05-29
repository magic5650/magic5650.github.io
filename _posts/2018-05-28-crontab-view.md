---
layout: post
title: crontab查看的各种姿势
author: magic
date:   2018-05-28
categories: Linux
tags: linux crontab sudo
permalink: /archivers/crontab-view
---
* 目录
{:toc}

## 背景
我们有些服务需要用root跑一些定时任务，但开发使用的服务器用户是普通用户，登上去是没有权限看到root的任务的

他们希望使用一个简单的命名就能看到root用户的定时任务，比如 ct
<!--more-->
## 方法
linux管理员都知道crontab配置也是基于文件和内存管理的

有关crontab知识可详见 [Linux定时任务crontab/cron.d介绍](http://cering.github.io/2015/11/02/%E8%BD%AC-Linux%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1crontab-cron-d%E4%BB%8B%E7%BB%8D/)

### 使用备份文件及alias
起初我们运维更新root用户的crontab都会备份前一份配置到 /tmp/backup/cron.bak

既然如此我们大可以更新完后，将当前的crontab配置也备份一份到 /tmp/backup/cron.now

那现在就变成了如何输入 ct 就能查看 /tmp/backup/cron.now 文件的问题了


编辑 ~/.bashrc
```
alias ct='cat /tmp/backup/cron.now'
```
这样用户执行ct的时候实际上是在执行 cat /tmp/backup/cron.now

### 使用sudo执行crontab
很显然，第一种方案需要管理员更新crontab的同时备份crontab配置

如果管理员忘记了备份最新的crontab配置，那么开发人员看到的就不是最新的crontab配置

所以我们需要一种不需要管理员备份最新配置的方案，更方便更简单直接 

这样我们理所应当的想到了执行 crontab -l，但由于开发者账号是没有root的权限的，这里我们使用sudo代替

编辑 /etc/sudoers 加入
```
Cmnd_Alias      YCMDS = /usr/bin/crontab -l -u root
developer ALL=(ALL) NOPASSWD: YCMDS
```
这里假设普通用户为 developer

这里别忘了加 alias

编辑 ~/.bashrc
```
alias ct='sudo /usr/bin/crontab -l -u root'
```
这样developer用户执行 ct 就能查看到root用户当前的crontab配置了，方便、实时且准确

### 使用特殊权限的二进制程序
如果你是个linux管理员，你应该知道普通用户修改自身密码背后的秘密；没错就是SUID

有关SUID，可以参考[linux中SUID，SGID和SBIT的奇妙用途](https://blog.csdn.net/xiaocainiaoshangxiao/article/details/17378611)

那现在我们要做的就是
1. 编写一个二进制程序，这个程序能够实时的获取root用户的crontab配置信息
2. 将这个程序的所有者设为root，并且添加SUID权限和可执行权限

在普遍的linux系统中，root的用户的定时任务配置文件会保存在文件 /var/spool/cron/root 中，
所以我们的任务就简化成了编写一个程序读取这个文件，这里我们考虑用最简单的C语言

ct.c
```
#include "stdio.h"
#include "stdlib.h"

int main (){
    FILE *fp;
    int c;
    fp = fopen("/var/spool/cron/root", "r");
    if (fp==NULL)
    {
        printf("no crontab for root\n");
        exit (1);
    }
    while ((c = fgetc(fp)) != EOF){
        printf ("%c", c);
    }
    fclose(fp);
    return 0;
}
```
编译并授权
```
gcc ct.c -o ct
cp ct /usr/bin/ct
chmod +s+x ct /usr/bin/ct
```

也许你还可以使用go，更简单且更优雅
```
package main

import (
    "io/ioutil"
    "fmt"
)

func main(){
    buf,err := ioutil.ReadFile("/var/spool/cron/root")
    if err != nil{
        fmt.Println("error:",err)
        return
    }
    fmt.Printf("%s",string(buf))
}
```