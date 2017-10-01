---
layout:     post
title:      javascript中的几种继承方式
subtitle:   js继承
date:       2017-10-01
author:     tinypoint
header-img: img/home-bg-art.jpg
catalog: true
tags:
    - 前端
    - javascript
    - 继承
    - 面试
---

> 本文首次发布于 [tinypoint\`s blog](http://tinypoint.github.io), 作者 [@张小点(tinypoint)](http://github.com/tinypoint) ,转载请保留原文链接.

## 前言

今天是普天同庆的大日子**十月一日国庆节**，祝福祖国国泰民安，繁荣昌盛，而且今年的十一与中秋连在一起，放假时间比过年还长，所以身边的朋友回家的回家，出去浪的出去浪，留我自己苦逼的在宿舍敲代码，不过没关系，误敲代码也要敲出风度来，让我们回顾一下在js中是如何实现继承的，话不多说，直接开始

### 原型链继承
缺点
- 引用类型被实例共享，每个实例对引用类型的操作，会直接影响到其他实例
- 父类无法传参（或者说，无法再不影响其他实例的情况下向父类传参）

```javascript
function Super() {
  this.hobby = ['ball', 'game'];
}

Super.prototype.addHobby = function (hobby) {
  this.hobby.push(hobby);
}

function Sub(name) {
  this.name = name;
}

Sub.prototype = new Super();
Sub.prototype.constructor = Sub;

var xiaoming = new Sub('xiaoming');
xiaoming.addHobby('read');
console.log(xiaoming.hobby); //"ball", "game", "read"

var xiaohong = new Sub('xiaohong');
console.log(xiaohong.hobby); //"ball", "game", "read"

```
### 借用构造函数继承
优点
- 可以向父类传参

缺点
- 构造函数老毛病，方法被每个实例创建一次，无法复用，造成内存浪费
- 父类无法传参（或者说，无法再不影响其他实例的情况下向父类传参）
- 父类原型中的方法，子类实例不可用

```javascript
function Super(name, hobby) {
    this.name = name;
    this.hobby = hobby;
}

Super.prototype.sayHi = function () {
    console.log('Hi');
}

function Sub(name, age, hobby) {
    Super.call(this, name, hobby);
    this.age = age;
}

Sub.prototype.addHobby = function (hobby) {
    this.hobby.push(hobby)
}

var a = new Sub('a', 22, ['ball', 'game']);
a.addHobby('read');
console.log(a.hobby); //"ball", "game", "read"

var b = new Sub('b', 23, ['ball', 'game']);
console.log(b.hobby); //"ball", "game"

a.sayHi(); //Error 没有此方法

```

### 组合继承（伪经典继承）

- 将原型链继承和借用构造函数继承两种技术组合到一块，发挥二者所长
- 使用原型链方法实现原型的属性和方法继承
- 使用借用构造函数实现实例属性的继承

```javascript
function Super(name, hobby) {
    this.name = name;
    this.hobby = hobby;
}

Super.prototype.sayHi = function () {
    console.log('Hi');
}

function Sub(name, age, hobby) {
    Super.call(this, name, hobby);
    this.age = age;
}

Sub.prototype = new Super();
Sub.prototype.constructor = Sub;

Sub.prototype.addHobby = function (hobby) {
    this.hobby.push(hobby)
}

var a = new Sub('a', 22, ['ball', 'game']);
a.addHobby('read');
console.log(a.hobby); //"ball", "game", "read"

var b = new Sub('b', 23, ['ball', 'game']);
console.log(b.hobby); //"ball", "game"

a.sayHi(); //'Hi'
```

### 原型式继承

将想要继承的对象放入函数中，从而不用创建自定义类型实现继承

缺点
- 依旧是引用类型共享问题

```javascript
function object(obj) {
    function F() {}
    F.prototype = obj;
    return new F();
}

var person = {
    name: 'xiaoming',
    hobby: ['ball', 'game']
}

var a = object(person);
a.name = 'xiaohong';
a.hobby.push('read');
console.log(a.hobby); //"ball", "game", "read"

var b = object(person);
b.name = 'xiaolu';
console.log(b.hobby); //"ball", "game", "read"

//上面的object函数等同于ES5中的Object.create()函数
var person = {
    name: 'xiaoming',
    hobby: ['ball', 'game']
}

var a = Object.create(person);
a.name = 'xiaohong';
a.hobby.push('read');
console.log(a.hobby); //"ball", "game", "read"

var b = Object.create(person);
b.name = 'xiaolu';
console.log(b.hobby); //"ball", "game", "read"
```

### 寄生式继承

与原型式继承紧密相关，在函数内部克隆要继承的对象，并强化新对象

缺点
- 类似构造函数模式的函数无法复用的问题

```javascript
var person = {
    name: 'xiaoming',
    hobby: ['ball', 'game']
}

function object(obj) {
    function F() {}
    F.prototype = obj;
    return new F();
}

function createSubObject (obj) {
    var clone = object(obj); //object可以替换为任意返回新对象的函数
    clone.sayHi = function () {
        console.log('sayHi');
    }
    return clone;
}

var a = createSubObject(person);
a.name = 'xiaohong';
a.sayHi();
```
### 寄生组合式继承


优点
- 解决了组合式继承的二次调用父类构造函数的问题
- 使用借用构造函数继承属性
- 使用原型链混成的形式继承方法
- 不必在指定原型而实例化父类，只需要父类原型的一个副本

```javascript
function object(obj) {
    function F() {}
    F.prototype = obj;
    return new F();
}

function Super(name) {
    this.name = name
}

Super.prototype.sayName = function () {
    console.log(this.name);
}

function Sub(name, age) {
    Super.call(this, name);
    this.age = age;
}

Sub.prototype = object(Super.prototype);
Sub.prototype.constructor = Sub;

Sub.prototype.sayAge = function () {
    console.log(this.age)
}
var a = new Sub('xiaohong', 22);

a.sayName(); //'xiaohong'  小红成功继承了父类原型的方法
```
> 参考
> - [JavaScript高级程序设计（第3版）
](https://baike.baidu.com/item/JavaScript%E9%AB%98%E7%BA%A7%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1/10576650)




