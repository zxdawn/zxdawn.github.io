---
title: Remarkable - Connecting to a hotspot which supports circumvention
date: 2018-01-18
tags: ["Remarkable", "web"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2018-01/remarkable.jpg"
summary: "Some tips for using Remarkable in China"
---

## Method 1

Using [VPN Hotspot](https://play.google.com/store/apps/details?id=be.mygod.vpnhotspot) to share the VPN you set on your Android phone.

## Method 2

### Set up VPN

<!--more-->

1. You should choose a software which supports virtual LAN, otherwise win10 hotspot won't support VPN. I will use [SSTAP](https://www.sockscap64.com/sstap/) which supports HTTP/SOCKS4/5/SHADOWSOCKS/SSR.

2. Before using the software, you need your own account. You have two choices: buy an account or set up VPN on VPS.

3. Following [this tutorial](https://www.jianshu.com/p/519e68b74646), you will set up the environment of VPN successfully and get a adapter:

   ![SSTAP_1](/images/sci-tech/2018-01/SSTAP_1.png)

### Turn your Windows laptop into a WiFI Hot Spot

1. press the Windows logo key + R and type "cmd".

   ![cmd](/images/sci-tech/2018-01/cmd.png)

2. Type "netsh wlan show drivers" (without the quotation marks) in Command Prompt to check whether or not your computer supports a hosted network. The "Hosted network supported" field should indicate "Yes" if your unit supports WiFi sharing. If it says "No," you'll have to download the corresponding driver for your WiFi adapter first before proceeding. For example, I load Windows 8.1 drivers from the [INF](https://www.killernetworking.com/driver-downloads/item/killer-drivers-inf), restarted and hosted network is supported.

   ![hoseted_network](/images/sci-tech/2018-01/hoseted_network.png)

3. To create a hotspot, type "netsh wlan set hostednetwork mode=allow ssid=yournetworkname key=yournetworkpassword," and hit Enter on your keyboard. I use 'Remarkable' as my network name.

4. To get the hotspot up and running, type "netsh wlan start hostednetwork."

5. Right click the WIFI button and click 'Open Network & Internet settings'. Then click 'WIFI' of the left panel and 'Change adapter options' of the right panel.

   ![opennetwork](/images/sci-tech/2018-01/opennetwork.png)

   ![adapter](/images/sci-tech/2018-01/adapter.png)![remarkable.png](https://i.loli.net/2018/01/18/5a600c6a97555.png)

6. Choose 'SSTAP 1', right click on it, select "Properties," and go to the "Sharing" tab. Check the option 
   "Allow other network users to connect through this computer's Internet connection." This time, select the WiFi hotspot you created earlier.

   ![sharing](/images/sci-tech/2018-01/sharing.png)

### Connect you Remarkable to a Wi-Fi network

1. Connect to Wi-Fi by choosing the network you want to connect to, and enter a password if required. The lock icon indicates that a password is required to connect to the Wi-Fi network.
2. The device will say “Connecting to ---” when your device is connecting to the selected network.
3. When you are successfully connected a white dot will appear next to the Wi-Fi name, as well as the text “Connected to …”. The signal strength is indicated by the icon next to the Wi-Fi name.

### Check for update

These are introductions from Remarkable:

> How to update with auto-update enabled:
>
> 1. Look for a notification in the lower left corner on the home screen. This notification will tell you to restart your device to complete the installation.
> 2. Restart your device to complete the installation.
>
> How to update manually:
>
> 1. Open Settings by tapping the rM-icon in the top left corner in My Files
> 2. Make sure you are connected to Wi-Fi in Wi-Fi settings.
> 3. In Device settings, tap Check for updates to see if there are any new updates available. 
> 4. Tap Download version** **to update to a new version.
> 5. When the download is finished, click Restart device to complete the update.

## Further Resources

- http://www.remarkabletabletuser.com/

- https://github.com/reHackable

- http://blog.lucafluri.ch/2017-11-21/customizing-remarkableTablet

- http://remarkablewiki.com/index.php?title=Main_Page

## References

1. https://www.dell.com/support/article/cn/zh/cndhs1/sln208643/how-to-turn-your-windows-laptop-into-a-wifi-hot-spot?lang=en
2. https://www.jianshu.com/p/519e68b74646
3. https://answers.microsoft.com/en-us/windows/forum/windows_10-networking/cant-create-hotspot-anymore/a81ec382-bd2b-46b0-94d8-9a526483150f?auth=1