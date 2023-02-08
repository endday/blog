---
title: 块格式化上下文（Block Formatting Context，BFC）和清除浮动的小结
url: https://www.yuque.com/endday/blog/xudf6f
---

<a name="a1bda266"></a>

## 块格式化上下文

- **根元素**或其它包含它的元素；
- **浮动** (元素的float不为none)；
- **绝对定位元素** (元素的position为absolute或fixed)；
- **行内块inline-blocks**(元素的 display: inline-block)；
- **表格单元格**(元素的display: table-cell，HTML表格单元格默认属性)；
- overflow的值不为visible的元素；
- **弹性盒 flex boxes** (元素的display: flex或inline-flex)；

<a name="86b954db"></a>

## 清除浮动

浮动的定义：使元素脱离文档流，按照指定方向发生移动，遇到父级边界或者相邻的浮动元素停了下来。

高度塌陷：浮动元素父元素高度自适应（父元素不写高度时，子元素写了浮动后，父元素会发生高度塌陷）

因为浮动会脱离文档流，使高度自适应的父元素高度塌陷，这样就会让布局错乱，所以在必要的时候需要清除浮动，让后面的元素能够正常布局，不会和浮动的元素相重叠。

一般会用到clear样式

```css
clear:left | right | both | none | inherit // 元素的某个方向上不能有浮动元素 
clear:both  // 在左右两侧均不允许浮动元素。
```

1. 添加空div

```css
{clear:both;height:0;overflow:hidden;}
```

2. 给浮动元素父级设置高度
3. 以浮制浮（父级同时浮动）

```javascript
嗯，这算是个基本没人用的蛋疼方法
```

4. 父级设置成inline-block

```css
{display: inline-block}
```

这就用到了bfc了

5. br 清浮动

```html
<div class="box">
    <div class="top"></div>
    <br clear="both" />
</div>
```

br 标签自带clear属性，将它设置成both其实和添加空div原理是一样的。

6. 父级添加overflow:hidden

```css
{overflow:hidden}
```

这也是用了bfc了

7. after伪类 清浮动（现在主流方法，推荐使用）

```css
.clear:after{content:'';display:block;clear:both;height:0;overflow:hidden;visibility:hidden;}
.clear{zoom:1;}
```
