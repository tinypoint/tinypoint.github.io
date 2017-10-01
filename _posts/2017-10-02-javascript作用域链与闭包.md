---
layout:     post
title:      理解js就靠它了，详解javascript中的作用域链与闭包
subtitle:   javascript中的作用域链与闭包
date:       2017-10-01
author:     tinypoint
header-img: img/home-bg-geek.jpg
catalog: true
tags:
    - 前端
    - javascript
    - 作用域链
    - 闭包
    - 面试
---

> 本文首次发布于 [tinypoint\`s blog](http://tinypoint.github.io), 作者 [@张小点(tinypoint)](http://github.com/tinypoint) ,转载请保留原文链接.

##前言

作用域链与闭包是javascript的基础也是重点，理解作用域链和闭包是你今后在js中大展拳脚的保证，所以，让我们一起来学习它们吧。
![]()

### 作用域链


#### 执行环境&变量对象
js中每一个函数都有自己的**执行环境**，而每一个执行环境又都有一个与之关联的**变量对象**，变量对象里面保存了该环境下定义的所有变量和函数。解析器会在处理数据的时候在后台使用它。

> *每个函数都有自己的执行环境，当执行流进入到一个函数时，该函数的执行环境就会被推入到一个环境栈中当。当函数执行后，栈会将该执行环境弹出，把控制权返回给之前的执行环境。*
> *<p style="text-align: right">--javascript高级程序设计</p>*   

```javascript
var color = 'black';

function fn () {
    var color = 'red';
    console.log(color);
}       
// 在此之上，环境栈中只有全局函数的执行环境

fn(); // fn函数的执行环境进入到环境栈中

//fn函数运行完毕后，他的执行环境从环境栈中弹出，将控制权交还给全局函数的执行环境
```
#### 作用域链
当代码在一个环境中执行时，会创建变量对象的一个作用域链，在作用域的前端始终是当前代码所在环境的变量对象，如果是函数，就会把他的活动对象作为变量对象。下一个变量对象则是包含环境，在下一个则是下一个包含环境，这样，一直延伸到全局执行环境。

而标识符的解析是沿着作用域链一级一级的搜索标识符的过程，也就是所变量的查找会沿着作用域链一直向上，直到找到为止。
```javascript
var color = 'black';

function fn () {
    console.log(color);
}       

fn(); // 'black'
//变量color的查找是沿着fn的活动对象的作用域链进行查找，在自己的
//活动对象中查找发现没有color变量后，会查找全局环境中的color变量

```

### 闭包

一般来说，外部环境并不能访问函数内部的变量，因为变量的查找（或者说标识符的解析是沿着作用域链向上的，方向不可逆）

正式如此我们需要引入闭包的概念：闭包是指有权访问另一个函数作用域中变量的函数。

让我们来创建一个闭包

```javascript
var color = 'black';

function createFn () {
    var color = 'blue';
    
    return function () {
        console.log(color);
    }
}

var fn = createFn();

fn(); //blue  

```
上面的fn函数就是我们说的闭包，原理其实很简单，fn函数的作用域链中，除了自己的变量对象，在上一级就是createFn的变量对象，再上一层才是全局变量对象。所以color是blue而不是black

一般来说，函数运行完该函数的活动对象就会被销毁，而闭包不同，当createFn函数运行完，虽然他的作用域链会被销毁，但是其活动对象被保存在了内存中，一直等到fn被销毁，才会被销毁，因此也就实现访问函数内部变量的功能，但是过度的闭包使用可能导致内存占用过多，请谨慎使用。


