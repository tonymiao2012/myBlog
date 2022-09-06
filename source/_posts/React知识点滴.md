---
title: React知识点滴（持续更新中）
date: 2022-08-15 11:59:20
categories: React
tags: 面试
---

#### immutable JS的使用
immutable.js是Facebook推出的持久性数据结构的库。其中immutable指的是数据一旦创建，就不能再被更改，任何修改或者添加操作都会返回一个新的immutable对象。通过这个库可以让我们更容易的处理缓存、回退、数据变化检测等问题。
```javascript
import { Map } from "immutable";
const map1 = Map({ a: { aa: 1}, b: 2, c: 3});
const map2 = map1.set('b', 50);
map1 !== map2; // true
map1.get('b');  // 2
map2.get('b');  // 50
map1.get('a') === map2.get('a'); //true

```
修改了map1的属性，赋值给map2；但是他们并不是指向同一个存储空间。map1操作不会改变map2，而且map1与map2的'a'指向同一个存储空间。下面举例它的其他一些操作。
```javascript
import { fromJS, isImmutable } from "immutable";
const obj = fromJS({
  a: 'test',
  b: [1, 2, 4]
}); // 支持混合类型
isImmutable(obj); // true
obj.size(); // 2
const obj1 = obj.toJS(); // 转换成原生 `js` 类型
```
ImmutableJS 最大的两个特性就是： <strong>immutable data structures（持久性数据结构）</strong>与 <strong>structural sharing（结构共享）</strong>，持久性数据结构保证数据一旦创建就不能修改，使用旧数据创建新数据时，旧数据也不会改变，不会像原生 js 那样新数据的操作会影响旧数据。而结构共享是指没有改变的数据共用一个引用，这样既减少了深拷贝的性能消耗，也减少了内存。比如下图：

![immutable特性](../image/immutable%E5%9B%BE%E4%BE%8B.png)

左边是旧值，右边是新值，我需要改变左边红色节点的值，生成的新值改变了红色节点到根节点路径之间的所有节点，也就是所有青色节点的值，旧值没有任何改变，其他使用它的地方并不会受影响，而超过一大半的蓝色节点还是和旧值共享的。在 ImmutableJS 内部，构造了一种特殊的数据结构，把原生的值结合一系列的私有属性，创建成 ImmutableJS 类型，每次改变值，先会通过私有属性的辅助检测，然后改变对应的需要改变的私有属性和真实值，最后生成一个新的值，中间会有很多的优化，所以性能会很高。

#### 在react中，setState是异步还是同步？

样例代码：

```javascript
constructor(props) {
    super(props);
    this.state = {
      data: 'data'
    }
  }

  componentDidMount() {
    this.setState({
      data: 'did mount state'
    })

    console.log("did mount state ", this.state.data);
    // did mount state data

    setTimeout(() => {
      this.setState({
        data: 'setTimeout'
      })

      console.log("setTimeout ", this.state.data);
    })
  }
```
第一额结果输出为data，而第二个会输出setTimeout。也就是说第一个setState是异步，第二个setState又成了同步。

先说下结论吧：

只要你进入了 react 的调度流程，那就是异步的。只要你没有进入 react 的调度流程，那就是同步的。什么东西不会进入 react 的调度流程？ setTimeout setInterval ，直接在 DOM 上绑定原生事件等。这些都不会走 React 的调度流程，你在这种情况下调用 setState ，那这次 setState 就是同步的。 否则就是异步的。

而 setState 同步执行的情况下， DOM 也会被同步更新，也就意味着如果你多次 setState ，会导致多次更新，这是毫无意义并且浪费性能的。[详细解析](https://zhuanlan.zhihu.com/p/350332132)

```javascript
//setTimeout事件
import React,{ Component } from "react";
class Count extends Component{
    constructor(props){
        super(props);
        this.state = {
            count:0
        }
    }

    render(){
        return (
            <>
                <p>count:{this.state.count}</p>
                <button onClick={this.btnAction}>增加</button>
            </>
        )
    }
    
    btnAction = ()=>{
        //不能直接修改state，需要通过setState进行修改
        //同步
        setTimeout(()=>{
            this.setState({
                count: this.state.count + 1
            });
            console.log(this.state.count);
        })
    }
}

export default Count;
```

```javascript
//自定义dom事件
import React,{ Component } from "react";
class Count extends Component{
    constructor(props){
        super(props);
        this.state = {
            count:0
        }
    }

    render(){
        return (
            <>
                <p>count:{this.state.count}</p>
                <button id="btn">绑定点击事件</button>
            </>
        )
    }
    
    componentDidMount(){
        //自定义dom事件，也是同步修改
        document.querySelector('#btn').addEventListener('click',()=>{
            this.setState({
                count: this.state.count + 1
            });
            console.log(this.state.count);
        });
    }
}

export default Count;
```
#### React Hooks的使用

 > <strong>逻辑与UI分离 </strong>React官方推荐在开发中将逻辑部分与视图部分节藕，指责清晰
 > <strong>函数组件拥有State</strong> 在函数组件中，如果要实现类似的拥有state的状态，必须要将组件转为class
 > <strong>逻辑组件复用</strong> 社区一直致力于逻辑层面组件复用，像render props/HOC，但是他们有对应的问题。Hooks目前是比较理想的方案

 这里举个例子，比如要实现一个Avatar头部图标。那么如果使用render props:
```javascript
export default function App() {
    return (
        <div className="App">
        <Avatar name="头像">
            {name=> <User name={name} />}
        </Avatar>
    );
}
```

使用HOC：

```javascript
import React, { Component } from 'react';

class Avatar extends Component {
    render() {
        return <div>{this.props.name}</div>
    }
}
// 对原组件进行包装
function HOCAvatar(Component) {
    return () => <Component name="头像"/>
}

export default HOCAvatar(Avatar)
```

那么看下如果使用Hooks（代码量更少）:

```javascript
import React, { useState } from "react";

export function HooksAvatar() {
  const [name, setName] = useState("头像");
  return <>{name}</>;
}
```

下面展示如何用hooks实现一个倒数定时器：
```javascript

```