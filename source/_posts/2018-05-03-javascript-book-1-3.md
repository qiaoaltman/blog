---
layout: post
title: "javascript book chapter 1-3"
date: 2018-05-03 19:37:27 +0800
comments: true
categories: javascript
---
javascript 高级程序设计 读书笔记

章节 1-3

1 .JavaScript简介

2 .在html中使用JavaScript

3 .基本概念

<!--more-->

### 1 .javascript 简介

####一个完整的JavaScript包含:

ECMAScript 

DOM

BOM

dom 文档对象模型

dom把每个页面映射成一个多层级的节点

```
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>

</body>
</html>
```
BOM 浏览器对象模型

BOM 只处理浏览器窗口和框架

####BOM扩展：

弹出新窗口功能

移动缩放和关闭浏览器窗口的功能

提供浏览器详细信息的navigator对象

提供浏览器所加载页面的详细信息的location对象

提供用户显示器分辨率详细信息的screen对象

对cookies的支持

像XMLHttpRequest 和IE的ActiveXObject这样的自定义对象

####总结：

ECMAScript，由 ECMA-262dinyi ,提供核心语言功能

DOM 文档对象模型，提供访问和操作页面内容的方法和接口

BOM 浏览器对象模型，提供与浏览器交互的方法和接口


### 2 .在html中使用JavaScript

####<script> 标签

defer defer可以延迟加载脚本，不过与将script标标签放入body结尾处相比，更多的会使用后者，因为defer只有少数浏览器支持。

noscript 可以指定在不支持脚本的浏览器显示的替代内容。

### 3 .基本概念

规范，驼峰命名

####关键字

break 跳出循环

else 否则

new 新建一个对象

var 定义变量

case 当 switch 变量等于case 执行代码

finally try和catch之后最终执行的代码

return 返回 之后的代码不要执行了

void 无返回值

catch 捕获异常

for 循环

switch 多个case 

while 循环

continue 继续下一个循环 当前循环下的代码不执行

function 声明方法

this 当前方法被调用的对象

with 

default 默认执行

if 如果

throw 抛出异常

delete 删除

in 循环对象

try 与 catch对应

do 做一次

instanceof 检查类型

typeof 检查对象type

变量 在没有使用var 声明的时候 为全局变量

var 为局部变量

JavaScript是弱类型语言

####typeof

'undefined' 如果这个值未定义

'boolean' 如果这个值是布尔值

'string' 如果这个值是字符串

'number' 如果这个值是数值

'object' 如果这个值是对象或者null

'function' 如果这个值是函数

声明变量未初始化 这个变量的值就是 'undefined'

在保存对象的变量还没有真正的保存对象，就应该明确的让该变量保存null 值，这样做不仅可以体现出null作为空对象指针的惯例，而且有助于进一步区分null和undefined

####BOOLEAN中为FALSE的值

false 

空字符串

0 和 NAN

null

undefined

####NAN与任何值都不相等 包括NAN本身 

```
NAN == NAN  // false
```
####Number()

true和false返回1和0

数值直接返回

null 返回 0

undefined 返回NAN

####Object类型

Object的每个实例都具有以下属性和方法：

constructor 保存着用于创建当前对象的函数。

hasOwnProperty(propertyName) 用于检查给定的属性在当前对象实例中是否存在

isPrototypeOf(Object) 用于检查传入的对象是否是另一个对象的原型

propertyIsEnumerable(propertyName) 用于检查给定的属性是否能够使用for-n语句

toString() 返回对象的字符串表示

valueOf() 返回对象的字符串，数值，或布尔值表示

####操作符

递增和递减的操作是在包含他们的语句求值之后才执行的

可以向函数传递任意数量的参数，可以通过arguments对象来访问这些参数

由于不存在函数签名的特性，函数不能重载

变量在声明它们的脚本或函数中都是有定义的，变量声明语句会被提前到脚本或函数的顶部。但是，变量初始化的操作还是在原来var语句的位置执行，在声明语句之前变量的值是undefined