---
title: ssh内网穿透
date: 2018-04-29 21:20:50
tags: ["Linux","HPC"]
categories: ["sci-tech"]
draft: false
toc: true
summary: "校外使用校内网的方法"
img: "/images/sci-tech/2018-04/intranet.png"
---

机房电脑，通过远程端口映射，使得VPS：2222 映射到机房电脑：22：

```
ssh -N -f -R 2222:127.0.0.1:22 root@vpsIP
```

VPS则可以ssh到机房电脑:

```
ssh -p 2222 username@localhost
```

用ssh代理来浏览内网:

在校外电脑上，通过本地端口映射，使得校外电脑：2222 映射到VPS: 2222

```
ssh -N -f -L 2222:127.0.0.1:2222 root@vpsIP
```

<!--more-->

现在可以登录机房电脑了:

```
ssh -p 2222 username@localhost 
```

也可设置7070端口，使得浏览器通过代理访问资源:

```
ssh -qTfnN -D 7070 username@localhost -p 2222
```

chrome设置:

```
socks5,127.0.0.1,7070
```

添加一行到～/.ssh/config对应的host下，即可连接hpc:

```
ProxyCommand nc -X 5 -x 127.0.0.1:7070 %h %p
```

这样就可以愉快的登录校内大型机和下载图书馆文献啦~

参考:

[ssh内网穿透，实现校外访问校内网](http://blog.xulihang.me/ssh-penetrate-nat/)