---
weight: 999
title: "Meerkat"
description: ""
icon: "article"
date: "2025-05-30T15:10:35+08:00"
lastmod: "2025-05-30T15:10:35+08:00"
draft: false
toc: true
---

解压密码为hacktheblue

>作为一家快速发展的初创公司，Forela 一直在使用业务管理平台。遗憾的是，我们的文档匮乏，且管理员的安全意识较为薄弱。作为新任安全服务提供商，我们希望您能查看我们导出的部分 PCAP 和日志数据，以确认我们是否遭受（或未遭受）入侵。

# Task1
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

# Task2
>我们相信攻击者可能使用了暴力破解攻击类别的一个子集——所进行的攻击名称是什么？
>Credential Stuffing

按照info和time排序，找到爆破行为：

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423151607.png)

查看

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423151634.png)


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423151659.png)

不像是暴力，更像是撞库
# Task3
>被利用的漏洞是否分配有 CVE 编号？如果有，是哪一个？
>CVE-2022-25237

能看见访问了这个路由

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423152321.png)

搜索得到编号，[CVE-2022-25237](https://zhuanlan.zhihu.com/p/575580483)

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423152441.png)

# Task4
>攻击者的漏洞利用在 API URL 路径后附加了哪个字符串以绕过授权过滤器？
>i18ntranslation

参见上一问即可
# Task5
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
# Task6
>哪一组用户名和密码组合成功了？
>`seb.broom@forela.co.uk:g0vernm3nt`

过滤响应码200的包
```wireshark
ip.addr == 172.31.6.44 && tcp.port == 8080 && http.response.code == 200
```

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423162024.png)

# Task7
>如果有的话，攻击者利用了哪个文本分享网站？
>pastes.io


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423162751.png)

# Task8
>请提供攻击者用于在我们主机上维持持久访问的公钥文件名。
>hffgra4unv

题目翻译的有点问题，意思应该是用哪个文件写入的公钥
打开上一题找到的url能看见

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423163628.png)


# Task9
>你能确认攻击者为维持持久访问所修改的文件吗？
>/home/ubuntu/.ssh/authorized_keys

见上一题
# Task10
>您能确认这种持久化机制对应的 MITRE 技术 ID 吗？
>T1098.004


![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/Pasted%20image%2020250423164013.png)