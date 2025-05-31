---
weight: 999
title: "Unit42"
description: ""
icon: "article"
date: "2025-05-30T15:10:35+08:00"
lastmod: "2025-05-30T15:10:35+08:00"
draft: true
toc: true
---

解压密码为hacktheblue

附件是一个evtx文件，windows事件查看器能直接打开
附上sysmon事件id表

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531190225279.png)

# Task1
>事件 ID 为 11 的日志记录共有多少条？
>
>56

筛选就能知道

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531190057112.png)

# Task2
>每当在内存中创建一个进程时，都会记录一个事件 ID 为 1 的事件，其中包含命令行、哈希值、进程路径、父进程路径等详细信息。这些信息对分析师非常有用，因为它允许我们查看系统上执行的所有程序，这意味着我们可以发现任何正在执行的恶意进程。受害者系统感染的恶意进程是什么？
>
>C:\Users\CyberJunkie\Downloads\Preventivo24.02.14.exe.exe

过滤id位1的事件，第二个的程序不正常

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531190211879.png)

# Task3
>哪种云盘被用来分发恶意软件？
>
>dropbox

查表，22是dns查询，翻id为22的事件，找到一个特殊域名

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531190520080.png)

确认是云盘

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531190532842.png)

# Task4
>对于许多写入磁盘的文件，初始恶意文件采用了一种名为“时间戳伪造”的防御规避技术，即更改文件创建日期使其显得更旧，从而与其他文件混为一体。PDF 文件的时间戳被修改为何时？
>
>2024-01-14 08:10:06.029

过滤事件id=2，搜索pdf

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531190542932.png)

日志是2-14号的，做旧就是1-14了
# Task5
>恶意文件在磁盘上释放了几个文件。"once.cmd"是在磁盘的哪个位置创建的？请提供包含文件名的完整路径作为答案。
>
>C:\Users\CyberJunkielAppData\Roaming\Photo and fax Vn\Photo and vn 1.1.2\install\F97891C\WindowsVolumel\Games\once.cmd

过滤事件id=11，搜索once.cmd

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531190555061.png)

# Task6
>该恶意文件尝试连接一个虚拟域名，很可能是为了检测网络连接状态。它试图连接的域名是什么？
>
>www.example.com

域名和dns有关，过滤事件22

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531190726738.png)

# Task7
>恶意进程试图连接的 IP 地址是哪个？
>
>93.184.216.34

过滤事件3

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531190826216.png)

# Task8
>恶意进程在感染 PC 后自行终止，该 PC 被植入了 UltraVNC 的后门变种。该进程是何时自行终止的？
>
>2024-02-14 03:41:58

过滤进程5

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531190837060.png)
