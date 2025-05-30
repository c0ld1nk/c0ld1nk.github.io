---
weight: 999
title: "Origins"
description: ""
icon: "article"
date: "2025-05-30T20:22:29+08:00"
lastmod: "2025-05-30T20:22:29+08:00"
draft: false
toc: true
---

解压密码为hacktheblue

>Forela 公司近期发生了一起重大安全事件。攻击者从其内部 S3 存储桶中窃取了约 20GB 数据，并正对该公司实施勒索。在根源分析过程中，一台 FTP 服务器被怀疑是攻击源头。调查发现该服务器同样遭到入侵，部分数据被窃取，进而导致整个网络环境遭受进一步渗透。现提供一份最小化的 PCAP 数据包文件，您的任务是寻找暴力破解和数据外泄的证据。

# Task1
>攻击者的 IP 地址是什么？
>15.206.185.207

先过滤ftp协议

![](https://gitee.com/inklong/blog-pic/raw/master/HTB/Origins-1.png)




发现就两个ip，一个内网一个公网，答案就是公网的ip
# Task2
>获取更多关于攻击者的信息至关重要，即使准确性不高。利用攻击者所用 IP 地址的地理定位数据，他们来自哪个城市？
>Mumbai

查询上一题的ip

![](https://gitee.com/inklong/blog-pic/raw/master/HTB/Origins-2.png)

# Task3
>备份服务器使用了哪种 FTP 应用程序？请输入完整名称及版本号。（格式：名称 版本）
>vsFTPd 3.0.5

前几个数据包就能看见

![](https://gitee.com/inklong/blog-pic/raw/master/HTB/Origins-3.png)

# Task4
>攻击者已对服务器发起暴力破解攻击。这次攻击是何时开始的？
>2024-05-03 04:12:54

很明显的爆破行为，前几个都是一个时间，点卡查看时间转化为UTC即可

![](https://gitee.com/inklong/blog-pic/raw/master/HTB/Origins-4.png)

# Task5
>攻击者用于获取访问权限的正确凭证是什么？（格式：用户名:密码）
>forela-ftp:ftprocks69$

一直翻数据包，找到有`Login successful`的
追踪这个数据包就能找到

![](https://gitee.com/inklong/blog-pic/raw/master/HTB/Origins-5.png)

# Task6
>攻击者从服务器窃取了文件。用于下载远程文件的 FTP 命令是什么？
>RETR

462号数据包开始

![](https://gitee.com/inklong/blog-pic/raw/master/HTB/Origins-6.png)

以二进制模式开始传输数据，下面的FTPdata就是传输的文件
# Task7
>攻击者成功获取了备份 SSH 服务器的凭证。该 SSH 服务器的密码是什么？
>\*\*B@ckup2024!**

可以发现ftp下载了一个文件

![](https://gitee.com/inklong/blog-pic/raw/master/HTB/Origins-7.png)

文件-导出对象-FTP-DATA

![](https://gitee.com/inklong/blog-pic/raw/master/HTB/Origins-8.png)

# Task8
>2023 年数据存档的 S3 存储桶 URL 是什么？
>https://2023-coldstorage.s3.amazonaws.com

和上一题差不多的方法，找到

![](https://gitee.com/inklong/blog-pic/raw/master/HTB/Origins-9.png)

往下翻就能找到response的值，494号包

![](https://gitee.com/inklong/blog-pic/raw/master/HTB/Origins-10.png)

# Task9
>此次事件影响范围巨大，Forela 公司的 s3 存储桶同样遭到入侵，数 GB 数据被窃取并泄露。调查还发现攻击者通过社会工程学手段获取敏感数据并进行勒索。攻击者在钓鱼邮件中用于获取 s3 存储桶敏感数据的内部邮箱地址是什么？
>archivebackups@forela.co.uk

和上一题在一块，就这一个邮箱地址