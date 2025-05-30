---
weight: 999
title: "Campfire-1"
description: ""
icon: "article"
date: "2025-05-30T15:10:35+08:00"
lastmod: "2025-05-30T15:10:35+08:00"
draft: true
toc: true
---

解压密码为hacktheblue

# Task1
>分析域控制器安全日志，您能否确认发生 Kerberoasting 攻击的具体日期和时间？

这种攻击方式的流程为：
1. 发现SPN服务  
2. 使用工具向 SPN 请求 TGS 票据  
3. 转储 .kirbi 或 ccache 或服务 HASH 的 TGS 票据  
4. 将 .kirbi 或 ccache 文件转换为可破解格式  
5. 使用hashcat等工具配合字典进行暴力破解
也就是要请求tgs票据，id为4769的日志记录此事件

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250525192813.png)

翻找发现mysql服务，别的都是域控服务，大概率就是这个

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250525193845.png)

转化为utc+0时间
2024-05-21 3:18:09
# Task2
>被攻击的服务名称是什么？

见上一题
MSSQLService
# Task3
>识别发生此活动的工作站至关重要。该工作站的 IP 地址是什么？

见上一题，日志里有客户端地址
172.17.79.129
# Task4
>既然我们已经定位到该工作站，现为您提供包括 PowerShell 日志和预取文件在内的诊断数据，以便深入分析，从而了解该端点上的活动是如何发生的。用于枚举 Active Directory 对象并可能在网络中查找可 Kerberoast 账户的文件名称是什么？

打开powershell日志，4104为执行命令，依次翻找，找到绕过限制执行ps文件

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250525194419.png)

那下一条应该就是具体的文件位置

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250525194448.png)

找到文件路径
powerview.ps1
# Task5
>该脚本是何时执行的？

见上一题时间，转为utc-0
2024-05-21 03:16:32
# Task6
>用于执行实际 kerberoasting 攻击的工具完整路径是什么？

>prefetch文件夹
Windows 中的预读取文件夹，存放系统已访问的文件的预读信息，程序首次运行时会存储有关在加载该文件时访问的所有文件的信息，还存储元数据

工具采集信息在11:17:25结束，从17分开始查看prefetch文件夹

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250525195207.png)

可疑程序rubeus.exe
使用[PECmd](https://ericzimmerman.github.io/#!index.md)读取

```bash
PECmd.exe -f "C:\Users\xxx\Downloads\Triage\Workstation\2024-05-21T033012_triage_asset\C\Windows\prefetch\RUBEUS.EXE-5873E24B.pf" --csv "./"
```


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250525195859.png)


C:\Users\Alonzo.spire\Downloads\Rubeus.exe
# Task7
>该工具是何时执行以转储凭据的？

表格有最后运行时间
2024-05-21 03:18:08