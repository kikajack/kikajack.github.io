后面的所有例子，都分为 3 层嵌套：父元素，子元素，内容。其中父子元素都是块级元素 div。
#1. flex 布局加属性 justify-content，align-items
父元素设置属性 `display: flex; justify-content: center;  align-items: center;`。
![这里写图片描述](http://img.blog.csdn.net/20180126120308188?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#2. flex 布局加子元素设置属性 margin
父元素设置属性 `display: flex;`，子元素设置属性 `margin: auto;`。
![这里写图片描述](http://img.blog.csdn.net/20180126120131650?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#3. 父元素 display: table，子元素 display: table-cell
父元素设置显示样式为 `display: table`，子元素设置显示样式为 `display: table-cell`。由于表格中只有一个单元格，**这个单元格会自动撑开占满整个表格，且宽高就是表格的宽高**。单元格中的内容就可以通过设置 `vertical-align: middle; text-align: center;` 自动水平垂直居中了。
优点：内容不需提前定义宽高，可以自动适应，自动居中。当内容太多时会自动把表格撑大，内容不会截断。
![这里写图片描述](http://img.blog.csdn.net/20180126111121234?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20180126112142214?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#4. 父元素 display: table-cell，子元素 display: inline-block
父元素设置显示样式为 `display:table-cell`，然后利用 vertical 和 text-align 让所有的行内块级元素水平垂直居中。子元素设置显示样式为 `display: inline-block`。
![这里写图片描述](http://img.blog.csdn.net/20180126125113219?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#5. 绝对定位加 margin
**绝对定位是相对于最近的非 static 的祖先元素进行定位的，所以我们要在其相对定位的元素上定义 display 属性。**
父元素设置 position 为 absolute。
子元素设置 top 为 50%， left 为 50%，margin-top 和 margin-left 为宽高一半的负值。即先把子元素左上角放在父元素中间，然后再向左上方移动子元素宽高的一半，使父子元素中心重合。
缺点：子元素的宽度和高度需要提前限定好。没有足够空间时，子元素的内容会溢出（可以设置 `overflow:auto;` 变成滚动条。）。
![这里写图片描述](http://img.blog.csdn.net/20180126113858795?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#6. 绝对定位加 transform，translate
**绝对定位是相对于最近的非 static 的祖先元素进行定位的，所以我们要在其相对定位的元素上定义 display 属性。**
不知道子元素的宽高，或者希望子元素可以随着内容多少而自动调整尺寸时，用此方法。
![这里写图片描述](http://img.blog.csdn.net/20180126114256574?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#7. 绝对定位加 top: 0; bottom: 0;
子元素设置属性 `top:0; bottom:0;`。如果子元素没有 height 属性时，默认高度占满父元素。但设置了 height 后，自动垂直居中。`margin:auto;` 自动实现水平居中。**兼容性有问题，别用。**
![这里写图片描述](http://img.blog.csdn.net/20180126115354564?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#8. 单行文本水平垂直居中
把 line-height 设置为那个对象的 height 值使文本垂直居中，把 text-align 设置为 center，使文本水平居中。
一般用于使按钮文本或单行文本等小元素。
![这里写图片描述](http://img.blog.csdn.net/20180126115730121?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#9. grid 布局加属性 justify-content: center 和 align-items:center
**grid 布局较新，兼容性有问题。**
![这里写图片描述](http://img.blog.csdn.net/20180126125255276?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)