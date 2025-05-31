---
weight: 999
title: "Bumblebee"
description: "asdasda"
icon: "article"
date: "2025-05-30T15:10:35+08:00"
lastmod: "2025-05-30T15:10:35+08:00"
draft: true
toc: true
---

解压密码为hacktheblue

>一名外部承包商通过访客 Wi-Fi 访问了 Forela 的内部论坛，并且似乎窃取了管理员用户的凭证！我们附上了一些论坛日志和完整的 sqlite3 格式数据库转储文件，以协助您的调查。

# Task1
>该外部承包商的用户名是什么？
>
>apoole

提示结尾有个1，在数据库50多项有俩，试了一下第二个是

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531190938380.png)

# Task2
>承包商使用哪个 IP 地址创建了他们的账户？
>
>10.10.0.78


![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531190948848.png)

# Task3
>承包商发布的恶意帖子的 post_id 是什么？
>
>9

phpbb_posts数据库里找


![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531191008396.png)

# Task4
>凭证窃取程序发送数据的完整 URI 是什么
>
>http://10.10.0.78/update.php

上一题的posts内容复制本地html打开，login标签找到

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531191102699.png)

# Task5
>承包商何时以管理员身份登录论坛？（UTC 时间
>
>26/04/2023 10:53:12

在phpbb_log找到管理员登录记录，在最后

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531191112172.png)

登录操作为post，过滤找到最底下的记录

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531191126822.png)

第一个应该就是登录记录26/Apr/2023:11:53:12 +0100，转换成UTC时间减一即可
# Task6
>论坛中有 LDAP 连接的明文凭证，密码是什么？
>
>Passw0rd1

在phpbb_config里面找到

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531191140036.png)

# Task7
>管理员用户的 User Agent 是什么？
>
>Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36

找时间靠前的adm访问记录即可

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531191152464.png)

# Task8
>承包商何时将自己加入管理员组？（UTC 时间
>
>26/04/2023 10:53:51

log数据库找到

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531191236093.png)

# Task9
>承包商在什么时间下载了数据库备份？（UTC 时间）
>
>26/04/2023 11:01:38

日志搜backup，能找到get一个gz

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531191247810.png)

# Task10
>根据 access.log 记录，数据库备份的大小是多少字节？
>
>34707

和上一题一个位置