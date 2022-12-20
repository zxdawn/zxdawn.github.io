---
title: Mount ntfs and start nfs
date: 2018-09-04 15:01:00
tags: ["Linux"]
categories: ["sci-tech"]
draft: false
toc: true
summary: "Mount and start NFS"
---

## Mount and start NFS

If the type of disk is ntfs, you need to install `ntfs-3g`.

```
mount -t ntfs-3g /dev/??? /new_dir
#start the nfs server service.
service nfsserver start
```

## Exporting Directories on the Server
<!--more-->

`man 5 exports`

```
DESCRIPTION
       The  file  /etc/exports  contains  a table of local physical file systems on an NFS server that are accessible to NFS clients.  The contents of the
       file are maintained by the server's system administrator.

       Each file system in this table has a list of options and an access control list.   The  table  is  used  by  exportfs(8)  to  give  information  to
       mountd(8).

       The  file  format  is  similar  to the SunOS exports file. Each line contains an export point and a whitespace-separated list of clients allowed to
       mount the file system at that point. Each listed client may be immediately followed by a parenthesized, comma-separated list of export options  for
       that client. No whitespace is permitted between a client and its option list.

       Also,  each  line  may  have  one or more specifications for default options after the path name, in the form of a dash ("-") followed by an option
       list. The option list is used for all subsequent exports on that line only.

       Blank lines are ignored.  A pound sign ("#") introduces a comment to the end of the line. Entries may be continued across newlines  using  a  back-
       slash.  If  an  export  name contains spaces it should be quoted using double quotes. You can also specify spaces or other unusual character in the
       export name using a backslash followed by the character code as three octal digits.

       To apply changes to this file, run exportfs-ra or restart the NFS server.
```

Edit `/etc/exports`:

```
/new_dir *(rw,no_root_squash,no_subtree_check,async)
```

## Mounting the NFS Shares on the Client

```
mount node??:/new_dir /client_dir
```

