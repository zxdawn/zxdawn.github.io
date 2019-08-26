---
title: Install git and oh-my-zsh without root
date: 2018-06-28 11:57:14
tags: ["Linux", "Git"]
categories: ["sci-tech"]
draft: false
toc: true
summary: "Install git and oh-my-zsh on hpc without root permission"
img: "/images/sci-tech/2018-06/OMZ.png"
---

# Git

Check `Installing from Source` part of the [guide](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).

What  I do is a little different from it:

## Curl

Since `git` depend on `curl`, I installed `curl` from source according to this [guide](https://curl.haxx.se/docs/install.html).

## Git

```
# Download git and uncompress
$ tar -zxf git-2.18.0.tar.gz
$ cd git-2.18.0
$ make configure

# Define CURLDIR=/public/home/zhangxin/softwares/curl if your curl header and library files are # in /public/home/zhangxin/softwares/curl/include and /public/home/zhangxin/softwares/curl/lib # directories.
$ ./configure CURLDIR=/public/home/zhangxin/softwares/curl --prefix=/public/home/zhangxin/softwares/git

# Since we don't need docs actually, just 'make'.
$ make CURLDIR=/public/home/zhangxin/softwares/curl
$ make install CURLDIR=/public/home/zhangxin/softwares/curl
```

<!--more-->

If you encounter missing dependencies, just google and install them from source. For me, I installed `xmlto` and `asciidoc`.

# oh-my-zsh

## Zsh

Zsh should be installed (v4.3.9 or more recent). If not pre-installed (zsh --version to confirm), check the following instruction here: [Installing ZSH](https://github.com/robbyrussell/oh-my-zsh/wiki/Installing-ZSH).

## [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)

1. Clone the repository:

   `git clone https://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh`

2. Create a new zsh configuration file

   `cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc`

3. Since I'm using bash, I added this to `.bashrc`:

   `exec /public/home/zhangxin/softwares/zsh/bin/zsh -l`

   Howerver, I got this error:

```
/etc/zshrc:54: compinit: function definition file not found
/public/home/zhangxin/.oh-my-zsh/lib/theme-and-appearance.zsh:2: colors: function definition file not found
/public/home/zhangxin/.oh-my-zsh/oh-my-zsh.sh:76: compinit: function definition file not found
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:81: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:95: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:102: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:115: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:125: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:135: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:144: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:150: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:153: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:156: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:159: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:169: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:172: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:174: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:190: command not found: compdef
/public/home/zhangxin/.oh-my-zsh/plugins/git/git.plugin.zsh:202: command not found: compdef
```

This can be fixed by adding this line to `.bashrc` before zsh:

```
export FPATH=/public/home/zhangxin/softwares/zsh/share/zsh/5.5.1/functions:$FPATH
```



