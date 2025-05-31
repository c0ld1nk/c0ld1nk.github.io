---
weight: 999
title: "Noxious"
description: ""
icon: "article"
date: "2025-05-30T15:10:35+08:00"
lastmod: "2025-05-30T15:10:35+08:00"
draft: true
toc: true
---

解压密码为hacktheblue

>入侵检测系统（IDS）设备发出警报，提示内部 Active Directory 网络中可能存在恶意设备。该系统还监测到异常的 LLMNR 协议流量，怀疑发生了 LLMNR 投毒攻击。异常流量指向 IP 地址为 172.17.79.136 的 Forela-WKstn002 工作站。作为网络取证专家，您已获得事发时段的部分数据包捕获文件。鉴于事件发生在 Active Directory 虚拟局域网内，建议结合 AD 攻击向量开展网络威胁狩猎，重点关注 LLMNR 投毒攻击。
# Task1
>安全团队怀疑 Forela 内网中存在运行 Responder 工具的恶意设备实施 LLMNR 投毒攻击。请定位该恶意设备的 IP 地址。
>
>172.17.79.135

LLMNR协议在UDP 5355，所以过滤`udp.port==5355`

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531192540557.png)

能看到`172.17.79.135`对工作站`172.17.79.136`的LLMNR查询进行了回应，冒充了dc
答案就是172.17.79.135
# Task2
>被入侵机器的主机名是什么？
>
>kali

过滤此ip，`ip.addr==172.17.79.135`，有一个dhcp请求分配动态ip，里面有主机名称

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531192553900.png)


答案是kali
# Task3
>现在我们需要确认攻击者是否获取了用户哈希值且该哈希可被破解！！被捕获哈希的用户名是什么？
>
>john.deacon

还是过滤上面的ip，有一条登录cookie数据包，说明攻击者成功破解密码并破解

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531192606252.png)


答案是john.deacon
# Task4
>在 NTLM 流量中可以看到受害者的凭证被多次中继到攻击者的机器上。这些哈希值首次是在何时被捕获的？
>
>2024-06-24 11:18:30

ntml协议通常使用HTTP、SMB、SMTP，查看协议占比可知smb2占比最多

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531192614238.png)

过滤smb2协议
搜索dcc01，能找到工作站请求NTLM认证。从这里开始进行中间人劫持哈希。

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531192623997.png)

答案是2024-06-24 11:18:30（ 注意是 UTC+0）
# Task5
>受害者在访问文件共享时输入了哪个拼写错误导致其凭据泄露？
>
>DCC01

task1可以看到请求了很多次dcc01
# Task6
>要获取受害用户的实际凭据，我们需要将 NTLM 协商数据包中的多个值拼接起来。NTLM 服务器质询值是多少？
>
>601019d191f054f1

查看这个包，9291，他进行了第一次ntml challenge，是攻击者转发的流量

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531192633961.png)

找到答案

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531192644409.png)


# Task7
>现在执行类似操作，找到 NTProofStr 的值。
>
>c0cc803a6d9fb5a9082253a04dbd4cd4

下一个客户端回复包就有

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531192654208.png)


# Task8
>要测试密码复杂度，可尝试从数据包捕获的信息中恢复密码。这是关键步骤，通过这种方式我们能发现攻击者是否成功破解密码以及破解速度。
>
>`NotMyPassword0k?`

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

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531192703895.png)

# Task9
>为了更全面地了解事件背景，受害者当时试图访问的具体文件共享是什么？
>
>`\\DC01\DC-Confidential`

第一个访问的共享文件夹

![](https://gitee.com/inklong/blog-pic/raw/master/images/20250531192713836.png)



参考文章：[0xdf/HTB Sherlock: Noxious](https://0xdf.gitlab.io/2024/09/04/htb-sherlock-noxious.html)
