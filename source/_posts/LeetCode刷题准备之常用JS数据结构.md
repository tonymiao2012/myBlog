---
title: LeetCode刷题准备之常用JS数据结构
date: 2020-10-12 22:54:53
categories: LeetCode
tags: 数据结构
---

#### 1. JS基本数据类型
基本类型（值类型）：Number, String, Boolean, Symbol, null, undefined。在内存中占据固定大小，保存在堆栈中

引用类型：Object, Function, Array, Date, RegExp, 特殊的基本包装类型（String, Number, Boolean）以及单位内置对象（Global, Math）等引用类型的值是对象 保存在对内存中，栈内存存储的是对象的变量标识符以及对象在堆内存中存储的地址。

#### 2. JS中的类型检测

##### typeof

``` javascript
console.log(typeof 1);               // number
console.log(typeof true);            // boolean
console.log(typeof 'mc');            // string
console.log(typeof Symbol)           // function
console.log(typeof function(){});    // function
console.log(typeof console.log());   // undefined
console.log(typeof []);              // object 
console.log(typeof {});              // object
console.log(typeof null);            // object
console.log(typeof undefined);       // undefined

```
Pros: 能够快速区分基本的数据类型

Cons: typeof不能将Object、Array、Null进行区分，返回为object

##### instanceof

``` javascript
console.log(1 instanceof Number);                    // false
console.log(true instanceof Boolean);                // false 
console.log('str' instanceof String);                // false  
console.log([] instanceof Array);                    // true
console.log(function(){} instanceof Function);       // true
console.log({} instanceof Object);                   // true

```

Pros: 能够区分Array, Object和Function

Cons: 基本数据类型不能区分

##### Object.prototype.toString.call()

``` javascript
var toString = Object.prototype.toString;
console.log(toString.call(1));                      //[object Number]
console.log(toString.call(true));                   //[object Boolean]
console.log(toString.call('mc'));                   //[object String]
console.log(toString.call([]));                     //[object Array]
console.log(toString.call({}));                     //[object Object]
console.log(toString.call(function(){}));           //[object Function]
console.log(toString.call(undefined));              //[object Undefined]
console.log(toString.call(null));                   //[object Null]

```

Pros: 类型的精确判断

Cons: 调用方式复杂


``` javascript

let greet = { 
    text: "hello",
    sayhi: function() {console.log(this.text)},
    sayBye: function() {console.log("bye")},
    set: na => {
        this.text = na
    }
}

    let greet1 = {
    text: "hello",
    sayhi: function() {console.log(this.text)},
    sayBye: function() {console.log("bye")},
    set: function(na) {
        this.text = na
    }
}
```
在调用greet1/greet.set('world')后

其中，展开的值是：
``` javascript
greet
{text: "hello", sayhi: ƒ, sayBye: ƒ, set: ƒ}
sayBye: ƒ ()
sayhi: ƒ ()
set: na => { this.text = na }
text: "hello"
__proto__: Object


greet1
{text: "world", sayhi: ƒ, sayBye: ƒ, set: ƒ}
sayBye: ƒ ()
sayhi: ƒ ()
set: ƒ (na)
text: "world"
__proto__: Object

// 关于隐式屏蔽
var anotherObject = {
    a: 2
}

var myObject = Object.create(anotherObject)

anotherObject.a;
myObject.a;

anotherObject.hasOwnProperty("a")   // true
myObject.hasOwnProperty("a")        // false

myObject.a++;       // 2
myObject.a;         // 3, 获取a属性值，做＋操作，并且用put将3赋值给myObject中新建的屏蔽属性a

myObject.hasOwnProperty("a")        // true

// 当new出一个对象后，发生了什么呢，比如：
function foo() {
    // ....
}

var a = new foo()
Object.getPrototypeOf(a) === foo.prototype  //true
```
对于new来调用的函数，或者说发生构造函数调用时，会自动执行如下操作：
1. 创建（或者说构造）一个全新的对象
2. 这个新对象会被执行"Prototype"连接
3. 这个对象会被绑定到函数调用的this
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象

所以对于上面的一步，就是将a内部的Prototype连接到foo.prototype所指向的对象。
