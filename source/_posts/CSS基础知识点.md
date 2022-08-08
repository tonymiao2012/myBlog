---
title: CSS基础知识点
date: 2021-03-09 13:18:26
categories:
tags:
---
#### CSS盒模型

1. 标准盒子模型：宽度=内容的宽度（content）+ border + padding + margin
低版本IE盒子模型：宽度=内容宽度（content+border+padding）+ margin

2. 用来控制元素的盒子模型的解析模式，默认为content-box
context-box：W3C的标准盒子模型，设置元素的 height/width 属性指的是content部分的高/宽
border-box：IE传统盒子模型。设置元素的height/width属性指的是border + padding + content部分的高/宽

#### CSS优先级算法和可继承性
1. 优先级（就近原则）：!important > [ id > class > tag ]
!important 比内联优先级高

2. CSS选择符：id选择器(#myid)、类选择器(.myclassname)、标签选择器(div, h1, p)、相邻选择器(h1 + p)、子选择器（ul > li）、后代选择器（li a）、通配符选择器（*）、属性选择器（a[rel="external"]）、伪类选择器（a:hover, li:nth-child）

可继承的属性：font-size, font-family, color

不可继承的样式：border, padding, margin, width, height

#### 数量优先级

1. 有三种不同的方法来选择要设置的样式。其优先级由高到低依次为：

    通过单独的 ID 或属性选择器进行引用
    通过类名在组合中进行引用
    通过元素标记进行引用
2. 一个规则的特性由基于上述编号列表中选择器类型所创建的三部分编号来评价。我们可以创建一个数字组合来计算优先级，这个数字组合以类似 [0, 0, 0] 的形式开始。当处理一个规则时，每个指向 ID 的选择器都会将第一个数字增加 1，所以数字组合会变为 [1, 0, 0]。

下面是个示例，这个规则的选择器中共有 3个 ID 选择器，所以数字组合变为 [3, 0, 0]：
``` css
#heading, #main, #menu, .text, .quote, .boxout, .news, .comments, p, blockquote, {
    font-family: Consolas;
    font-size: 16px;
}
```
接着，选择器中的类选择器的数量决定了数字组合中的第二位数字。在上例中，共有 5个类选择器，所以规则优先级现在是 [3, 5, 0].

最后，计算出所有指向元素标记的选择器数量，这个数字被放在数字组合的最后。在上例中有两部分（p 和 blockquote），所以最终的数字组合是 [3, 5, 2]，这就是与其他选择器比较优先级时所需要的全部。

在每种类型选择器的数量不超过 9个的情况下，可以将数字组合直接写为一个十进制的数字，比如上例中经过量化之后的优先级为 352。其他经过同样规则量化之后的 CSS 规则，若数值比这个数字小，优先级就比它低；数值比它大的，优先级就高于它。当两条规则拥有相同的数值时，采取”就近原则“，即最近被应用的优先。

### CSS3新增的伪类
* p:first-of-type 选择属于其父元素的首个元素  
* p:last-of-type 选择属于其父元素的最后元素  
* p:only-of-type 选择属于其父元素唯一的元素  
* p:only-child 选择属于其父元素的唯一子元素  
* p:nth-child(2) 选择属于其父元素的第二个子元素  
* :enabled :disabled 表单控件的禁用状态  
* :checked 单选框或复选框被选中  

### position跟display、overflow、float这些特性相互叠加后会怎么样？
display属性规定元素应该生成的框的类型；position属性规定元素的定位类型；float属性是一种布局方式，定义元素在哪个方向浮动。
类似于优先级机制：position：absolute/fixed优先级最高，有他们在时，float不起作用，display值需要调整。float 或者absolute定位的元素，只能是块元素或表格。

### display有哪些值
* inline（默认）--内联  
* none--隐藏  
* block--块显示  
* table--表格显示  
* list-item--项目列表  
* inline-block  

### position的值
static（默认）：按照正常文档流进行排列；  
relative（相对定位）：不脱离文档流，参考自身静态位置通过 top, bottom, left, right 定位；  
absolute(绝对定位)：参考距其最近一个不为static的父级元素通过top, bottom, left, right 定位；  
fixed(固定定位)：所固定的参照对像是可视窗口  

### FLEX布局？

### 常见的兼容性问题

1. 不同浏览器的标签默认的margin和padding不一样。
*{margin:0;padding:0;}

2. IE6双边距bug：块属性标签float后，又有横行的margin情况下，在IE6显示margin比设置的大。hack：display:inline;将其转化为行内属性。
3. 渐进识别的方式，从总体中逐渐排除局部。首先，巧妙的使用“9”这一标记，将IE浏览器从所有情况中分离出来。接着，再次使用“+”将IE8和IE7、IE6分离开来，这样IE8已经独立识别。
``` CSS
{
background-color:#f1ee18;/*所有识别*/
.background-color:#00deff\9; /*IE6、7、8识别*/
+background-color:#a200ff;/*IE6、7识别*/
_background-color:#1e0bd1;/*IE6识别*/
}
```
4. 设置较小高度标签（一般小于10px），在IE6，IE7中高度超出自己设置高度。hack：给超出高度的标签设置overflow:hidden;或者设置行高line-height 小于你设置的高度。
5. IE下，可以使用获取常规属性的方法来获取自定义属性,也可以使用getAttribute()获取自定义属性；Firefox下，只能使用getAttribute()获取自定义属性。解决方法:统一通过getAttribute()获取自定义属性。
6. Chrome 中文界面下默认会将小于 12px 的文本强制按照 12px 显示,可通过加入 CSS 属性 -webkit-text-size-adjust: none; 解决。
7. 超链接访问过后hover样式就不出现了，被点击访问过的超链接样式不再具有hover和active了。解决方法是改变CSS属性的排列顺序:L-V-H-A ( love hate ): a:link {} a:visited {} a:hover {} a:active {}

### 对BFC规范(块级格式化上下文：block formatting context)的理解？ 
BFC规定了内部的Block Box如何布局。
定位方案：

内部的Box会在垂直方向上一个接一个放置。
Box垂直方向的距离由margin决定，属于同一个BFC的两个相邻Box的margin会发生重叠。
每个元素的margin box 的左边，与包含块border box的左边相接触。
BFC的区域不会与float box重叠。
BFC是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。
计算BFC的高度时，浮动元素也会参与计算。
满足下列条件之一就可触发BFC

根元素，即html
float的值不为none（默认）
overflow的值不为visible（默认）
display的值为inline-block、table-cell、table-caption
position的值为absolute或fixed
### 为什么会出现浮动和什么时候需要清除浮动？清除浮动的方式？
浮动元素碰到包含它的边框或者浮动元素的边框停留。由于浮动元素不在文档流中，所以文档流的块框表现得就像浮动框不存在一样。浮动元素会漂浮在文档流的块框上。
浮动带来的问题：

父元素的高度无法被撑开，影响与父元素同级的元素
与浮动元素同级的非浮动元素（内联元素）会跟随其后
若非第一个元素浮动，则该元素之前的元素也需要浮动，否则会影响页面显示的结构。
清除浮动的方式：

父级div定义height
最后一个浮动元素后加空div标签 并添加样式clear:both。
包含浮动元素的父标签添加样式overflow为hidden或auto。
父级div定义zoom
