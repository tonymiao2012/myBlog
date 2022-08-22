---
title: React原理一把撸
date: 2022-08-21 13:49:00
categories: React
tags: React
---

尝试从零来自己实现撸react代码，从而更好的理解React原理。注意本文基于React 16.X版本

> 粗浅的理解：先来看看什么是虚拟DOM。虚拟dom就是由真实的html标签生成的一个描述数据结构，里面包含当前层级的props，children，key等等的信息。当触发re-render时候，由react调度机制进行当前虚拟dom节点的`diff`操作，也就是比较当前dom节点是否有改变，如果有的话再进行真实dom操作，这样大大节省了dom节点操作的开销。当然虚拟dom的形成没有光依靠react官方提供的基础库，还依赖babel插件对jsx语法进行的转译，并形成解析后的中间产物，后面代码会有体现。先看下如果简单实现一个虚拟dom的大概思路：

```javascript

// 将：<div class="div">hello world</div>转化为
/*
    { 
        type: 'div', 
        props: { 
            class: 'div',
            children: [{ type: '', props: { nodeValue: 'hello world' }}]
        }
    }
*/

const isArr = Array.isArray
const toArray = arr = isArr(arr ?? []) ? arr : [arr]
const isText = txt => typeof txt === 'string' || typeof txt === 'number'
// 递归解析层级嵌套的arr，并在根节点创建一个VNode
const flatten = arr => [...arr.map(ar => isArr(ar) 
    ? [...faltten(ar)] 
    : isText(ar) ? createTextVNode(ar) : ar)]

function h(type, props, ...kids) {
    props = props ?? {}
    kids = flatten(toArray(props.children ?? kids)).filter(Boolean)

    if(keys.length) props.children = kids.length === 1 ? kids[0] : kids
    // 提取传入的ref和key元素
    const key = props.key ?? null
    const ref = props.ref ?? null

    delete props.key
    delete props.ref

    return createVNode(type, props, key, ref)
}

function createTextVNode(text) {
    return {
        type: '',
        props: { nodeValue: text + '' }
    }
}

function createVNode(type, props, key, ref) {
    return {
        type, props, key, ref
    }
}

```

看完了上面的部分，可以再实现下HTML的部分，引用上面的函数进行VDOM创建：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <script type="module">
    import { h } from './index.js'

    console.log(
      // 引用时候，传入babel解析jsx后的标签，classname, 子元素dom描述等等。由h进行构造VDOM
      h('div',
        { className: 'container' },
        h('div', { className: 'empty' }, 'hello world!')
      )
    )
  </script>
</head>
<body>
  <div class='container'>
    <div class='empty'>
      hello world!
    </div>
  </div>
</body>
</html>
```
执行后可以看到VDom的信息结果。

#### 调度scheduler

接下来思考一下如何完成调度部分。React实现调度的原理是基于`[MessageChannel](https://developer.mozilla.org/zh-CN/docs/Web/API/MessageChannel)`这个WebAPI来实现零延时宏任务调度的。为什么不用`微任务`来做调度呢？因为微任务是优先级是高于`宏任务`的，而且触发时机是在render之前。所以并不满足调度的要求。简单看下MessageChannel是如何建立的。

我们假设有一个页面，嵌套一个iframe子页面；在主页面里有输入框，当点击按钮（`click`）后，由主页面向子页面传递消息，并在子页面`iframe`里面接收消息并输出显示。就这样一个功能。代码实现大概如下：

主页面：
```javascript
const input = document.getElementById('message-input');
const output = document.getElementById('message-output');
const button = document.querySelector('button');
const iframe = document.querySelector('iframe');

const channel = new MessageChannel();
const port1 = channel.port1;

// Wait for the iframe to load
iframe.addEventListener("load", onLoad);

function onLoad() {
  // Listen for button clicks
  button.addEventListener('click', onClick);

  // Listen for messages on port1
  port1.onmessage = onMessage;

  // 向iframe传入消息port2参数
  iframe.contentWindow.postMessage('init', '*', [channel.port2]);
}

// Post a message on port1 when the button is clicked
function onClick(e) {
  e.preventDefault();
  port1.postMessage(input.value);
}

// Handle messages received on port
function onMessage(e) {
  output.innerHTML = e.data;
  input.value = ''; //收到消息后清空input框
}

```
其中，`iframe.contentWindow.postMessage`参数说明如下：

> 1. The message being sent. For this initial port transferring this message could be an empty string but in this example it is set to 'init'.
> 2. The origin the message is to be sent to. * means "any origin".
> 3. An object, the ownership of which is transferred to the receiving browsing context. In this case, we are transferring MessageChannel.port2 to the IFrame, so it can be used to communicate with the main page.

子页面接收：
```javascript
const list = document.querySelector('ul');
let port2;

// Listen for the initial port transfer message
window.addEventListener('message', initPort);

// Setup the transferred port
function initPort(e) {
  port2 = e.ports[0];   // 获取port2传入
  port2.onmessage = onMessage;  // 监听消息，定义处理函数
}

// Handle messages received on port2
function onMessage(e) {
  const listItem = document.createElement('li');
  listItem.textContent = e.data;
  list.appendChild(listItem);
  port2.postMessage(`Message received by IFrame: "${e.data}"`); // 向port1传递已接受消息
}
```

这样就完成了从主页面到子页面相互隔离的环境下的消息通信。

进一步思考，还有个webAPI叫`requestIdleCallback`的方法，也可以用作任务调度，而且是利用空闲时间片处理任务。听上去是既能执行任务调度，又能压榨浏览器性能，分片时间就能吧任务执行完了。可是现实很残酷，没有被React团队采用也是有他的理由的。用这个API是有`50ms性能优化问题`，按照我的理解，大概为：**长任务是指执行耗时在 50ms 以上的任务，而Chrome 浏览器页面渲染和 V8 引擎用的是一个线程，如果 JS 脚本执行耗时太长，就会阻塞渲染线程，进而导致页面卡顿。**

但是我们也来理解下这个API，从它的参数开始看：

```javascript
// 接受回调任务
typeRequestIdleCallback = (cb: (deadline: Deadline) => void, options?: Options) =>number
// 回调函数接受的参数
typeDeadline = {
    timeRemaining: ()=>number// 当前剩余的可用时间。即该帧剩余时间。
    didTimeout: boolean// 是否超时。
}
```

结合上面参数，看个简单的demo:

```javascript
// 一万个任务，这里使用 ES2021 数值分隔符
const unit = 10_000;
// 单个任务需要处理如下
const onOneUnit = ()=>{
    for(let i = 0; i <= 500_000; i++) {}
}
// 每个任务预留执行时间 1ms
const FREE_TIME = 1;
// 执行到第几个任务
let _u = 0;

function cb(deadline) {
    // 当任务还没有被处理完 & 一帧还有的空闲时间 > 1ms
    while(_u < unit && deadline.timeRemaining() > FREE_TIME) {
        onOneUnit();
        _u++;
    }
    // 任务干完
    if(_u >= unit) return;
    // 任务没完成, 继续等空闲执行 
    window.requestIdleCallback(cb)
}

window.requestIdleCallback(cb)
```

故事到这里，其实还有几个备选的疑问，我在这里统一罗列下吧：

1. 调度为什么不选用`setTimeout`? -> setTimeout有坑，执行会有4ms延迟。
2. 为什么不选用`web worker`来做？ -> 浏览器底层算法，会引起`结构化克隆`。数据量大会有很大的性能问题
3. 消息调度很像`generator`的机制，为什么没有采用？用团队原话：**"The generators are stateful. You cannot resume it in the middle of it."**换句话说，generator不能中断，只能从头再来。

----------------

扯的远了，我们再来看下调度如何来实现。
```javascript
// scheduler
/**
 * schedule -> 把任务放进一个队列，然后开始（以某种节奏[ric (requestIdleCallback)]执行）
 * shouldYield -> should yield -> generator yiled. (本质上，就是返回 true/false 的函数)
 */

const queue = []
const threshold = 1000 / 60

// git transtions
const transtions = []
let deadline = 0

const now = () => performance.now()
const peek = arr => arr[0]

export function startTranstion(cb) {
  transtions.push(cb) && postMessage()
}

// 二合一，push / exec
export function schedule (cb) {
  queue.push({ cb })
  startTranstion(flush)
}

// 触发宏任务，渲染 / 宏任务
const postMessage = (() => {
  const cb = () => transtions.splice(0, 1).forEach(c => c())
  const { port1, port2 } = new MessageChannel()
  port1.onmessage = cb
  return () => port2.postMessage(null)
})()

export function shouldYield () {
  return navigator.scheduling.isInputPending() || now() >= deadline
}

function flush() {
  deadline = now() + threshold
  let task = peek(queue)

  while(task && !shouldYield()) {
    const { cb } = task
    task.cb = null
    const next = cb()

    if (next && typeof next === 'function') {
      task.cb = next
    }
    else {
      queue.shift()
    }

    task = peek(queue)
  }

  task && startTranstion(flush)
}

```

#### Fragment
很简单的原理
```javascript
export function Fragment(props) {
    return props.children
}
```