# CSS

## float 

### float 参考链接
- [那些年我们一起清除过的浮动](http://www.iyunlu.com/view/css-xhtml/55.html)

## BFC

块级格式上下文。CSS 渲染的一部分，是块级盒子布局发生的区域。它会产生一个隔离的区域，不受外部干扰且不会干扰外界的区域。

创建 `BFC` 的条件：
- `html` 元素
- `float` 不为 `none`
- `position` 为 `absolute` 或 `fixed`
- `overflow` 不为 `visible`
- `display`:
    - `inline-block`
    - `flex`
    - `inline-flex`
    - `grid`
    - `inline-grid`
    - `table` 及其相关的一些类别
    - ...
- 其余一些存在兼容性的属性，更多可查阅 `MDN` ...

可解决的问题:
- 浮动导致的高度坍塌
- 外边距合并
- 自适应两栏布局

### BFC 参考链接

- [MDN](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)

## 层叠上下文

产生层叠上下文的几种方式:

- html 元素
- `inline-block` 元素
- `position` 不为 `relative` 或 `absolute` 且` z-index` 不为`auto`
- `position` 为 `fixed` 或 `sticky` 
- `flex`/`grid` 的子元素且 `z-index` 不为 `auto`
- `transform` 不为 `none`
- `opacity` 小于 `1`
- `filter` 不为 `none`
- 含有 `will-change` 属性
- ......

### 参考链接

- [深入理解CSS中的层叠上下文和层叠顺序](https://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/)
- [MDN 层叠上下文](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context)
- [深入理解 CSS 层叠上下文](https://blog.lbinin.com/frontEnd/CSS/css-stacking-context.html#层叠顺序)

## 盒模型

- 标准盒模型 `content-box`
- 怪异盒模型 `border-box`



## 替换元素/非替换元素

### 替换元素

替换元素会根据标签和属性来决定其具体显示内容。

- `img`
- `iframe`
- `video` 
- `embed`
- 有些元素会在特定情况下可视作为替换元素, 详见 MDN  

### 非替换元素

除上述之外就属于非替换元素，其内容直接表现给浏览器。

### 参考链接

- [MDN 可替换元素](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Replaced_element)


## Flex

块级元素: `display: flex`

行内元素: `display: inline-flex`


当元素设置为 `flex` 布局时，子元素的 `float`、`clear`、`vertical-align` 属性会失效


### 容器可选属性

#### `flex-direction` 

设置主轴方向。

可选值:
- `row` 默认值
- `row-reverse`
- `column`
- `column-reverse`

#### `flex-wrap` 

控制是否换行。

可选值:
- `no-wrap` 默认
- `wrap` 换行
- `wrap-reverse` 换行。第一行在下方

#### `flex-flow` 

该属性为 `flex-direction` & `flex-wrap` 的简写： `flex-flow: <flex-direction> || <flex-wrap>;`。

默认值: `flex-flow: row nowrap`

#### `justify-content`

主轴对齐方式。

可选值:
- `flex-start`
- `flex-end`
- `center`
- `space-around`
- `space-between`

#### `align-items`

交叉轴对齐方式

可选值:
- `flex-start`
- `flex-end`
- `center`
- `baseline` 子项的第一行文字基线对齐
- `stretch` 如果子项未设置高度或设置为 `auto`, 将占满整个高度

#### `align-content`

交叉轴定义了多根轴线（允许换行的情况）的对齐方式

可选值:
- `flex-start`
- `flex-end`
- `center`
- `space-around`
- `space-between`
- `stretch`

### 子元素可选属性

#### order

子元素在主轴的排序。可选值为数字，数字越大越靠前，默认为 `0`

#### `flex-grow`

子元素自身放大的比例。可选值为数字，默认为 `0`, 即有剩余空间也不放大。

如果大于 `0`, 会根据子元素的 `flex-grow` 值比例划分剩余空间。

#### `flex-shrink`

子元素自身缩小的比例。可选值为 `0` ~ `1`。默认值为 `1`，代表如果可用空间不足，将会缩小自身。

如果所有子元素都设置为 `1`, 则空间不足时，所有子元素会等比缩小。

如果一个子元素为 `0`, 其余为 `1` 时，则前者不缩小，后者等比缩小。

#### `flex-basic`

分配多余空间之前，子元素占据的主轴空间。浏览器会根据这个值来计算剩余空间。

可选值同 `width`。默认值为 `auto`，即项目本身大小。

#### `flex`

`flex-grow` & `flex-shrink` & `flex-basic` 的缩写。

可选值: `flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]`

默认值为 `0 1 auto`

flex 属性可以指定1个，2个或3个值。

##### 单值语法

值必须为以下其中之一:

- 一个无单位数(`number`): 它会被当作`flex-grow`的值。等价于 `flex: <flex-grow> 1 0%`
- 一个有效的宽度(`width`)值: 它会被当作 `flex-basis` 的值。等价于 `flex: 1 1 <flex-basis>`
- 关键字 
    - `none` 等价于 `0 0 auto`
    - `auto` 等价于 `1 1 auto`
    - `initial` 等价于 `0 1 auto`

##### 双值语法

- 第一个值必须为一个无单位数，并且它会被当作 `flex-grow` 的值。

- 第二个值必须为以下之一：
    - 一个无单位数：它会被当作 `flex-shrink` 的值。
    - 一个有效的宽度值: 它会被当作 `flex-basis` 的值。

##### 三值语法:

第一个值必须为一个无单位数，并且它会被当作 `flex-grow` 的值。

第二个值必须为一个无单位数，并且它会被当作  `flex-shrink` 的值。

第三个值必须为一个有效的宽度值， 并且它会被当作 `flex-basis` 的值。


#### `align-self`

子元素自身在交叉轴的对齐方式

可选值:
- `flex-start`
- `flex-end`
- `center`
- `baseline` 子项的第一行文字基线对齐
- `stretch` 如果子项未设置高度或设置为 `auto`, 将占满整个高度

#### Flex 参考链接

- [MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex)
- [阮一峰 Flex](https://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)
- [flex设置成1和auto有什么区别](https://segmentfault.com/q/1010000004080910)