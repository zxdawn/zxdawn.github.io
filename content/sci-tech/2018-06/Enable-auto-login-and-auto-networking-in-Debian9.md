---
title: Enable auto-login and auto-networking in Debian 9
date: 2018-06-15 10:42:48
tags: ["Linux"]
categories: ["sci-tech"]
draft: false
toc: true
summary: "Auto login the NUIST internet"
---

# Auto-login

`sudo nano /usr/share/lightdm/lightdm.conf.d/01_debian.conf`

Add these lines below

```
[SeatDefaults]
autologin-user=username
autologin-user-timeout=0
```

# Auto-networking

## Python script 

This is the example ([python实现校园网自动登录](https://blog.csdn.net/shenhuaifeng/article/details/78333851)):

<!--more-->

```
#!/usr/bin/python3
# -*- coding: utf-8 -*-
'''
   FileName:conNet.py
   Author:shenhuixiang
   Copyright(c)2017,shenhuixiang
'''
import base64
import requests
'''
输入账号密码和登录的网络
网络参数为如果是移动的则填写CMCC
如果是学号则填NUIST
'''
USER_ACCOUNT='110'
DOMAIN_SELECTION='CMCC'
USER_PASSWATD='123456'

#登录地址
post_addr="http://a.nuist.edu.cn/index.php/index/login"

#构造头部信息
post_header={
    'Host': 'a.nuist.edu.cn',
    'User-Agent':'Mozilla/5.0 (X11; Linux x86_64; rv:55.0) Gecko/20100101 Firefox/55.0',
    'Accept': 'application/json, text/javascript, */*; q=0.01',
    'Accept-Language':'zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3',
    'Accept-Encoding': 'gzip, deflate',
    'Content-Type': 'application/x-www-form-urlencoded',
    'X-Requested-With':'XMLHttpRequest',
    'Referer':'http://a.nuist.edu.cn/index.php?url=aHR0cDovL2RldGVjdHBvcnRhbC5maXJlZm94LmNvbS9zdWNjZXNzLnR4dA==',
    'Content-Length': '67',

    'Cookie':'_gscu_1147341576=059821653286gq10; sunriseUsername='+USER_ACCOUNT+';\
    sunriseDomain='+DOMAIN_SELECTION+';sunriseRememberPassword=true; sunrisePassword='+USER_PASSWATD+';\
    PHPSESSID=hb0o9bkct2f6ge164oj3vj0me5;think_language=zh-CN',
    'Connection':'keep-alive',
}

'''
password在post的参数中经过base64编码,
为了查找password加密方式...吐血三升.
'''
post_data={'domain':DOMAIN_SELECTION,
           'enablemacauth':'0',
           'password':base64.b64encode(USER_PASSWATD.encode()),
           'username':USER_ACCOUNT
          }
#发送post请求登录网页
z=requests.post(post_addr,data=post_data,headers=post_header)
#z.text为str类型的json数据因此先编码成byte类型在解码成unicode型这样就可以正常输出中文
s=z.text.encode('utf-8').decode('unicode-escape')
print(s)
```

## Make Python script run at startup

Save your script named `inuist.py`;

`vi ~/inuist.sh`

```
#!/bin/bash
python ~/inuist.py
```

`chmod +x ~/inuist.sh`

`vi ~/.bash_profile`

```
~/inuist.sh
```

# Auto-remote

Check the location of Anydesk or any software you use by`which anydesk`

copy it to `/etc/init.d`