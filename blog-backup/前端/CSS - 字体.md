[CSS 字体参考资料](http://www.w3school.com.cn/css/css_font.asp)
#1. 字体相关属性
|属性|意思|
|:-|-|
|font-size|字体大小，单位有 px 像素，em 相对父元素，rem 相对于根 HTML 元素。PC 端浏览器默认是 16px。|
|font-family|字体系列，常用的有 'PingFang SC'，'Microsoft Yahei'，simsun|
|font-weight|字体粗细，9 级加粗度（100 ~ 900）。400 等价于 normal，700 等价于 bold。|
|font-style|字体的风格，默认值 normal 标准字体样式，可以使用：italic 斜体，oblique 倾斜，inherit 从父元素继承字体样式。|
#2. font-family
##1. 字体分类
CSS 的字体系列，按照是否通用可以分为两类：

- 通用字体系列 - 拥有相似外观的字体系统组合，CSS 中有 5 种通用字体：Serif 衬线字体，Sans-serif 非衬线字体，Monospace 等宽字体（易于对齐，IDE中常见），Cursive 字体，Fantasy 字体。
- 特定字体系列 - 具体的字体系列（比如宋体，微软雅黑，苹方字体）

按照有无衬线可以分为两类：

- Serif 衬线字体，在字的笔画开始、结束的地方有额外的装饰，而且笔画的粗细会有所不同。易读性比较高，适合**用于正文**。包括宋体（微软自带中易宋体 SimSun，由于含有过细笔画的文字和笔画粗细差异较大的字体，适合印刷而非屏幕显示），Georgia，Times New Roman。
- Sans-serif 非衬线字体，没有这些额外的装饰，而且笔画的粗细差不多。醒目，适合**做标题**。包括黑体（SimHei），微软雅黑，Arial，Tahoma，Verdana。

##2. web 开发的注意事项
####1. 在所有 font-family 规则中最后提供一个通用字体系列
这样就提供了一条后路，在用户代理无法提供与规则匹配的特定字体时，就可以选择一个候选字体。比如：`font-family: "Microsoft YaHei",tahoma,arial,"Hiragino Sans GB","\\5b8b\4f53",sans-serif` ，
知乎的标题和正文统一用非衬线字体：`font-family: -apple-system,BlinkMacSystemFont,"Helvetica Neue",Helvetica,Arial,"PingFang SC","Hiragino Sans GB","WenQuanYi Micro Hei","Microsoft Yahei",Lato,proxima-nova,sans-serif`。
####2. 如果字体名中有一个或多个空格（比如 New York），或者如果字体名包括 `#` 或 `$` 之类的符号，需要在 font-family 声明中加单引号或双引号。
####3. 字体名称中不要出现中文
为了提高兼容性，保证在不支持中文的平台上也可以正常使用，可以将中文名称的字体转换为Unicode编码。
|中文|英文|Unicode|
|-|-|
|黑体|SimHei|\9ED1\4F53|
|宋体|SimSun|\5B8B\4F53|
|微软雅黑|Microsoft YaHei|\5FAE\8F6F\96C5\9ED1|
####4. 在 html 中增加 lang 标签，显式声明网页所用语言
`<html lang="zh-CN">`
网页所用语言明确，有利于加载对应的字体等。
[网页头部的声明应该是用 lang="zh" 还是 lang="zh-cn" 参考这里](https://www.zhihu.com/question/20797118)
##3. 字体和操作系统的相关知识
###1. Windows
1. Windows 下，font-family 默认就是 中易宋体 SimSun，可以采用 微软雅黑（从 Vista 开始）、Tahoma（XP 和 Vista）或 Arial（早期 Windows）。Tahoma 是英文 Windows 操作系统的默认字体，这个字体比较均衡，显示中英文混排很不错，是经久耐看的一款字体。
2. 关于 微软雅黑 和 宋体，低分辨率的显示器或小号字体（14px 及以下）建议用宋体，高分辨率显示器或中大号字体可以用微软雅黑（矢量字体）。[参考这里](https://www.zhihu.com/question/19685531)。大面积的文字，用微软雅黑会更柔和一些。比如凤凰网的文章，正文用 16px + 微软雅黑，评论用 14px + 宋体。
###2. Mac
1. Mac OS X 系统的默认字体：Helvetica，可以指定为苹方字体。
- `font-family: -apple-system, "PingFang SC";` ：调用苹方字体。
- `font-family: sans-serif;` 配合 lang 属性：调用到各个地区的苹方字体（`<html lang="zh-CN">` 调简体中文的平方字体）。
- `font-family: Helvetica;` 或者 `sans-serif` 且不加 lang 属性，可以调用苹方的 Light、Regular 和 Bold 字重。
- `font-family: "Helvetica Neue";` 可以调用苹方的所有字重。

###3. 安卓
默认字体 Roboto，中文字体有 Segoe UI，思源黑体 (Noto Sans CJK)

知乎在移动端的字体是：`font-family: "Helvetica Neue",Helvetica,Roboto,"Segoe UI",Arial,sans-serif;` 前两个配合苹果，中间两个配合安卓，最后一个确保万无一失。