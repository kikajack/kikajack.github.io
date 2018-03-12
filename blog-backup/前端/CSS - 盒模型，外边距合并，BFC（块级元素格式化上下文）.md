#一. 块级元素，行内元素，行内块级元素
块级元素：display:block，对于 div，p，h1 等标签，每个元素独占一行，并**尽可能的充满整个容器**。多个元素**从上至下依次排列**。
行内元素：display:inline，对于 span，a 等标签，多个元素可以在**一行内依次排列**。元素的宽度和高度无法设置。垂直方向的内外边距和边框无法调整。需要调整垂直尺寸时，可以用 `line-height` 行高。
行内块级元素：display:inline-block，对于 img，input，td 标签，和行内元素表现类似，但是可以设置宽度和高度。

可以通过 CSS 转换块级元素和行内元素：
转换为块级元素：`display:block;`
转换为行内元素：`display:inline;`
转换为行内块级元素：`display:inline-block;`
#二. 盒模型
[MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Introduction_to_the_CSS_box_model)
![这里写图片描述](http://img.blog.csdn.net/20180109114141727?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
CSS的盒模型中，从内到外依次是：content 内容区，padding 内边距，border 边框，margin 外边距。

width 和 height 指的是内容区的宽度和高度。内边距，边框和外边距的尺寸改变不会影响内容区的尺寸，但会影响盒子的总尺寸。
**外边距可以用负值。**
#三. 外边距合并（外边距塌陷）
永远不会合并的外边距：块级元素水平方向的外边距，行内元素、浮动定位和绝对定位之间的外边距。
[参考 MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing)
块的**顶部**外边距和**底部**外边距有时被组合(折叠)为单个外边距，其大小是最大的那个外边距，这种行为称为外边距塌陷。

三种基本情况：

- 相邻块级元素
相邻元素之间的外边距会塌陷（除非后者元素需要清除浮动）。这两个元素的距离，是两者的外边距的较大值。
```
<p style="margin-bottom: 30px;">XXX</p>
<p style="margin-top: 20px;">YYY</p>
```
- 父子块级元素
1. 块级父元素与其第一个子元素上外边距合并：如果块级父元素中，不存在上边框、上内边距、内联元素、块格式化上下文、 清除浮动 这五条（也可以说，当上边框宽度及上内边距距离为0时），此时这个块级父元素和其第一个子元素就会发生上外边距合并现象。
2. 块级父元素与其最后一个子元素下外边距合并：若块级父元素的 margin-bottom 与它的最后一个子元素的margin-bottom 之间没有父元素的 border、padding、inline content、height、min-height、 max-height 分隔时，就会发生下外边距合并现象。
3. 父子块级元素上外边距合并时，给子元素添加一个 `margin-top` 时，父子元素同时掉下来：
```
<style>
    .father {
        width: 200px;
        height: 200px;
        background: skyblue;
    }
    .son {
        width: 100px;
        height: 100px;
        background: red;
        margin-top: 50px;
    }
</style>
<body>
    <div class="father">
        <div class="son"></div>
    </div>
</body>
```
![这里写图片描述](http://img.blog.csdn.net/20180126104551556?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
- 空块元素
对于空的块级元素，其 border、padding、inline content、height、min-height 都不存在。此时它的上下边距中间将没有任何阻隔，上下外边距将会合并。下例中，中间 div 的上下外边距会合并。
```
<p style="margin-bottom: 0px;">这个段落的和下面段落的距离将为20px</p>
<div style="margin-top: 20px; margin-bottom: 20px;"></div>
<p style="margin-top: 0px;">这个段落的和上面段落的距离将为20px</p>
```
例外：

- BFC（块格式化上下文）与元素外边距合并时：

1. 当两个元素属于不同的 BFC 时，外边距不会合并。
2. 当块级元素和 BFC 相邻时，外边距不会合并。
3. 但在同一个BFC内，两个相邻元素的外边距仍会合并。
- 根元素（html ）的盒子的外边距永远不会坍塌。当你为 html 标签加上 margin 时，无论有无毗邻的外边距，这个 margin 都不会坍塌，是多少就多少。

#四. 块级元素格式化上下文
[BFC 渲染机制参考这里](https://funteas.com/topic/5a6a31643ed4587016d0f7a8)
##1. BFC 创建方式
BFC（Block Formatting Contexts），由以下之一创建：

- **根元素**或其它包含它的元素
- **浮动元素** (元素的 float 不是 none)
- **绝对定位元素** (元素的 position 为 absolute 或 fixed)
- **具有 overflow 且值不是 visible 的块元素**
- **display 值不为默认**
 - 内联块 (元素具有 display: inline-block)
 - 表格单元格 (元素具有 display: table-cell，HTML表格单元格默认属性)
 - 表格标题 (元素具有 display: table-caption, HTML表格标题默认属性)
 - display: flow-root
- column-span: all 应当总是会创建一个新的格式化上下文，即便具有 column-span: all 的元素并不被包裹在一个多列容器中。

一个块格式化上下文包括创建它的元素内部所有内容，除了被包含于创建新的块级格式化上下文的后代元素内的元素。

块格式化上下文对于定位 (参见 float) 与清除浮动 (参见 clear) 很重要。定位和清除浮动的样式规则只适用于处于同一块格式化上下文内的元素。浮动不会影响其它块格式化上下文中元素的布局，并且清除浮动只能清除同一块格式化上下文中在它前面的元素的浮动。
##2. BFC 特点
- 元素的定位方式有3种（普通流、绝对定位、浮动），BFC 属于普通流的一种。
- BFC 区域不会与浮动区域重叠（可以自动跟在浮动元素后面）。
- 当两个元素属于不同的 BFC 时，外边距不会合并。
- BFC 盒子内的元素与外部隔离，不会在布局上影响外部元素。
##3. BFC 用途
###1） 防止外边距合并
两个块级元素相邻并且在同一个块级格式化上下文时，垂直方向会发生外边距合并。产生新的 BFC 可以阻止外边距合并。
####1. 防止兄弟元素外边距合并
下例中，中间的 p 放入了 BFC 中，防止兄弟外边距合并：
```
  <p>P1</p>
  <div style="overflow: hidden;"><p>P2</p></div>
  <p>P3</p>
```
####2. 防止父子元素外边距合并
父元素不是 BFC 时，与第一个 p 元素发生外边距合并。
![父元素不是 BFC 时](http://img.blog.csdn.net/20180109172203613?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
父元素是 BFC 时，不发生外边距合并，第一个 p 元素的上外边距成为父元素的 content。
![父元素是 BFC 时](http://img.blog.csdn.net/20180109172216866?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
###2） 阻止元素被浮动元素覆盖
浮动元素与非浮动的块级元素相邻时，会发生重叠，块级元素位于浮动元素之下。
![这里写图片描述](http://img.blog.csdn.net/20180109175009309?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20180109175020378?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
###3） 包含浮动元素，防止高度塌陷
浮动的元素会脱离普通流，不仅会影响兄弟元素，也会影响父容器（高度塌陷）以及父容器兄弟元素。可以给父容器添加 overflow:hidden 属性，构成 BFC。
![这里写图片描述](http://img.blog.csdn.net/20180109174320442?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20180109174333607?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20180109174344036?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
###4） 清除浮动
通过设置 overflow:hidden 构建 BFC 可以清除浮动，同时对布局造成的影响较小。

其他清除浮动方法：
- 在父容器末尾添加空div，通过 clear:both 清除浮动。
- 为容器添加:after伪元素，并给伪元素设置overflow属性，实现清除浮动。
``` 
.clearfix{(*zoom: 1;} 
.clearfix:after{content:”;height:0;display:block; clear:both;/overflow:hidden; /*clear,overflow二选一*/}
```
