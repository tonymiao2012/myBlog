---
title: React Native优化那些事儿(持续更新中)
date: 2022-08-15 11:18:02
categories: ReactNative
tags: react native
---

总结一下常用的RN优化的一些经验手段

#### 减少re-render

##### 1.shouldComponentUpdate
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

##### 2.React.memo
另一个选择就是使用React新加入的一个能力，专门针对函数组件的高阶组件：[React.memo](https://react.docschina.org/docs/react-api.html#reactmemo)

```javascript
const memoButton = React.memo(function myComponent(props) {
  // 使用props进行渲染
  return <button color={this.props.color} />;
});
```
值得注意的是，默认情况下React.memo是进行浅比较的。因为它的实现就是高阶组件，在原有的组件进行了一层封装。当然，如果想和shouldComponentUpdate一样加入比较，可以在使用时候传入比较函数：
```javascript
function Button(props) {
  return <button color={this.props.color} />;
}
function areEqual(prevProps, nextProps) {
  if (prevProps.color !== nextProps.color) {
      return false;
    }
  return true;
}
export default React.memo(MyComponent, areEqual);
```

##### 3.React.PureComponent
[官方文档](https://react.docschina.org/docs/react-api.html#reactpurecomponent)，举例：
```javascript
class PureButton extends React.PureComponent {
  render() {
    return <button color={this.props.color}/>
  }
}
```
React.PureComponent在组件更新前对props和state进行一次**浅比较**，当数据有多层嵌套时候，并不是很适用。这种情况建议：
1. 将组件拆分为更小粒度的组件，传入数据尽量不包含嵌套；组件内部统一用PureComponent进行渲染
2. 使用Immutable.js来配合PureComponent，进行渲染优化。见[有赞实践](https://juejin.cn/post/6844903501290536967#heading-4)
...

总之，这部分也要充分结合业务需求，来进行定制化的优化。

##### 4. React.useMemo和React.useCallback

在React 16版本后，引入了[useMemo](https://zh-hans.reactjs.org/docs/hooks-reference.html#usememo)和[useCallback](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecallback)，同样可以避免每次渲染时都进行的高开销计算。
```javascript
//返回一个memoized值，把“创建”函数和依赖项数组作为参数传入 useMemo，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);

//返回一个memoized回调函数。把内联回调函数及依赖项数组作为参数传入 useCallback，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。
const memoizedCallback = useCallback(
  () => {
    yourLogic(a, b);
  },
  [a, b]
);
```

##### 5. redux状态管理使用
**redux**作为React的状态管理，可以对项目数据提供store。但是随项目复杂度提升，引用的组件和依赖的redux状态越来越多，这会导致一个依赖redux子状态改动，会使整个页面re-render，即使这个字状态只占页面的很小一部分。

应对这种情况，需要将redux的包装由page层级改为组件层级，这样就会把组件重回引起的性能影响控制到最小：
![redux包装](../image/redux%E6%8B%86%E8%A7%A3.png)

#### 减少渲染压力
RN的布局系统底层依赖[Yoga]()这个跨平台的布局库，将虚拟DOM映射到原生的布局节点上的。但是这不意味着每个Virtual DOM会映射一个真实DOM。来看这个案例：
```javascript
render() {
  return {
    <View>
      <View style={{backgroundColor: 'orange'}}>
        <View style={{backgroundColor: 'yellow'}}>
          <Text>Card1</Text>
        </View>
      </View>
    </View>
    <View style={{backgroundColor: 'orange'}}>
      <View>
        <Text>Card2</Text>
      <View>
    </View>
  }
}
```
使用react-devtools可以看到页面的嵌套层级，代码结构是一一对应的：
![代码层级](../image/%E4%BB%A3%E7%A0%81%E5%B1%82%E7%BA%A7.png)
我们再看看 React Native 渲染到原生视图后的嵌套层级（iOS 用 Debug View Hierarchay，Android 用 Layout Inspector）：
![原生视图层级](../image/%E5%8E%9F%E7%94%9F%E5%B1%82%E7%BA%A7.png)
可以发现，ios的card2原生的View还在，但是安卓的第二个卡片View不见了。

这个现象是因为，就会发现 React Native Android UI 布局前，会对只有布局属性的 View[LAYOUT_ONLY_PROPS 源码](https://github.com/facebook/react-native/blob/c2c4b43dfe098342a6958a20f6a1d841f7526e48/ReactAndroid/src/main/java/com/facebook/react/uimanager/ViewProps.java#L185)进行过滤，这样可以减少 View 节点和嵌套，对碎片化的 Android 更加友好。

所以React组件映射到原生View时候，嵌套结构并不是一一对应的。那么在这个背景下如何进行优化呢？

##### 1. 减少嵌套层级数量（React.Fragment）
[Fragments](https://zh-hans.reactjs.org/docs/fragments.html)允许你将子列表分组，无需向DOM添加额外的节点
```javascript
class Table extends React.Component {
  render() {
    return (
      <table>
        <tr>
          <Columns />
        </tr>
      </table>
    );
  }
}

class Columns extends React.Component {
  render() {
    return (
      <React.Fragment>  // <-如果写成<View>那么会增加一级嵌套
        <td>Hello</td>
        <td>World</td>
      </React.Fragment>
    );
  }
}
```
最终输出的结果，会组合成一个table
```javascript
<table>
  <tr>
    <td>Hello</td>
    <td>World</td>
  </tr>
</table>
```
可见Fragments主要作用是减少嵌套的层级，可以使用在业务组件封装的场景。

##### 2. 减少GPU过度绘制
业务开发时候，如果不慎在多个层级都使用了相同的style，比如如下例子：
```javascript
render() {
  return (
    <View>
      <View style={{backgroundColor: 'white'}}>
        <View style={{backgroundColor: 'white'}}>
          <Text style={{backgroundColor: 'white'}}>Card1</Text>
        </View>
      </View>
      <View>
        <View>
          <Text>Card2</Text>
        </View>
      </View>
    </View>
  );
};
```
在渲染的时候，iOS与安卓会触发不同的GPU渲染机制。虽然背景色结果都是白色，但是GPU 的优化是不一样的。我们用 iOS 的 Color Blended Layers 和 Android 的[GPU过度绘制](https://developer.android.com/topic/performance/rendering/inspect-gpu-rendering#debug_overdraw)调试工具查看最后的渲染结果：
![过度绘制](../image/%E8%BF%87%E5%BA%A6%E7%BB%98%E5%88%B6.png)
对于 iOS 来说，出现红色区域，就说明出现了颜色混合：

Card1 的几个 View 都设置了非透明背景色，GPU 获取到顶层的颜色后，就不再计算下层的颜色了
Card2 的 Text View 背景色是透明的，所以 GPU 还要获取下一层的颜色进行混合

对于 Android 来说，GPU 会多此一举地渲染对用户不可见的像素。有一个颜色指示条：白 -> 蓝 -> 绿 -> 粉 -> 红，颜色越往后表示过度绘制越严重。

Card1 的几个 View 都设置了非透明背景色，红色表示起码发生了 4 次过度绘制
Card2 只有文字发生了过度绘制

由于过度绘制在iOS和安卓判定机制不一样，做视图优化时候，可以优先考虑优化安卓。可以从以下考虑优化方案：

**减少背景色的重复设置**：每个 View 都设置背景色的话，在 Android 上会造成非常严重的过度绘制；并且只有布局属性时，React Native 还会减少 Android 的布局嵌套
**避免设置半透明颜色**：半透明色区域 iOS Android 都会引起过度绘制
**避免设置圆角**：圆角部位 iOS Android 都会引起过度绘制
**避免设置阴影**：阴影区域 iOS Android 都会引起过度绘制
...

避免过度绘制的细节往往很多，建议长列表优化时候可以考虑。

#### 长列表性能优化


#### 预加载

预加载优化，主要分几个关键的阶段：

1. 终端页面
2. 引擎加载
3. JS包加载
4. JS执行
5. 渲染页面

### Bundle加载过程：
