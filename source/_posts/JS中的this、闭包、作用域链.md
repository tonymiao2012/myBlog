---
title: JS中的this、闭包、作用域链
date: 2022-08-19 14:31:58
categories:
tags:
---

# 【转】this 指向 ｜作用域与闭包

> 波比 - 资深航空母舰拼接工程师｜一级螺丝工
>



---

大纲

- 实战是检验真理的唯一标准
- 深入理解 `this`
- 作用域
- 闭包到底是什么

---



## `this` 问题总结



这一节课程，以实战为引子，带领大家一起总结出 `this` 指向问题的规律。



### 默认绑定（函数直接调用）



非严格模式下：

```js
function fn() {
  console.log(this) // window变量。函数直接执行时候this指向全局
}

fn()
```



严格模式下：

```js
function fn() {
  'use strict'
  console.log(this) // console what ?
}

fn()
```



> TIP1 👉 非严格模式下，默认绑定指向全局（`node` 中式 `global`）



当然，面试官可能会这样迷惑一下你：



```js
var a = 1
function fn() {
  var a = 2
  console.log(this.a) // 等同于window.a, 输出1
}
fn()

// -------------- 变 -----------------
// 1. 把最外层 var a = 1 -> let a = 1，输出结果是？
// Answer: undefined. let 声明了自己的作用域，没有绑定到全局 
// -------------- 变 -----------------
var b = 1
function outer () {
  var b = 2
  function inner () { 
    console.log(this.b) // console what ?
  }
  inner()
}

outer()

// --------------- 变 -----------------
const obj = {
  a: 1,
  fn: function() {
    console.log(this.a)
  }
}

obj.fn() // console 1, fn作用域绑定到obj，所以执行时候为1
const f = obj.fn
f() // undefined，相当于函数直接调用。访问全局的a, undefined
```



### 隐式绑定（属性访问调用）



这种也是非常常见的情况



```js
function fn () {
  console.log(this.a)
}

const obj = {
  a: 1
}

obj.fn = fn
obj.fn() // console 1. 隐式数据绑定。
```



> TIP 👉 隐式绑定的 `this` 指的是调用堆栈的**上一级**（`.`前面**一**个）



```js
function fn () {
  console.log(this.a)
}

const obj1 = {
  a: 1,
  fn
}

const obj2 = {
  a: 2,
  obj1
}

obj2.obj1.fn() // console 1 指向调用堆栈的上一级
```



面试官一般问的是一些边界 `case`，比如隐式绑定失效（列举部分）：

```js
// 第一种 是前面提过的情况
const obj1 = {
  a: 1,
  fn: function() {
    console.log(this.a)
  }
}

const fn1 = obj1.fn // 将引用给了 fn1，等同于写了 function fn1() { console.log(this.a) }
fn1() // 所以这里其实已经变成了默认绑定规则了，该函数 `fn1` 执行的环境就是全局环境

// 第二种 setTimeout
setTimeout(obj1.fn, 1000) // 这里执行的环境同样是全局

// 第三种 函数作为参数传递
function run(fn) {
  fn()
}
run(obj1.fn) // 这里传进去的是一个引用

// 第四种 一般匿名函数也是会指向全局的
var name = 'The Window';
var obj = {
    name: 'My obj',
    getName: function() {
        return function() { // 这是一个匿名函数
            console.log(this.name)
        };
    }
}
obj.getName()()

// 第五种 函数赋值也会改变 this 指向，下边练习题会有 case，react 中事件处理函数为啥要 bind 一下的原因
// 第六种 IIFE
```



### 显式绑定（`call`、 `bind`、 `apply`）

通过显式的一些方法去强行的绑定 `this` 上下文

```js
function fn () {
  console.log(this.a)
}

const obj = {
  a: 100
}

fn.call(obj) // console 100，显示绑定关系。强行绑定了obj的上下文环境
```



> TIP 👉 这种根本还是取决于第一个参数
>
> 但是第一个为 `null` 的时候还是绑到全局的



`bind` 这里单拎出来，因为面试常常问是吧



```js
function fn() {
  console.log(this)
}

// 为啥可以绑定基本类型 ?
// boxing(装箱) -> (1 ----> Number(1))
// bind 只看第一个 bind（堆栈的上下文，上一个，写的顺序来看就是第一个）
fn.bind(1).bind(2)() // console 结果为1
```



看看实现也很有趣（`FROM MDN`）：

```js
//  Yes, it does work with `new (funcA.bind(thisArg, args))`
if (!Function.prototype.bind) (function(){
  var ArrayPrototypeSlice = Array.prototype.slice; // 为了 this
  Function.prototype.bind = function(otherThis) {
    // 调用者必须是函数，这里的 this 指向调用者：fn.bind(ctx, ...args) / fn
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var baseArgs= ArrayPrototypeSlice.call(arguments, 1), // 取余下的参数
        baseArgsLength = baseArgs.length,
        fToBind = this, // 调用者
        fNOP    = function() {}, // 寄生组合集成需要一个中间函数，避免两次构造
        fBound  = function() {
          // const newFn = fn.bind(ctx, 1); newFn(2) -> arguments: [1, 2]
          baseArgs.length = baseArgsLength; // reset to default base arguments
          baseArgs.push.apply(baseArgs, arguments); // 参数收集
          return fToBind.apply( // apply 显示绑定 this
            // 判断是不是 new 调用的情况，这里也说明了后边要讲的优先级问题      
            fNOP.prototype.isPrototypeOf(this) ? this : otherThis, baseArgs
          );
        };
		// 下边是为了实现原型继承
    if (this.prototype) { // 函数的原型指向其构造函数，构造函数的原型指向函数
      // Function.prototype doesn't have a prototype property
      fNOP.prototype = this.prototype; // 就是让中间函数的构造函数指向调用者的构造
    }
    fBound.prototype = new fNOP(); // 继承中间函数，其实这里也继承了调用者了

    return fBound; // new fn()
  };
})();
```



### new

`new` 也是灰常有趣，我们这里只关注其 `this` 指向的问题



这次，我们先实现一下

```js
// new 关键字会进行如下的操作：

// 1. 创建一个空的简单JavaScript对象（即{}）；
// 2. 链接该对象（设置该对象的constructor）到另一个对象 ；
// 3. 将步骤1新创建的对象作为this的上下文 ；// 🔥
// 4. 如果该函数没有返回对象，则返回this。

// 我们来模拟实现一个 new
// new Fn(); 
// myNew(Fn, ...args);
import _ from 'lodash';

function myNew(fn, ...args) {
  // fn 必须是一个函数
  if (typeof fn !== 'function') throw new Error('fn must be a function.')
  // es6 new.target
  myNew.target = fn
  // 原型继承
  const temp = Object.create(fn.prototype) // 步骤 1. 2.
  // fn执行绑定 this 环境
  const res = fn.apply(temp, ...args) // 步骤 3.
  // 如果该函数没有返回对象，则返回this。
  return _.isObject(res) ? res : temp
}

```



> TIP 👉 如果函数 `constructor` 里没有返回对象的话，`this` 指向的是 `new` 之后得到的实例



```js
function foo(a) {
  this.a = a
}

const f = new foo(2)
f.a // console what?

// ------------------------- 变 ---------------------------
function bar(a) {
  this.a = a
  return {
    a: 100
  }
}
const b = new bar(3)
b.a // console 100
```



### 箭头函数

箭头函数这种情况比较特殊，编译期间确定的上下文，不会被改变，哪怕你 `new`，指向的就是**上一层**的上下文,



> TIP 👉 箭头函数本身是没有 `this` 的，继承的是外层的



```js
function fn() {
  return {
    b: () => {
      console.log(this)
    }
  }
}

fn().b() // console what? => window变量
fn().b.bind(1)() // console what? => window变量
fn.bind(2)().b.bind(3)() // console what? => Number(2)
```



### 学习

这里只是抛砖引玉的给大家介绍一些常见的情况，更多更细节的内容还是需要大家主动的去总结，去实践，可以看看[更多可参见 MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)



### 优先级



这上面的各种方式一定是有先后顺序的，同时作用于一个函数的时候，以哪一个为准呢？这取决于优先级



```js
// 隐式 vs 默认 -> 结论：隐式 > 默认
function fn() {
  console.log(this)
}

const obj = {
  fn
}

obj.fn() // what ?

// 显式 vs 隐式 -> 结论：显式 > 隐式
obj.fn.bind(5)() // what ?

// new vs 显式 -> 结论：new > 显式
function foo (a) {
    this.a = a
}

const obj1 = {}

var bar = foo.bind(obj1)
bar(2)
console.log(obj1.a) // what ?

// new
var baz = new bar(3)
console.log( obj1.a ) // what ?
console.log( baz.a ) // what ?

// 箭头函数没有 this，比较没有意义
```



> TIP 👉 优先级「new 绑」 > 「显绑」 > 「隐绑」 > 「默认绑定」



### 实战一波

网上随便找了几道题给大家练手

```js
// 1.
function foo() {
  console.log( this.a ) // console what -> 2，相当于全局执行
}
var a = 2;
(function(){
  "use strict" // 迷惑大家的
  foo();
})();

// 2.
var name="the window"

var object={
  name:"My Object", 
  getName: function(){ 
    return this.name
  } 
}
object.getName() // console what => my object
(object.getName)() // console what => my object
(object.getName = object.getName)() // console what => the window (=运算符丢失了this指向)
(object.getName, object.getName)() // console what => the window

// 3.
var x = 3
var obj3 = {
  x: 1,
  getX: function() {
    var x = 5
    return function() {
      return this.x
    }(); // ⚠️
  }
}
console.log(obj3.getX()) // console what? => 3   IIFE函数丢失this指向，所以指向全局x

// 4. 
function a(x){
  this.x = x
  return this
}
var x = a(5) // 替换为 let 再试试
var y = a(6) // 替换为 let 再试试 // 再换回 var，但是去掉 y 的情况，再试试

console.log(x.x) // console what ? => undefined
console.log(y.x) // console what ? => 6

// 等价于
window.x = 5;
window.x = window;

window.x = 6;
window.y = window;

console.log(x.x) // void 0 其实执行的是 Number(6).x
console.log(y.x) // 6
```



### 再骚气一点

作业：尝试着去读一读，理解一下

[this keyword](https://tc39.es/ecma262/#sec-this-keyword)



## 作用域和闭包



上边基础的部分完成了，我们开始抽象一下了。



### 存储空间、执行上下文

👉 数据是怎么存的？

> 本质是将数据映射成 `0` `1` ，然后通过触发器存储这类信息（电信号）



👉 栈 和 堆 / 静态内存分配 和 动态内存分配

堆栈这里指的是存储数据结构，当然本身也可以是一种数据结构的概念（二叉堆、栈）

> 静态内存分配:
>
> - 编译期知道所需内存空间大小。
> - 编译期执行
> - 申请到栈空间
> - FILO(先进后出)



> 动态内存分配:
>
> - 编译期不知道所需内存空间大小
> - 运行期执行
> - 申请到堆空间
> - 没有特定的顺序



```rust
// rust -> wasm
fn main(){
   let arr:[i32;4] = [10,20,30,40];
   println!("array is {:?}",arr);
   println!("array size is :{}",arr.len());

   for index in 0..4 {
      println!("index is: {} & value is : {}",index,arr[index]);
   }
}
```



```js
// js
function main() {
  let arr = [10,20,30,40];
  // ...
}
```





👉 执行上下文 和 可执行代码

> Execution context (abbreviated form — EC) is the abstract concept used by ECMA-262 specification for typification and differentiation of an executable code. 
>
> ​																																	-------- ECMA262



当控制器转到一段**可执行代码**的时候就会进入到一个**执行上下文**。执行上下文是一个堆栈结构(先进后出), 栈底部永远是全局上下文，栈顶是当前活动的上下文。其余都是在等待的状态，这也印证了`JS`中函数执行的原子性

可执行代码与执行上下文是相对的，某些时刻二者等价



>  可执行代码（大致可以这么划分）：
>
> - 全局代码
> - 函数
> - eval



🤔 递归 & 尾调用优化



[![gEdBIU.png](https://z3.ax1x.com/2021/04/30/gEdBIU.png)](https://imgtu.com/i/gEdBIU)



执行上下文（简称 `EC`）中主要分为三部分内容：

- `VO` / ` AO` 变量对象
- 作用域链
- `This` 



所以这个流程可以梳理出来：

1. 遇到可执行代码

2. 创建一个执行上下文 （可执行代码的生命周期：编译、运行）

   2.1 初始化 `VO`

   2.2 建立作用域链

   2.3 确定 `This` 上下文

3. 可执行代码执行阶段

   3.1 参数、变量赋值、提升

   3.2 函数引用

   3.3 ...

4. 出栈



👉 作用域链

> 每一个执行上下文都与一个作用域链相关联。作用域链是一个对象组成的链表，**求值标识符**的时候会搜索它。当控制进入执行上下文时，就根据代码类型创建一个作用域链，并用初始化对象（`VO/AO`）填充。执行一个上下文的时候，其作用域链只会被 `with` 声明和 `catch` 语句所影响



体会一下

```js
var a = 20;
function foo(){
    var b = 100;
    alert( a + b );
}
foo();

// 两个阶段：创建 - 执行

// --------------------------- 创建 ------------------------------

// 模拟 VO/AO 对象
AO(foo) {
  b: void 0
}

// [[scope]] 不是作用域链，只是函数的一个属性（规范里的，不是实际实现）
// 在函数创建时被存储，静态（不变的），永远永远，直到函数被销毁
foo.[[scope]]: [VO(global)]

VO(global) {
  a: void 0,
  foo: Reference<'foo'>
}
  
// --------------------------- 调用 ------------------------------
  
// 可以这么去理解，近似的用一个 concant 模拟，就是将当前的活动对象放作用域链最前边
Scope = [AO|VO].concat([[Scope]])
  
  
// ---------------------------- 执行时 EC --------------------------------
EC(global) {
  VO(global) {
    a: void 0,
    foo: Reference<'foo'>
  },
  Scope: [VO(global)]，
  // this
}
  
EC(foo) {
  AO(foo) { // 声明的变量，参数
    b: void 0
  },
  Scope: [AO(foo), VO(global)] // 查找顺序 -> RHS LHS  
}
```



特殊情况：

- `Function`  构造的函数 `[[scope]]` 里只有全局的变量对象

```js
// 证明
var a = 10;

function foo(){
  var b = 20;
  // 函数声明
  function f1(){ // EC(f1) { Scope: [AO(f1), VO(foo), VO(g)] }
    console.log(a, b);
  }

  // 函数表达式
  var f2 = function(){
    console.log(a, b);
  }

  var f3 = Function('console.log(a,b)')

  f1(); // 10, 20
  f2(); // 10, 20
  f3(); // 10, b is not defined
}

foo();
```



- `with` & `catch` & `eval`



> 本质上 `eval` 之类的恐怖之处是可以很方便的修改作用域链，**执行完后又回归最初状态**

```js
// 这样好理解
Scope = [ withObj|catchObj ].concat( [ AO|VO ].concat( [[ scope ]] ) )
// 初始状态 [VO(foo), VO(global)]
// with 一下：[VO(with)❓, VO(foo), VO(global)]
// with 完事儿了，还要恢复 👈
```



```js
var a = 15, b = 15;

with( { a: 10 } ){
  var a = 30, b = 30;
  alert(a); // 30
  alert(b); // 30
}

alert(a); // ? answer: 15
alert(b); // 30
```



👉 闭包



> 函数拥有对其词法作用域的访问，哪怕是在当前作用域之外执行

> 对于现代浏览器机制来说，闭包其实就是`逃逸分析` -> 通过浏览器的机制，哪些变量用了，哪些没用，哪些未来还会用。对于没有用的，浏览器执行垃圾回收。有引用的，浏览器不会立刻回收，保留这个被引用的对象。

闭包的数据存哪里？-> 堆区存储

存的其实是啥？









