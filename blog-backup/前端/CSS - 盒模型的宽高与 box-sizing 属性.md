#1. 盒模型的宽高
##1.1 标准盒模型
标准盒模型中，设置盒子的 width 和 height 属性时，实际上只是设置了盒子中 content 内容区的 width 和 height ，盒子实际宽高还要加上 border 和 padding。
标准盒模型的盒子的宽度 = `border-left` + `padding-left` + `width` + `padding-right` + `border-right`，下图例子中，盒子宽度为 50 + 50 + 100 + 50 + 50 = 300px。
![这里写图片描述](http://img.blog.csdn.net/20180128184557764?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
##1.2 IE盒模型
IE盒模型中，设置盒子的 width 和 height 属性时，就是设置盒子的真实 width 和 height 。不用再进行麻烦的计算，也不用担心元素互相影响。
##1.3 比较
下图中，上下两个盒子的父元素 content 宽度都是 200 - 40 = 160px，两个子元素的宽度都是 100%。标准盒模型（content-box）中，设置 width 属性是针对 content 生效，实际盒子的宽度是 content 宽度加上左右边框加上左右内边距，即 160 + 40 + 20 = 220px。IE盒模型（border-box）中，设置的 width 属性就是盒子的总的 width，实际盒子宽度就是 160px。
![这里写图片描述](http://img.blog.csdn.net/20180128230533452?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#2. box-sizing 属性
盒模型可以通过 box-sizing 来设置：
语法：`box-sizing:  content-box | border-box | inherit;`

- content-box：标准盒模型，CSS 定义的宽高只是 content 内容区的宽高。盒子实际宽高是内容区、内边距与边框的尺寸之和。内边距 padding 和边框 border 的尺寸改变**不会影响内容区的宽高**，但会影响盒子的总尺寸。 
- border-box：IE盒模型，CSS 定义的宽高包括了 content，padding 和 border。内边距 padding 和边框 border 的尺寸改变**会影响内容区的宽高**，但不会影响盒子的总尺寸。 