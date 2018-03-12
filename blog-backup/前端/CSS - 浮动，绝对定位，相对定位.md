#1. 文档流
文档流是文档中的盒子模型按顺序排列形成的，是相对于盒子模型讲的。
文本流是由文字段落组成的。
**浮动、绝对定位、相对定位使得元素脱离文档流。**
#2. 浮动 float
##1） 特性
元素添加 float 属性浮动后，会跳出文档流，加入当前盒子模型的浮动流。
当浮动元素后面还有正常文档流中的元素时，会直接在浮动元素的下面布局（可以通过构建 BFC 防止覆盖）。但是文本流中的文字会围绕浮动元素布局，不会被覆盖（**文档流会被浮动元素覆盖，文本流则不会**）。
如果包含块所剩空间不足以容纳下一个浮动元素，这个浮动元素会自动另起一行。新行的浮动元素并不是靠左对齐，而是靠上对齐（靠近高度最小的浮动元素）。示例：
![这里写图片描述](http://img.blog.csdn.net/20180110151659134?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
##2） 值
float 属性的三个值：none（默认），left 左浮动，right 右浮动。
##3） 清除浮动 
###1. clear
clear 属性的值可以是 left，right，both，none，表示块级元素的哪些边取消可能存在的浮动。
`clear: left;` 清除左侧的浮动对象。
`clear: both;` 清除两侧的浮动对象。

浮动元素脱离文档流，导致只有浮动子元素的父元素塌陷。可以在浮动元素最后加一个 div，设定 `clear: both;`（这是无意义的标记，仅用于实现样式）防止塌陷，同时后面的元素不再受浮动影响。
###2. overflow
overflow 属性的值设为 hidden 或 auto 时，可以自动清理包含的所有浮动元素。
###3. :after 伪类和内容声明
声明为 block 块级元素独占一行，同时高度为0不占空间，visibility 为 hidden 用户不可见，同时清理所有浮动（不需要添加无意义的标记）。
```
.clear:after {
  content: ".";
  height: 0;
  visibility: hidden;
  display: block;
  clear: both;
}
```
##5） 示例
多个浮动元素依次排列：
![这里写图片描述](http://img.blog.csdn.net/20180109233826041?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
浮动元素覆盖正常文档流的元素：
![这里写图片描述](http://img.blog.csdn.net/20180109234014277?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
文本流不会被浮动定位的元素覆盖：
![这里写图片描述](http://img.blog.csdn.net/20180109234346474?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
清除浮动：
![这里写图片描述](http://img.blog.csdn.net/20180109235119232?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#3. 绝对定位
绝对定位后，**盒子元素会完全脱离文档流，文字元素也会脱离文本流**。
对于正常文档流中的元素而言，绝对定位的元素是不存在的。
绝对定位的元素的位置，是相对于距离它最近的已定位的祖先元素而言的。若没有已定位的祖先元素，则相对于初始包含块（画布或 HTML 元素）。可以在包含块的 top，left，bottom，right 四个方向移动。
z-index 可以设置 z 轴高度，数字越大则越靠上，不容易被覆盖。
#4. 相对定位
相对于元素最初的位置，做偏移。可以在 top，left，bottom，right 四个方向做偏移。
相对定位属于普通文档流。
使用相对定位时，不管元素是否移动，元素在文档流中仍然占据原先的空间。不影响其他元素的布局，但是可能会覆盖其他元素。
```
div {
  position: relative;
  left: 100px;
  top: 100px;
}
```