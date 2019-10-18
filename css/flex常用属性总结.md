### 结构
```
<div class="box">
    <div class="item">子元素</div>
    <div class="item">子元素</div>
    <div class="item">子元素</div>
    <div class="item">子元素</div>
    <div class="item">子元素</div>
</div>
```
### 基本构图

```
//最外层元素设置
.box{
    display:flex;
}
```
# 包裹对象所有的属性

### 最外层容器设置完成之后 会包含
# flex-direction //主轴方向 就是排序 row | row-reverse | column | column-reverse;
### flex-wrap  是否换行
# flex-flow flex-wrap 和 flex-direction 的简写 flex-flow: <flex-direction> <flex-wrap> ;
#### 水平对其方式 justify-content : flex-start/flex-end/center/space-between/space-around
#### 垂直对齐方式 align-items :flex-start | flex-end | center | baseline | stretch;
# align-content **多根轴线** 的对齐方式。如果项目只有一根轴线，该属性不起作用。

# 子对象所有的属性

* order 排序
* flew-grow 默认为0 有剩余空间不放大 ***1***
* flex-shrink 默认为 1 空间不足会缩小

ios 10 版本
flex 居中不起作用

Safari 10 and below uses min/max width/height declarations for actually rendering the size of flex items, but it ignores those values when calculating how many items should be on a single line of a multi-line flex container. Instead, it simply uses the item's flex-basis value, or its width if the flex basis is set to auto. see bug. Fixed in all versions > 10.
n Safari 10.1 and below, the height of (non flex) children are not recognized in percentages. However other browsers recognize and scale the children based on percentage heights. Fixed in all versions > 10.1 (See bug) The bug also appeared in Chrome but was fixed in Chrome 51




