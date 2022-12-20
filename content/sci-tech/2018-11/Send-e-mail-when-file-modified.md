---
title: Send e-mail when file modified
date: 2018-11-18 22:00:48
tags: ["Linux"]
categories: ["sci-tech"]
draft: false
toc: true
summary: "Send e-mail when file modified"
img: "/images/sci-tech/2018-11/Gmail.png"
---

## mailx

Download `mailx`  ([openSUSE](https://software.opensuse.org/package/mailx) is the distribution of my server) and install it.

```
$ make PREFIX=/public/home/zhangxin/softwares/mailx SYSCONFDIR=/public/home/zhangxin/softwares/mailx/etc
$ make install UCBINSTALL=/usr/bin/install PREFIX=/public/home/zhangxin/softwares/mailx SYSCONFDIR=/public/home/zhangxin/softwares/mailx/etc
```

Get your authorization code and edit `mailx/etc/nail.rc`

```
set from="363910399@qq.com"
set smtp=smtp.qq.com
set smtp-auth-user="363910399@qq.com"
set smtp-auth-password="authorization_code"
set smtp-auth=login
```

<!--more-->

Test:

```
$ echo "haha" | mailx -v -s "Test" xinzhang1215@gmail.com
```

Usage:

```
mailx -T FILE -u USER -h hops -r address -s SUBJECT -a FILE -q FILE -f FILE -A ACCOUNT -b USERS -c USERS -S OPTION users
```

## inotifywait

Download inotifywait from [here](https://github.com/rvoicilas/inotify-tools/wiki#info) and install it.

```
#!/bin/bash
inotifywait -m /public/software/anaconda/requirements.txt --timefmt '%d/%m/%y %H:%M' --format '%T %w %f' -e modify -e move -e delete_self | while read date time action dir file; do
    FILECHANGE=${dir}${file}
    echo "At ${time} on ${date}, file $FILECHANGE was touched via '$action'" | mailx -s "requirements" -a /public/software/anaconda/requirements.txt xinzhang1215@gmail.com
done
```

Explanation:

```
	-m|--monitor  	Keep listening for events forever.  Without
	              	this option, inotifywait will exit after one
	              	event is received.
	-e|--event <event1> [ -e|--event <event2> ... ]
		Listen for specific event(s).  If omitted, all events are 
		listened for.
```
