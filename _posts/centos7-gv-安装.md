---
title: centos7 gv 安装
date: 2019-03-21 10:48:14
tags:
categories: 软件安装
---


# [gv下载](https://centos.pkgs.org/7/epel-x86_64/gv-3.7.4-6.el7.x86_64.rpm.html)

```
[root@localhost download]# rpm -ivh gv-3.7.4-8.sdl7.x86_64.rpm
warning: gv-3.7.4-8.sdl7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 41a40948: NOKEY
error: Failed dependencies:
        /usr/bin/update-desktop-database is needed by gv-3.7.4-8.sdl7.x86_64
        libXaw3d.so.8()(64bit) is needed by gv-3.7.4-8.sdl7.x86_64

[root@localhost download]# yum install desktop-file-utils
...
...

[root@localhost download]# rpm -ivh gv-3.7.4-8.sdl7.x86_64.rpm
warning: gv-3.7.4-8.sdl7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 41a40948: NOKEY
error: Failed dependencies:
        libXaw3d.so.8()(64bit) is needed by gv-3.7.4-8.sdl7.x86_64

		
[root@localhost download]# rpm -ivh Xaw3d-1.6.2-4.sdl7.x86_64.rpm
warning: Xaw3d-1.6.2-4.sdl7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 41a40948: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:Xaw3d-1.6.2-4.sdl7               ################################# [100%]

[root@localhost download]# rpm -ivh gv-3.7.4-8.sdl7.x86_64.rpm
warning: gv-3.7.4-8.sdl7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 41a40948: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:gv-3.7.4-8.sdl7                  ################################# [100%]
[root@localhost download]# gv
gv: Unable to open the display.
[root@localhost download]# gv -h
Usage: gv [OPTION]... [FILE]
PostScript and PDF viewer.
...

```
