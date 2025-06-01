---
weight: 999
title: "BFT"
description: ""
icon: "article"
date: "2025-06-01T16:22:12+08:00"
lastmod: "2025-06-01T16:22:12+08:00"
draft: false
toc: true
---

解压密码为hacktheblue

>在本期福尔摩斯探案中，您将深入了解 MFT（主文件表）取证技术。我们将介绍用于分析 MFT 痕迹以识别恶意活动的知名工具与方法论。在分析过程中，您将使用 MFTECmd 工具解析提供的 MFT 文件，通过 TimeLine Explorer 打开并分析解析结果，并借助十六进制编辑器从 MFT 中恢复文件内容。


>MFT，全称Master File Table，即主文件表，它是NTFS文件系统的核心。它是包含了NTFS卷中所有文件信息的数据库，在$MFT中每个文件（包括MFT本身）至少有一个MFT，记录着该文件的各种信息。这些信息被称为属性。
>
NTFS使用MFT条目定义它们对应的文件，有关文件的所有信息，比如大小、时间、权限等都存在MFT条目中，或者由MFT条目描述存储在MFT外部的空间中。
>
MFT由一个个MFT项（也称为文件记录(File Record)）组成，每个MFT项占用1024字节的空间。这个概念相当于Linux中的inode，File Record在$MFT文件中物理上是连续的，且从0开始编号，每个MFT项的前部几十个字节有着固定的头结构，用来描述本MFT项的相关信息。后面的字节存放着“属性”。(-via 百度百科)

# Task1
>西蒙·斯塔克在 2 月 13 日遭到攻击者针对。他从邮件中的链接下载了一个 ZIP 文件。请问他从该链接下载的 ZIP 文件名称是什么？
>
>Stage-20240213T093324Z-001.zip

下载工具[MftCmd](https://ericzimmerman.github.io/#!index.md)来提取MFT条目，或者直接RStudio提取文件内容也能找到(不推荐，因为文件数据流没了)

西蒙用户下载目录下有两个zip文件

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250601140624.png)

第二个的命名有213，那就是第二个

或者使用mftcmd转成csv文件再用`Time Line Explore`搜索zip文件

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250601150635.png)

# Task2
>检查最初下载的 ZIP 文件的区域标识符内容。该字段显示了文件下载来源的 HostUrl，这是我们调查/分析中重要的入侵指标(IOC)。这个 ZIP 文件的完整下载主机 URL 是什么？
>
>`https://storage.googleapis.com/drive-bulk-export-anonymous/20240213T093324.039Z/4133399871716478688/a40aecd0-1cf3-4f88-b55a-e188d5c1c04f/1/c277a8b4-afa9-4d34-b8ca-e1eb5e5f983c?authuser`


>什么是区域标识符：下载文件的时候会产生一个数据流Zone.Indentifier，里面包含了文件的来源

上一题搜索得到的结果下一条就是

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250601150647.png)

# Task3
>执行恶意代码并连接到 C2 服务器的恶意文件的完整路径和名称是什么？
>
>`C:\Users\simon.stark\Downloads\Stage-20240213T093324Z-001\Stage\invoice\invoices\invoice.bat`

可以直接R-Studio找到第一题压缩包解压出来的bat文件

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250601150605.png)

或者csv接着翻也能看见

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250601150640.png)

接着搜invoice，得到答案

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250601150696.png)

# Task4
>分析先前识别文件中$Created0x30 时间戳。该文件是何时被创建在磁盘上的？
>
>2024-02-13 16:38:39

这次只能看csv了，bat文件的时间戳

![](https://gitee.com/inklong/blog-pic/raw/master/images/HTB/Sherlock/pic/20250601150663.png)

# Task5
>在许多调查场景中，找到 MFT 记录的十六进制偏移量非常有用。请找出问题 3 中所述暂存器文件的十六进制偏移量。
>
>16E3000

条目编号乘1024即可，R-Studio也能看，右键get-info看MFT-number

23436 * 1024 = 0x16E3000

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250601150663%201.png)

# Task6
>每个 MFT 记录的大小为 1024 字节。如果磁盘上的文件小于 1024 字节，它们可以直接存储在 MFT 文件本身中。这些被称为 MFT 常驻文件。在 Windows 文件系统调查期间，查找可能驻留在 MFT 中的任何恶意/可疑文件至关重要。通过这种方式，我们可以找到恶意文件/脚本的内容。请找出问题 3 中识别的恶意暂存器的内容，并以 C2 IP 和端口作为答案。
>
>43.204.110.203:6666

R-Studio能直接还原bat文件然后查看

正常来说是找到偏移地址16E3000之后使用winhex打开源文件，然后跳转到目标偏移就能看见内容了

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250601150641.png)