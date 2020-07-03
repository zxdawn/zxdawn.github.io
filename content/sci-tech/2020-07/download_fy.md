---
title: Download FENGYUN data in one script
date: 2020-07-03
tags: ["satellite","fengyun"]
categories: ["sci-tech"]
draft: false
toc: true
img: "/images/sci-tech/2020-07/NSMC.png"
summary: "I can't stand the official download method any more !!!"
---

## Background

The download method of FENGYUN satellite is so inconvenient, especially for large datasets ...

So, I decide to scrape the FTP links from the "Order" page and save them into one file.

Then, `lftp` could speed up the download speed a lot!

## Requirements

Python installed with three packages: logging, getpass and selenium

## Usage

Script and example: <https://github.com/zxdawn/weather_data/tree/master/FY>

1. Order data on the [FY website](http://satellite.nsmc.org.cn/portalsite/default.aspx) (all platforms) or using the [FY Toolkit](http://fy4.nsmc.org.cn/nsmc/en/data/pcclient.html) (Windows).

2. Run the script from the terminal

   ```
   $ python fy.py
   ```

   Input the requested infos ....

3. Check the bash script named `download_fy.sh`

   (You can change the name by `savename` in the `fy,py` script)

4. Run the bash script

   ```
   $ chmod +x download_fy.sh
   $ ./download_fy.sh
   ```

   Example of the bash script:

   ```
   #!/bin/bash
   lftp -e "mget -c ftp://AO20200701000066936:Uo6O5__j@ftp.nsmc.org.cn/*" &
   lftp -e "mget -c ftp://AO20200701000065328:0lK_rxpW@ftp.nsmc.org.cn/AO202007010000653280001/*" &
   lftp -e "mget -c ftp://AO20200701000065328:0lK_rxpW@ftp.nsmc.org.cn/AO202007010000653280002/*" &
   lftp -e "mget -c ftp://AO20200701000065328:0lK_rxpW@ftp.nsmc.org.cn/AO202007010000653280003/*" &
   ```

## Version control

| Version | Action | Time       |
| ------- | ------ | ---------- |
| 1.0     | Init   | 2020-07-03 |