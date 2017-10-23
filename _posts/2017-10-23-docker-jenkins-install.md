---
layout: post
title: 使用docker部署jenkins
author: magic
date:   2017-10-23
categories: Jenkins
tags: docker jenkins
permalink: /archivers/docker-jenkins-install
---
* 目录
{:toc}

## 使用docker部署jenkins
[官方部署文档](https://hub.docker.com/_/jenkins/)  
[jenkins官方的docker镜像](https://hub.docker.com/_/jenkins/)  
一个带maven的镜像  
[java8-jenkins-maven-git-vim](https://hub.docker.com/r/lw96/java8-jenkins-maven-git-vim/)

主机环境：
AWS免费主机，系统Amazon Linux AMI release 2017.09，相当于centos7

### 安装docker

> \# yum安装  
> yum install docker -y  
> \# 启动docker服务  
> service docker start  
> \# 查看docker进程  
> ps -ef|grep docker  

### 获取jenkins镜像

> \# 拉取镜像  
> docker pull jenkins  
> \# 查看镜像  
> docker images  

### 启动jenkins镜像

> \# 创建外挂配置目录  
> mkdir /var/jenkins_home  
> \# 赋权  
> chown 1000 /var/jenkins_home  

> \# 后台启动  
> docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v /var/jenkins_home:/var/jenkins_home jenkins

- 这句命令的意思是：在后台运行一个基于jenkins镜像的容器, 容器的名字叫做 jenkins,把容器的8080,50000端口映射为主机端口8080,50000端口，并且把服务器上的/var/jenkins_home目录挂在到docker容器上的/var/jenkins_home目录。  
- -d 后台运行docker容器；如果不加-d则，容器运行会占用此终端，如果终端关闭，则容器也相应关闭，jenkins就无法访问了。加上-d,容器会在后台运行。  
- --name 为容器起个别名；如果不起别名，则系统会默认分配一个随机别名，类似gklasd_sdfwe。起了别名后，后续会通过该别名管理该docker容器，也就是管理jenkins。  
- -p docker容器端口映射；jenkins服务是运行在docker里的，docker默认不对外暴露端口的。  
- -v 文件挂载；如果不挂载，则jenkins所有log、用户配置文件都会在docker容器内，如果容器销毁，则jenkins得重新配置一遍。挂载出来方便jenkins迁移以及管理。

使用 'docker ps' 查看是否启动成功

### 配置jenkins

不出意外，浏览器访问http://host:8080 ，会出现如下界面
![](http://upload-images.jianshu.io/upload_images/1300665-dbdf6cbc78e7d23c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打开initialAdminPassword文件/var/jenkins_home/secrets/initialAdminPassword
复制里面的密码，粘贴到Administrator password里

选择安装推荐插件即可
![](http://upload-images.jianshu.io/upload_images/1300665-85ac0e0e4e8ce926.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

……

### 采坑过程
镜像启动失败
```
…… /var/jenkins_home/copy_reference_file.log: Permission denied
```
[解决办法](https://github.com/jenkinsci/docker/issues/177)  
> sudo chown 1000 volume_dir

### 参考链接

[一步一步打造jenkins+docker+nodejs项目的自动部署环境](http://www.jianshu.com/p/052a2401595a)  
[Jenkins + Docker 项目持续部署实践](https://blog.kinpzz.com/2017/06/08/jenkins-docker-ci-cd/)  
[使用Jenkins自动部署nodejs应用](http://rrestjs.lofter.com/post/80f11_3e80927)  