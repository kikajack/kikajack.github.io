  最近在做手机版网站，对比了jQuery Mobile，zepto，mui，最后选定mui。
  官方文档位置：http://dev.dcloud.net.cn/mui/ui/
注意事项：

##1. DOM结构
关于mui页面的dom，你需要知道如下规则。

###固定栏靠前

所谓的固定栏，也就是带有.mui-bar属性的节点，都是基于fixed定位的元素；常见组件包括：顶部导航栏（.mui-bar-nav）、底部工具条(.mui-bar-footer)、底部选项卡（.mui-bar-tab）;这些元素使用时需遵循一个规则：放在.mui-content元素之前，即使是底部工具条和底部选项卡，也要放在.mui-content之前，否则固定栏会遮住部分主内容；

###一切内容都要包裹在mui-content中

除了固定栏之外，其它内容都要包裹在.mui-content中，否则就有可能被固定栏遮罩，原因：固定栏基于Fixed定位，不受流式布局限制，普通内容依然会从top:0的位置开始布局，这样就会被固定栏遮罩，mui为了解决这个问题，定义了如下css代码：

    .mui-bar-nav ~ .mui-content {
        padding-top: 44px;
    }
    .mui-bar-footer ~ .mui-content {
        padding-bottom: 44px;
    }
    .mui-bar-tab ~ .mui-content {
        padding-bottom: 50px;
    }
你当然可以通过自定义CSS的方式实现如上类似效果，但为了使用简便，建议将除固定栏之外的所有内容，全部放在.mui-content中。

###始终为button按钮添加type属性

若button按钮没有type属性，浏览器默认按照type=submit逻辑处理，这样若将没有type的button放在form表单中，点击按钮就会执行form表单提交，页面就会刷新，用户体验极差。

###窗口管理
页面初始化：必须执行mui.init方法

mui在页面初始化时，初始化了很多参数配置，比如：按键监听、手势监听等，因此mui页面都必须调用一次mui.init()方法；

###页面跳转：抛弃href跳转

当浏览器加载一个新页面时，若页面DOM尚未渲染完毕，页面会先显示空白，然后等DOM渲染完毕后，再显示具体内容，这是WEB浏览器技术无法逾越的体验障碍；为解决这个问题，建议使用mui.openWindow方法打开一个新的webview，mui会自动监听新页面的loaded事件，若加载完毕，再自动显示新页面；

###页面关闭：勿重复监听backbutton

mui框架自动封装了页面关闭逻辑，若希望自定义返回逻辑（例如编辑页面的返回，需用户确认放弃草稿后再执行返回逻辑），则需要重写mui.back方法，切勿简单通过addEventListener添加backbutton监听，因为addEventListener只会增加新的执行程序，mui默认封装的监听执行逻辑依然会继续执行，因此若仅addEventListener添加用户确认框，则用户即使选择了取消，也会继续关闭窗口。

###手势操作
点击：忘记click

快速响应是mobile App实现的重中之重，研究表明，当延迟超过100毫秒，用户就能感受到界面的卡顿，然而手机浏览器的click点击存在300毫秒延迟（至于为何会延迟，及300毫秒的来龙去脉，请自行谷百），mui为了解决这个问题，封装了tap事件，因此在任何点击的时候，请忘记click及onclick操作，统统使用如下代码：

    element.addEventListener('tap',function(){
        //点击响应逻辑
    });
###常见错误
Uncaught ReferenceError: plus is not defined

在app开发中，若要使用HTML5+扩展api，必须等plusready事件发生后才能正常使用，否则可能会报“plus is not defined”的错误；
mui为简化开发，将plusReady事件封装成了mui.plusReady()方法，凡涉及到HTML5+的api，建议都写在mui.plusReady方法中；

##2.mui适用场景说明

为解决HTML5在低端Android机上的性能缺陷，mui引入了原生加速，其中最关键的就是webview控件，因此mui若要发挥其全部能力，需和5+ App配合适用，若脱离5+ App，mui功能会受限，主要涉及三个部分：

###webview窗口相关
涉及webview的，除了5+App，其它所有手机浏览器及PC浏览器均无法使用，涉及功能点包括：

- webview模式窗体动画

- 创建子窗口（除了为解决区域滚动的常见双webview场景，还涉及webview模式的选项卡等多webview场景）

- webview模式的侧滑菜单（也有div方式侧滑菜单）

- webview模式的tab选项卡（也有div方式选项卡）

- nativeUI，如原生的警告框、确认框、popover、actionsheet、toast。这些也有HTML5的实现。

- 预加载

- 自定义事件

###第三方扩展插件
涉及webview的，除了5+App，其它所有手机浏览器及PC浏览器均无法使用，目前主要包括：语音输入；

###Touch事件相关（注意pc浏览器没有touch事件）
Touch事件相关的，手机端浏览器均可使用、pc端chrome模拟手机浏览器也可以正常使用。
但普通PC端浏览器因为没有touch事件，可以显示控件但滑动操作功能会受限；涉及功能点包括：

- 手势事件

- mui封装的tap相关处理业务：折叠面板、二级列表、二级选项卡；

- mui封装的swipe、drag相关处理业务：图片轮播、可左右滑动的图文表格、可左右滑动的9宫格、滑动触发列表项菜单、可拖动式侧滑菜单、下拉刷新和上拉加载、可拖动式选项卡
【备注】：在PC端，大家将tap替换成click，将HTML5默认的Drag事件替换mui 的swipe和drag，就可以解决如上两个问题。

##3.mui框架如何实现页面间传值
在App开发中，页面间传值是很常见的开发需求，mui框架根据业务场景不同，提供了两种传值模式。
###1、页面初始化时，通过扩展参数传值；
mui在初始化页面时，提供了extras配置参数，通过该参数可以设置页面参数，从而实现页面间传值；
mui框架在如下几种场景下，会执行页面初始化操作：
- 通过mui.openWindow()打开新页面（若目标页面为已预加载成功的页面，则在openWindow方法中传递的extras参数无效）；
- 通过mui.init()方法创建子页面；
- 通过mui.init()方法预加载页面；
- 通过mui.preload()方法预加载页面

示例，假设我们有如下需求：
在首页中打开关于页面时，传递当前产品名称及版本号，然后在关于页面中读取这两个参数并显示出来；

首页实现代码：

  mui.openWindow({
      url:'info.html',
      id:'info.html',
      extras:{
          name:'mui',
          version:'0.5.8'
      }
  });
  关于页面实现代码：

  var self = plus.webview.currentWebview();
  var name = self.name;
  var version = self.version;
###2、页面已创建，通过自定义事件传值
参考mui官网中自定义事件的介绍

##4.mui如何增加自定义icon图标
mui框架遵循极简原则，在icon图标集上也是如此，mui仅集成了原生系统中最常用的图标；其次，mui中的图标并不是图片，而是字体，至于为什么使用字体图标而不是图片，相信web开发者多少都有所了解，简单列举几条：

- 多个图标字体合成一个字体文件，避免每张图片都需要联网请求；

- 字体可任意缩放，而图片放大会失真、缩小则浪费像素；

- 可通过css任意改变颜色、设置阴影及透明效果；

在实际项目中，开发者难免会需要自定义图标，此时该如何操作呢？本文以阿里巴巴矢量图标库为例（同样的网站有很多，比如icomoon，欢迎热心用户分享其它平台的使用方法），介绍一种用户自定义图标的方法，假设我们要做一个电商项目，需要补充男装、女装、购物车三个图标，如下为分步实现操作；

###登录

浏览器访问阿里巴巴矢量图标库官网，选择登录方式，可直接使用新浪微博账号登录；

###搜索图标

在右上角搜索“男装”，会列出当前网站上的所有男装图标，如下：
image
选择自己喜欢的图标，点击，会添加到右上角的购物车中，如下：
image
同样的方式分别搜索选择女装、购物车图标，结果如下：
image
之后点击“存储为项目”，输入项目名字，例如“mui-icon-custom”，点击“存储”按钮后，会跳转到项目管理页面，如下图所示：
image

###下载字体

点击“下载到本地”按钮，会将合并后的字体文件及自动生成的css全部下载，如下：
image

###修改css

默认的css代码如下：

  @font-face {font-family: "iconfont";
    src: url('iconfont.eot'); /* IE9*/
    src: url('iconfont.eot?#iefix') format('embedded-opentype'), /* IE6-IE8 */
    url('iconfont.woff') format('woff'), /* chrome、firefox */
    url('iconfont.ttf') format('truetype'), /* chrome、firefox、opera、Safari, Android, iOS 4.2+*/
    url('iconfont.svg#iconfont') format('svg'); /* iOS 4.1- */
  }
  
  .iconfont {
    font-family:"iconfont" !important;
    font-size:16px;
    font-style:normal;
    -webkit-font-smoothing: antialiased;
    -webkit-text-stroke-width: 0.2px;
    -moz-osx-font-smoothing: grayscale;
  }
  
  .icon-nanzhuang:before { content: "\e600"; }
  
  .icon-nvzhuang:before { content: "\e601"; }
  
  .icon-gouwuche:before { content: "\e602"; }
我们可稍作如下修改：

为保证和mui目录结构统一，建议将字体文件放在fonts目录下，这样我们需要修改@font-face下得url属性；

只兼容iOS和Android版本的话，我们仅需要ttf格式的字体即可，其它字体可以删除；同时，我们也仅需保留-webkit前缀语法，-moz前缀部分可以删除；

修改后的css代码如下：

    @font-face {font-family: "iconfont";
        src:url('../fonts/iconfont.ttf') format('truetype'); /* chrome、firefox、opera、Safari, Android, iOS 4.2+*/
    }

    .iconfont {
        font-family:"iconfont" !important;
        font-size:16px;
        font-style:normal;
        -webkit-font-smoothing: antialiased;
        -webkit-text-stroke-width: 0.2px;
    }

    .icon-nanzhuang:before { content: "\e600"; }

    .icon-nvzhuang:before { content: "\e601"; }

    .icon-gouwuche:before { content: "\e602"; }
###集成mui

将iconfont.css及iconfont.ttf两个文件分别拷贝到mui工程css及fonts目录下，然后即可在mui中引用刚生成的字体图标，我们以选项卡为例，代码如下：

    <nav class="mui-bar mui-bar-tab">
        <a class="mui-tab-item mui-active">
            <span class="mui-icon iconfont icon-nanzhuang"></span>
            <span class="mui-tab-label">男装</span>
        </a>
        <a class="mui-tab-item">
            <span class="mui-icon iconfont icon-nvzhuang"></span>
            <span class="mui-tab-label">女装</span>
        </a>
        <a class="mui-tab-item">
            <span class="mui-icon iconfont icon-gouwuche"></span>
            <span class="mui-tab-label">购物车</span>
        </a>
        <a class="mui-tab-item">
            <span class="mui-icon mui-icon-gear"></span>
            <span class="mui-tab-label">设置</span>
        </a>
    </nav>
主要代码：将mui默认的icon（如mui-icon-home）替换成iconfont icon-nanzhuang，修改后预览效果如下：

![](http://ask-pic.qiniudn.com/mui-icon-custom-6.png)