---
title: React知识点滴
date: 2022-08-15 11:59:20
categories: React
tags: 面试
---

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