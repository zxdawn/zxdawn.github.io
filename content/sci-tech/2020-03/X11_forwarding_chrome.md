---
title: X11 forwarding Chrome of internal PC
date: 2020-03-11
tags: ["Linux"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2020-03/bridge.jpg"
summary: "Access the internal resource by proxy"
---

## Method

Just type this in one terminal:

```
 ssh -L 8822:internal_pc:22 user@proxy_server
```

Then, open another terminal and type:

```
ssh -p 8822 -X localhost
```

Note that if you have opened the Chrome on your internal PC, you need to specify a new **user-data-dir** like this:

```
google-chrome --user-data-dir=~/.config/windos-chrome
```

If you don't have a proxy server which has the internal access, you can check another [blog](https://dreambooker.site/2018/04/29/ssh%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F/) about how to use VPS as the proxy server.

## Reference

1. [PuTTY X11 forwarding can't forward Google Chrome](https://unix.stackexchange.com/questions/80478/putty-x11-forwarding-cant-forward-google-chrome)
5. Photo by [Edvinas Daugirdas](https://unsplash.com/@edvinasd?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/bridge?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

## Version control

| Version | Action | Time       |
| ------- | ------ | ---------- |
| 1.0     | Init   | 2020-03-11 |