---
title: Tricks of TH-2 VPN
date: 2018-12-05 13:22:09
tags: ["Linux"]
categories: ["sci-tech"]
draft: false
toc: true
summary: "Make it easier to connect TH-2 VPN"
img: "/images/sci-tech/2018-12/TH-2.jpg"
---

## Install vpnc

### Link

https://pkgs.org/download/vpnc

### Edit vpnc config file

`/etc/vpnc/default.conf`

```
IPSec gateway vpn2.nscc-gz.cn
IPSec ID VPN
IPSec secret gzcszx@123
#IKE Authmode hybrid
Xauth username vpn_username
Xauth password vpn_password
```

<!--more-->

### Edit ssh config file

`~/.ssh/config`

```
Host    th
        Hostname        172.16.22.11
        IdentityFile    your_private_key
        User            username
```

### Login

`$ sudo vpnc`

`$ ssh th`

### Disconnect

`$ sudo vpnc-disconnect`

## vpnc traffic routing

### Background

I am having an issue when I connect to TH-2 VPN through VPNC.

The problem is when the VPN is up, all network traffic seems to get routed through the VPN, and I lose most internet connectivity, because the VPN is setup for internal work network access.

### Solution

1. Check normal IP before VPNC starts

   `$ sudo ifconfig`

   ```
   eno1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
           inet 10.13.35.197  netmask 255.255.0.0  broadcast 10.13.255.255
           inet6 2001:da8:1035:3::6:d746  prefixlen 128  scopeid 0x0<global>
           inet6 fe80::4639:c4ff:fe8e:31  prefixlen 64  scopeid 0x20<link>
           ether 44:39:c4:8e:00:31  txqueuelen 1000  (Ethernet)
           RX packets 90399666  bytes 73791857163 (68.7 GiB)
           RX errors 0  dropped 0  overruns 0  frame 0
           TX packets 120320074  bytes 143428796457 (133.5 GiB)
           TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
           device interrupt 20  memory 0xf7f00000-f7f20000  

   lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
           inet 127.0.0.1  netmask 255.0.0.0
           inet6 ::1  prefixlen 128  scopeid 0x10<host>
           loop  txqueuelen 1  (Local Loopback)
           RX packets 5203988  bytes 4549957994 (4.2 GiB)
           RX errors 0  dropped 0  overruns 0  frame 0
           TX packets 5203988  bytes 4549957994 (4.2 GiB)
           TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
   ```

   `$ sudo route`

   ```
   Kernel IP routing table
   Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
   default         bogon           0.0.0.0         UG    0      0        0 eno1
   default         bogon           0.0.0.0         UG    100    0        0 eno1
   10.13.0.0       0.0.0.0         255.255.0.0     U     100    0        0 eno1
   ```

2. Connect VPN and check again

   `$ sudo ifconfig`

   ```
   eno1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
           inet 10.13.35.197  netmask 255.255.0.0  broadcast 10.13.255.255
           inet6 2001:da8:1035:3::6:d746  prefixlen 128  scopeid 0x0<global>
           inet6 fe80::4639:c4ff:fe8e:31  prefixlen 64  scopeid 0x20<link>
           ether 44:39:c4:8e:00:31  txqueuelen 1000  (Ethernet)
           RX packets 90400426  bytes 73792065098 (68.7 GiB)
           RX errors 0  dropped 0  overruns 0  frame 0
           TX packets 120320839  bytes 143428966097 (133.5 GiB)
           TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
           device interrupt 20  memory 0xf7f00000-f7f20000  

   lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
           inet 127.0.0.1  netmask 255.0.0.0
           inet6 ::1  prefixlen 128  scopeid 0x10<host>
           loop  txqueuelen 1  (Local Loopback)
           RX packets 5204004  bytes 4549958800 (4.2 GiB)
           RX errors 0  dropped 0  overruns 0  frame 0
           TX packets 5204004  bytes 4549958800 (4.2 GiB)
           TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

   tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1412
           inet 172.17.63.9  netmask 255.255.255.255  destination 172.17.63.9
           inet6 fe80::64d2:86d6:7c39:c107  prefixlen 64  scopeid 0x20<link>
           unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 500  (UNSPEC)
           RX packets 0  bytes 0 (0.0 B)
           RX errors 0  dropped 0  overruns 0  frame 0
           TX packets 37  bytes 2922 (2.8 KiB)
           TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
   ```

   `$ sudo route`

   ```
   Kernel IP routing table
   Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
   default         0.0.0.0         0.0.0.0         U     0      0        0 tun0
   default         10.13.255.254   0.0.0.0         UG    100    0        0 eno1
   10.13.0.0       0.0.0.0         255.255.0.0     U     100    0        0 eno1
   114.67.37.68    10.13.255.254   255.255.255.255 UGH   0      0        0 eno1
   ```

3. Delete the route that vpnc adds, and add my route I need to go through the VPN.

   `$ sudo route add -host 172.16.22.11 dev tun0`

   `$ sudo route del default dev tun0`

   `$ sudo route`

   ```
   Kernel IP routing table
   Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
   default         10.13.255.254   0.0.0.0         UG    100    0        0 eno1
   10.13.0.0       0.0.0.0         255.255.0.0     U     100    0        0 eno1
   114.67.37.68    10.13.255.254   255.255.255.255 UGH   0      0        0 eno1
   172.16.22.11    0.0.0.0         255.255.255.255 UH    0      0        0 tun0
   ```

### Conclusion

It's better to write a script for this issue:

```
#!/bin/bash

echo "Connecting to myVPN..."
vpnc

echo "Setting up routing table..."
route del default dev tun0
route add -host 172.16.22.11 dev tun0

echo -n "Press Enter to continue..."
read
```

## Reference

[vpnc traffic routing - IPSec target network](https://ubuntuforums.org/showthread.php?t=1623624)