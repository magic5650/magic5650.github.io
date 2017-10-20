---
layout: post
title: linux sudo免密码的多种方式
author: magic
date:   2017-10-19
categories: Linux
tags: linux sudo
permalink: /archivers/linux-sudo-nopasswd
---
* 目录
{:toc}

## linux sudo免密码的多种方式

最近在使用chroot制作跳板机的时候，遇到了有关sudo -i 和sudo -s 切换至root用户的异同，于是看了下sudo的man说明文档，看到了-S这个参数，于是做了下面的实验。

系统执行sudo命令的时候有时需要输入密码，如果是写在脚本里面就比较麻烦了
一般我们会想到使用expect，但其实还是太麻烦了

<!-- more -->
### 使用-S选项
```
man sudo
……
-S   The -S (stdin) option causes sudo to read the password from the standard input instead of the terminal device.  The password
must be followed by a newline character.
……
```
译文：
-S：-S (stdin)选项使sudo从标准输入中读取密码，而不是终端设备获取密码。密码后面必须有一个换行符。
例如:
```
echo 'passwd' sudo -S pwd
sudo -S pwd < passwd.txt
```

### 使用-A选项
```
man sudo
……
-A   Normally, if sudo requires a password, it will read it from the user's terminal.  If the -A (askpass) option is specified, a
 (possibly graphical) helper program is executed to read the user's password and output the password to the standard output.  If the SUDO_ASKPASS environment variable is set, it specifies the path to the helper program.  Otherwise, if /etc/sudo.conf con-tains a line specifying the askpass program, that value will be used.  For example:

    # Path to askpass helper program
    Path askpass /usr/X11R6/bin/ssh-askpass

If no askpass program is available, sudo will exit with an error.
……
```
译文：
-A：通常，如果sudo需要一个密码，它将从用户的终端读取它。如果指定了-A(askpass)选项，那么(可能是图形化的)助手程序被执行来读取用户的密码并将密码输出到标准输出。如果设置了sudoaskpass环境变量，它将指定帮助程序的路径。否则,如果/etc/sudo.conf指定了一个指定askpass的程序，用于输入密码。例如:
\# Path to askpass helper program
Path askpass /usr/X11R6/bin/ssh-askpass
如果没有askpass程序可用，sudo将会以错误退出。

 1、创建一个密码文件，如_PWD_TEMP_
```
vim  _PWD_TEMP_
```
写入内容：
```
#! /bin/bash
echo  password
```
2、在脚本中执行sudo 命令之前引入环境变量SUDO_ASKPASS
```
export SUDO_ASKPASS=./_PWD_TEMP_
```
3、 执行命令
```
sudo -A  command
```

### 配置 /etc/sudoers （NOPASSWD）
```
magic ALL=(ALL)  NOPASSWD:  ALL
```
简单粗暴
### 还是配置 /etc/sudoers 
To disable asking for a password for user USER_NAME:
```
magic ALL=(ALL)   ALL
Defaults:magic     !authenticate
```
来自[wiki.archlinux.org](https://wiki.archlinux.org/index.php/sudo#Example_entries)

**这边AWS上的EC2默认的用户ec2-user没有使用上述任何的手段，就可以使用sudo并且不用密码，很是不解。**