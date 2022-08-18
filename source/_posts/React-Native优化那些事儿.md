---
title: React Native优化那些事儿
date: 2022-08-15 11:18:02
categories: ReactNative
tags: react native
---

总结一下常用的RN优化的一些经验手段

#### 减少re-render

先了解下RN的生命周期
![生命周期](../image/RN%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

当组件的state和props发生改变后，re-render过程会被触发。这个阶段，可以借助[shouldComponentUpdate](https://react.docschina.org/docs/optimizing-performance.html#shouldcomponentupdate-in-actionsx)来避免不必要的render，从而提升性能。
```javascript
class Button extends React.Component {
  shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    return false;
  }

  render() {
    return <button color={this.props.color} />;
  }
}
```
大部分情况下，可以使用React.PureComponent来替代手写shouldComponentUpdate。但它只进行浅比较，所以当 props 或者 state 某种程度是可变的话，浅比较会有遗漏，那你就不能使用它了。所以涉及数据嵌套层级过多时，比如说你 props 传入了一个两层嵌套的 Object，这时候 shouldComponentUpdate 就很为难了

另一个选择就是使用React新加入的一个能力，专门针对函数组件的高阶组件：[React.memo](https://react.docschina.org/docs/react-api.html#reactmemo)

```javascript
const memoButton = React.memo(function myComponent(props) {
  // 使用props进行渲染
  return <button color={this.props.color} />;
});
```
值得注意的是，默认情况下React.memo是进行浅比较的。因为它的实现就是高阶组件，在原有的组件进行了一层封装。

#### 减少渲染压力
1. 减少嵌套层级数量（React.Fragment）
2. 减少GPU过度绘制


#### 预加载

预加载优化，主要分几个关键的阶段：

1. 终端页面
2. 引擎加载
3. JS包加载
4. JS执行
5. 渲染页面

### Bundle加载过程：
