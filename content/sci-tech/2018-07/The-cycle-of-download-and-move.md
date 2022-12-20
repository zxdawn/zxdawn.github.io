---
title: The cycle of download and move
date: 2018-07-11 09:27:50
tags: ["Linux"]
categories: ["sci-tech"]
draft: false
toc: true
summary: "Download files on node_1 and move all of them to node_2 at intervals."
---

# Background

My cluster has two nodes:

​	node_1: Public IP, small disk space;

​	node_2: Private IP, large disk space;

So, I need to download files on node\_1 and move all of them to node\_2 at intervals.

# Method

1. After enough large files are downloaded, I just move them to node\_2;

2. Copy files' attributes to node\_1

   ```
   cp -r --attributes-only <node_1_directory> <ndoe_2_directory>
   ```

3. Download again by `-nc` option on node\_1:

   ```
   wget --no-check-certificate -nc -e robots=off -r -l inf -np -R .html,.tmp -nH --cut-dirs=3 https://ladsweb.modaps.eosdis.nasa.gov/archive/orders/501249136/ --header "Authorization: Bearer <APP_KEY>" -P ./
   ```

4. Delete empty fiels:

   ```
   find . -type f -empty -delete
   ```

5. Repeat 1-4......