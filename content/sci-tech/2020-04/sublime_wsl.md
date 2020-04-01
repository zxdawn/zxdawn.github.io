---
title: Run sublimetext in WSL
date: 2020-03-24
tags: ["linux"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2020-04/sublime_text_logo.jpg"
summary: "Enjoy sublime text in WSL"
---

## Background

I need to install `xESMF` package on my windows laptop. However, it just supports Linux and Mac.

So, I switch to the Windows Subsystem for Linux (WSL). The [installation](https://docs.microsoft.com/en-us/windows/wsl/install-win10) is easy.

## Install sublime text

Since it's quite slow to install sublime text3 from its official website using `apt-get`, I downloaded the tarball from the [sublime text website](https://www.sublimetext.com/3).

Then, add it to `~/.bashrc`:

```
$ wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
$ echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
$ sudo apt update
$ apt install sublime-text
$ sudo ln -s /opt/sublime/sublime_text /usr/bin/subl
```

## Run sublime text

I'm using [MobaXterm](https://mobaxterm.mobatek.net/) which is an enhanced terminal for Windows with X11 server, tabbed SSH client, network tools and much more.

However when I type `subl`, the GUI looks blurry:

![blurry](/images/sci-tech/2020-04/blurry.jpg)

Because my screen is 4K, I need some tricks for this:

### Override the high DPI scaling

The font becomes so small:
   ![high_dpi](/images/sci-tech/2020-04/high_dpi.jpg)
   ![small_font](/images/sci-tech/2020-04/small_font.jpg)

### Scale the UI

`Prefrence >> Settings`:

```
// Settings in here override those in "Default/Preferences.sublime-settings",
// and are overridden in turn by syntax-specific settings.
{
	"ui_scale" : 2.5
}
```

Restart the sublime text and the fontsize and tab_font size looks great!

But, the menus all still small.

![small_menu](/images/sci-tech/2020-04/small_menu.jpg)

### GDK_scale

Finally, I use the GDK_scale and set the `font_size` manually:

```
$ export GDK_SCALE=2
```

Settings:

```
// Settings in here override those in "Default/Preferences.sublime-settings",
// and are overridden in turn by syntax-specific settings.
{
	// "ui_scale" : 2.5
	"font_size": 15
}

```

  ![gdk_scale](/images/sci-tech/2020-04/gdk_scale.jpg)


## References

1. [Run sublime in WSL on 4K screen](https://github.com/sublimehq/sublime_text/issues/3251)
2. [Blurry X11 forwarding via MobaXTerm](https://stackoverflow.com/questions/46890879/blurry-x11-forwarding-via-mobaxterm)
3. [Blurry fonts on using Windows default scaling with WSL GUI applications (HiDPI)](https://superuser.com/questions/1370361/blurry-fonts-on-using-windows-default-scaling-with-wsl-gui-applications-hidpi)
4. [How do I edit the Solarized (Light) theme in Sublime Text 3](https://stackoverflow.com/questions/18746993/how-do-i-edit-the-solarized-light-theme-in-sublime-text-3)
5. [How to increase the file tree and tab font size?](https://forum.sublimetext.com/t/how-to-increase-the-file-tree-and-tab-font-size/5821)
6. [SUBLIME INCREASE TAB FONT SIZE](https://phpcodez.com/sublime-increase-tab-font-size/)