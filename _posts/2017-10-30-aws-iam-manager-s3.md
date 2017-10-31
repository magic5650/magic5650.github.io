---
layout: post
title: AWS的IAM授权管理S3存储桶
author: magic
date:   2017-10-30
categories: AWS
tags: aws iam s3
permalink: /archivers/aws-iam-manager-s3
---
* 目录
{:toc}

## AWS的IAM授权管理S3存储桶    
[配置 AWS CLI](http://docs.aws.amazon.com/zh_cn/cli/latest/userguide/cli-chap-getting-started.html)  
[通过 AWS Command Line Interface使用高级别 s3 命令](http://docs.aws.amazon.com/zh_cn/cli/latest/userguide/using-s3-commands.html)  
[Amazon S3：允许对特定的 S3 存储桶进行读写访问](http://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/reference_policies_examples_s3_rw-bucket.html)  
主机环境：  
AWS CLI环境，aws cli环境搭建，略    
[安装 AWS Command Line Interface](http://docs.aws.amazon.com/zh_cn/cli/latest/userguide/installing.html)   
<!--more-->
### 使用场景  
创建S3桶作为静态网站，使用Jenkins创建发布流程，需要将最新的静态网站同步到S3，使用脚本同步  
[在 Amazon S3 上托管静态网站](http://docs.aws.amazon.com/zh_cn/AmazonS3/latest/dev/WebsiteHosting.html)  
### 授权过程一  
AWS有自带的策略生成器  
[策略生成器](https://console.aws.amazon.com/iam/home?region=us-east-1#/policies$new)  
按理说，我们选择了所有操作即获得了所有权限  
我们指定一个存储桶  
于是我们生成了如下的策略申明  
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1509420933000",
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::play.example.com"
            ]
        }
    ]
}
```
然后我将这个策略附加到用户A上面，用户A来上传更新S3存储桶play.example.com的内容  
> export AWS_ACCESS_KEY_ID=*********  
> export AWS_SECRET_ACCESS_KEY=*********  
> export AWS_DEFAULT_REGION=us-east-1  
> export AWS_DEFAULT_OUTPUT=json  
> aws s3 sync . s3://play.example.com/ --delete --exclude '.git/*'  

我满怀期待它能够工作，结果却失败了，报"Access Deny"  

### 授权过程二   
一定是哪里出了问题，我想了想，改正如下
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1509420933000",
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::play.example.com/*"
            ]
        }
    ]
}
```
与上面不同的是，我在资源的后面加了"/*"  
我觉得它一定能工作，但结果是一样的，报"Access Deny"  
看来我不得不求助于官方文档了  

### 授权过程三  
我试着从这寻找答案
[IAM 故障排除策略](http://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/troubleshoot_policies.html#morethanonepolicyblock)  
```
{	
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": "s3:*",
    "Resource": [
      "arn:aws:s3:::play.example.com",
      "arn:aws:s3:::play.example.com/*"
    ]
  }
}
```
很明显，这一次，我把一和二结合起来了  
就这样我获得了成功  

### 进一步细化  
为了更精细化的控制权限，我在策略生成器的选项里面找到了我所需要的全新，更改了申明  
最终，我创建的策略如下  
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::play.example.com"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::play.example.com/*"
            ]
        }
    ]
}
```
这个看起来跟AWS的文档说明是差不多的  
[允许对特定的 S3 存储桶进行读写访问](http://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/reference_policies_examples_s3_rw-bucket.html)  
一个桶就是Bucket，桶里面的统称为Object，这样我们分别对Bucket和Object授权  
终于可以执行我的同步命令了，跟linux rsync同步命令非常的相似  
> aws s3 sync . s3://play.example.com/ --delete --exclude '.git/*'  