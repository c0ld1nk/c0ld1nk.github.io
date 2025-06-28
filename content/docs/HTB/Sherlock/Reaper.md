---
weight: 999
title: "Reaper"
description: ""
icon: "article"
date: "2025-06-28T19:34:07+08:00"
lastmod: "2025-06-28T19:34:07+08:00"
draft: false
toc: true
---
解压密码为hacktheblue

>我们的 SIEM 系统检测到可疑登录事件需立即核查。警报详情显示 IP 地址与源工作站名称不匹配。现提供事件时间范围内的网络抓包记录和事件日志，请关联分析现有证据并向安全运营中心经理提交报告。

# TASK1

>Forela-Wkstn001 的 IP 地址是什么？
>172.17.79.129

搜索`Forela-Wkstn001`，能找到21号dns包返回了ip解析地址

![](https://gitee.com/inklong/blog-pic/raw/master/images/2025062818060563.png)

或者过滤`netbios name service`(名字服务。名字服务器，类似dns)

![](https://gitee.com/inklong/blog-pic/raw/master/images/2025062819065864.png)

# TASK2
>Forela-Wkstn002 的 IP 地址是什么？
>172.17.79.136

和上一题一样，找`nbns`协议

![](https://gitee.com/inklong/blog-pic/raw/master/images/2025062819063990.png)

# TASK3
>攻击者窃取的密码哈希对应的账户用户名是什么？
>arthur.kyle

136与129都是正常服务主机，只有135未知，过滤ip为135的smb2协议

>ip.addr == 172.17.79.135&&smb2

看到一个smb登录请求，就是答案

![](https://gitee.com/inklong/blog-pic/raw/master/images/2025062819063512.png)

# TASK4
>攻击者用于拦截凭证的未知设备 IP 地址是什么？
>172.17.79.135

就是上一题的ip
# TASK5
>受害者用户账户访问的文件共享是什么？
>\\DC01\Trip

去安全日志搜索用户名`172.17.79.135`，有两个，一个是访问共享目录，一个是成功登录`Forela-Wkstn002`，过滤他的ip

找到访问的共享文件夹

![](https://gitee.com/inklong/blog-pic/raw/master/images/2025062819064004.png)
# TASK6
>使用被入侵账户登录目标工作站时使用的源端口是什么？
>40252

上一题成功登录的安全日志，能看到相关信息

![](https://gitee.com/inklong/blog-pic/raw/master/images/2025062819063275.png)
# TASK7
>恶意会话的登录 ID 是什么？
>0x64A799

还是第五题的安全日志

![](https://gitee.com/inklong/blog-pic/raw/master/images/2025062819061481.png)
# TASK8
>检测依据是主机名与分配的 IP 地址不匹配。恶意登录发生的工作站名称和源 IP 地址是什么？
>FORELA-WKSTN002, 172.17.79.135

还是登录安全日志，可以发现显示的ip是135

![](https://gitee.com/inklong/blog-pic/raw/master/images/2025062819063275.png)

与在流量包内的nbns响应不一致

![](https://gitee.com/inklong/blog-pic/raw/master/images/2025062819063990.png)

可以判断ip被更改
# TASK9
>恶意登录发生的 UTC 时间是什么时候？
>2024-07-31 04:55:16

同上，减8即可

![](https://gitee.com/inklong/blog-pic/raw/master/images/2025062819060367.png)
# TASK10
>攻击者使用的恶意工具在认证过程中访问的共享名称是什么？
>`\\*\IPC$`

安全日志中查找用户名时，还有一个共享文件夹访问记录

![](https://gitee.com/inklong/blog-pic/raw/master/images/2025062819062582.png)