---
layout: post
title: python 发送带图片的邮件
author: magic
date:   2019-08-23
categories: python
tags: python email
permalink: /archivers/python-send-email
---
* 目录
{:toc}

## python 发送带图片的邮件
<!--more-->
**废话少说，直接上代码**
```
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
from email.mime.base import MIMEBase
from email.header import Header
from email.utils import parseaddr, formataddr
from email import encoders
import smtplib
import os
import configparser
from datetime import datetime,timedelta

'''
发送多张图片邮件的python代码
'''
img_list = '/home/xxxx/magicreport/picture/'
allfile=[]

'''
遍历文件夹获得所有图片绝对路径
'''
def getallfile(path):
    allfilelist = os.listdir(path)
    for file in allfilelist:
        filepath = os.path.join(path,file)
        if os.path.isdir(filepath):
            getallfile(filepath)
        else:
            allfile.append(filepath)
    return allfile

def sendEmail(receivers, subject, filelist, ccreceivers = ''):
        # 设定root信息
        mail_host = 'mail.xxx.com'
        mail_user = 'xxx@xxx.com'
        mail_pass = '******'  # 口令
        sender = 'xxx@xxx.com'

        msgRoot = MIMEMultipart('related')
        msgRoot['Subject'] = Header(subject, 'utf-8').encode()
        msgRoot['From'] = mail_user
        msgRoot['To'] = receivers
        msgRoot['Cc'] = ccreceivers
        '''
        图片id加入所在位置
        '''
        content = '<b>' + subject + '</b><br>'
        #content += '<b><a href="http://xxx.xxx.com/?orgId=1">10天趋势图</a></b><br>'
        content += '<br>'

        filelist.sort()

        for file in filelist:
            filename = file.split('/')[-1]
            content += '<img src="cid:' + filename + '"><br>'

        msgText = MIMEText(content, 'html', 'utf-8')
        msgRoot.attach(msgText)
        '''
        将图片和id位置对应起来,使用文件名
        '''
        for file in filelist:
            fp = open(file, 'rb')
            msgImage = MIMEImage(fp.read())
            fp.close()
            filename = file.split('/')[-1]
            msgImage.add_header('Content-ID', '<' + filename + '>')
            msgRoot.attach(msgImage)

        print(datetime.now().strftime("%Y-%m-%d"))
        try:
            #发送邮件
            smtp = smtplib.SMTP_SSL()
            smtp.connect(mail_host, 465)
            # smtp = smtplib.SMTP()
            # smtp.connect(mail_host, 25)
            smtp.login(mail_user, mail_pass)
            smtp.sendmail(sender, receivers.split(',') + ccreceivers.split(','), msgRoot.as_string())
            smtp.quit()
            print("邮件发送成功!")
        except smtplib.SMTPException as e:
            print("失败：" + str(e))

if __name__ == '__main__' :
        yestoday = datetime.now() - timedelta(days=1)
        subject = '带图片的邮件--' + yestoday.strftime("%Y-%m-%d")
        filelist = getallfile(img_list)

        config = configparser.ConfigParser()
        config.read("/home/xxxxx/emailaddr.conf")
        xxxre = config.get("xxx-ops", "receivers")
        wsre = config.get("ws", "receivers")
        sendEmail(xxxre, subject, filelist, wsre)
```

## 需要特别注意的地方

[Mail multipart/alternative vs multipart/mixed](https://stackoverflow.com/questions/3902455/mail-multipart-alternative-vs-multipart-mixed)
```
mixed
    alternative
        text
        related
            html
            inline image
            inline image
    attachment
    attachment
in outlook inline image must use parent related,other email client can use both related and alternative
```