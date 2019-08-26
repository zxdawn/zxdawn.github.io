---
title: Running Win10 Debian Zsh in Cmder
date: 2018-05-05 15:21:53
tags: ["Linux", "Windows","Cmder"]
categories: ["sci-tech"]
draft: false
toc: true
summary: "Running Debian on Win10 and use Zsh in Cmder"
img: "/images/sci-tech/2018-05/cmder.jpg"
---

# References

[Running Windows 10 Ubuntu Bash in Cmder](https://gingter.org/2016/11/16/running-windows-10-ubuntu-bash-in-cmder/)

[How to Install and Use the Linux Bash Shell on Windows 10](https://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/)

# Prerequisites

1. Bash on Debian on Windows enabled 

2. Cmder. 

   If you don’t have that, you can simply follow these instructions:

> [Install Bash On Ubuntu on Windows](https://msdn.microsoft.com/de-de/commandline/wsl/install_guide) (the same way to install Debian)
>
> Download and unzip [Cmder](http://cmder.net/) to a convenient place on your disk. Then start *Cmder.exe*.
>
> [Install (Oh-my-) Zsh on Bash on Ubuntu on Windows](https://gingter.org/2016/08/17/install-and-run-zsh-on-windows/)

# Set up Zsh in Cmder

<!--more-->

1. Navigate to *Settings -> Startup -> Tasks* in Cmder.

2. Create a new Task by clicking on the ‘*+*‘ Button at the bottom.

3. Input task name `zsh::Debian`.

4. Input Task parameters ` /icon "%CMDER_ROOT%\icons\cmder.ico"`; I choose cmder default icon here. You can change this to your own Debian icon or other.

5. Input Commands `%windir%\system32\bash.exe ~ -c zsh -cur_console:p`.

6. Set zsh::Debian as your default task. You can navigate to *Settings -> Startup* and change *Specified named task*.

7. Save settings.

# Set up Cmder as Sublime Terminal

1. Download [Package Control](https://packagecontrol.io/) and use the *Package Control: Install Package* command from the command palette. Using Package Control ensures Terminal will stay up to date automatically.

2. Install `Terminal in Packages`

3. The default settings can be viewed by accessing the *Preferences > Package Settings > Terminal > Settings – Default* menu entry. To ensure settings are not lost when the package is upgraded, make sure all edits are saved to *Settings – User*.

4. Then set settings for Cmder:

   ```
   {
     // Replace with your own path to cmder.exe
     "terminal": "C:\\Program Files\\cmder_mini\\cmder.exe",
     "parameters": ["/START", "%CWD%"]
   }
   ```

5. Then you can open terminal at file press `ctrl+shift+t` or open terminal at project folder press `ctrl+alt+shift+t`.
