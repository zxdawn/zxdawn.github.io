---
title: Setting up my Elementary OS
date: 2020-12-04
tags: ["linux"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2020-12/elementary_os.png"
summary: "Enjoy the beautiful Linux system: Elementary OS"
---

## Mirror

Edit the `/etc/apt/sources.list` if you live in China:

```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

Update the system:
```
sudo apt-get update && sudo apt-get upgrade
```

## NVIDIA Driver

AppCenter --> NVIDIA X Server Settings --> restart --> AppCenter --> install the newest nvidia driver 

However, it doesn't work for my 4K screen.

I have to use xrandr to set it manually and restart the PC:

```
xrandr # see available screens
gsettings set org.gnome.desktop.interface scaling-factor 2 # 2x scale
gsettings set org.gnome.desktop.interface scaling-factor 1 # 1x scale
xrandr --output DP-1-1 --scale 1x1 --rate 60 # 60Hz at 1x scale. DP-1-1 is the name you see with "xrand", 
```

## Tweaks

### Installation

```
sudo apt install software-properties-common
sudo add-apt-repository ppa:philip.scott/elementary-tweaks
sudo apt update
sudo apt install elementary-tweaks
```

### Disable single-click

System Settings --> Tweaks --> Files --> disable single-click

### Disable natural copy-paste in terminal

Tweaks --> Terminal --> disable natural copy-paste.

### Windows style

Elementary OS has only close and maximize buttons by default.

Tweaks appearance --> Windows.

## Input method

```
sudo apt-get install fcitx fcitx-googlepinyin
# download the deb from http://pinyin.sogou.com/linux/download.php?f=linux&bit=64
sudo dpkg -i <deb_filename>
im-config # choose fcitx
fcitx-configtool # set shorcuts for fcitx
```

## [Enable tray icons for third-party apps](https://averagelinuxuser.com/after-install-elementary-juno/#14-enable-tray-icons-for-third-party-apps)

edit the *indicator-application* by adding `Pantheon` to `OnlyShowIn=Unity;GNOME;` line in `/etc/xdg/autostart/indicator-application.desktop`.

Then install the **old panel indicator** (`wingpanel-indicator-ayatan`) by downloading it from [launchpad.net](http://ppa.launchpad.net/elementary-os/stable/ubuntu/pool/main/w/wingpanel-indicator-ayatana/) and running:

```
sudo dpkg -i wingpanel-indicator-ayatana_2.0.3+r27+pkg17~ubuntu0.4.1.1_amd64.deb 
```

## Make Keyring password Blank

```
sudo apt-get install seahorse
```

launch “seahorse” --> Right-click on “Login” and select “Change Password.” --> Enter the old password when you see the pop-up. Then leave the new password field blank. Don’t enter even space. Click ‘Continue’. --> You should see an obvious warning pop-up that passwords will be unencrypted. Click ‘Continue’.

## Softwares (Linux)

- GDebi

We can use GDebi to install deb packages easily.
```
sudo apt install gdebi
```

- [Typora](https://typora.io/)

```
wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -

# add Typora's repository
sudo add-apt-repository 'deb https://typora.io/linux ./'
sudo apt-get update

# install typora
sudo apt-get install typora
```

- Git

```
sudo apt-get install git

# set poxy (optional)
git config --global http.proxy 'socks5://127.0.0.1:1089'
```

- hugo

As my website is based on the hugo of old version, I installed it as below:

```
wget -O /tmp/hugo.deb https://github.com/gohugoio/hugo/releases/download/v0.50/hugo_0.50_Linux-64bit.deb
sudo dpkg -i /tmp/hugo.deb
```

- Texlive

```
sudo apt install texlive latexmk texlive-latex-extra texlive-bibtex-extra
```

- [Sublime Text](https://www.sublimetext.com/)

```
wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
sudo apt-get update
sudo apt-get install sublime-text
```

- [Package Control](https://packagecontrol.io/installation)

Open the command palette --> ctrl+shift+p --> Type Install Package Control, press enter

- [LaTeXTools](https://latextools.readthedocs.io/en/latest/install/)

Ctrl+shift+p --> select the Package Control: Install Package --> LaTeXTools

- [Qv2ray](https://github.com/Qv2ray/Qv2ray)

Run it on Startup:

Applications --> System settings --> Applications --> Startup --> `/home/xin/Softwares/qv2ray/Qv2ray.v2.6.2.linux-x64.AppImage --`

- [Notion](https://www.notion.so/)

Unfortunately, official Notion isn't supported on Linux. I use [Lotion](https://github.com/puneetsl/lotion) instead.

Clone the repo and link the `Lotion.desktop` to `/usr/share/applications/Lotion.desktop`.

## Softwares (Windows)

Download and install [deepin-wine](https://github.com/wszqkzqk/deepin-wine-ubuntu ) which supports TIM and Wechat.