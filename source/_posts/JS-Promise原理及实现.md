---
title: JS Promise原理及实现
date: 2021-04-12 20:12:08
categories: 深入JS
tags: 前端
---

# Promise A+规范
Promise是JS异步编程的解决方案之一。它的内部实现相当于一个状态机，具备不同的状态间切换。

## 概念

1. promise 是一个有then方法的对象或者是函数，行为遵循本规范
2. thenable 是一个有then方法的对象或者是函数
3. value 是promise状态成功时的值，也就是resolve的参数, 包括各种数据类型, 也包括undefined/thenable或者是 promise
4. reason 是promise状态失败时的值, 也就是reject的参数, 表示拒绝的原因
5. exception 是一个使用throw抛出的异常值
6. 规范详见[点这个](https://promisesaplus.com/)

## Promise三种状态

1. pending

    1.1 初始的状态, 可改变.
    1.2 一个promise在resolve或者reject前都处于这个状态。
    1.3 可以通过 resolve -> fulfilled 状态;
    1.4 可以通过 reject -> rejected 状态;

2. fulfilled

    2.1 最终态, 不可变.
    2.2 一个promise被resolve后会变成这个状态.
    2.3 必须拥有一个value值

3. rejected

    3.1 最终态, 不可变.
    3.2 一个promise被reject后会变成这个状态
    3.3 必须拥有一个reason

Tips: 总结一下, 就是promise的状态流转是这样的  
pending -> resolve(value) -> fulfilled  
pending -> reject(reason) -> rejected


