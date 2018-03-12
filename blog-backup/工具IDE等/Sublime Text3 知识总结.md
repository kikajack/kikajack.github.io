#一. 安装插件
[常用插件参考](http://www.cnblogs.com/hykun/p/sublimeText3.html)
##1. Package Control 插件总管
下面的 2 种安装方式均可以。
###1) 通过命令行安装
Package Control 插件是 Sublime Text 插件家族的大管家。通过 `ctrl+\`（Esc 下面那个键）` 快捷键或者 View > Show Console 菜单打开控制台，复制粘贴回车如下代码即可。
```
import urllib.request,os,hashlib; h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```
###2） 通过下载插件放到对应文件夹内来安装
1. 通过菜单 `Preferences > Browse Packages` 进入文件系统。
2. 去上级目录，找到 `Installed Packages/` 目录。
3. [下载插件](https://sublime.wbond.net/Package%20Control.sublime-package)，放到 `Installed Packages/` 目录中，重启生效。
###3）使用镜像
因为 Package Control 的官方镜像 packagecontrol.io 太不好用，如果打不开或太慢的话，可以使用镜像。在菜单栏通过 `preferencts->settings` 打开设置，编辑右侧的 `User` 配置信息，添加 channels 属性，数组中首先写镜像的地址即可。
```
"channels": 
[
"https://web.archive.org/web/20150905194312/https://packagecontrol.io/channel_v3.json",
"https://packagecontrol.io/channel_v3.json"
],
```
##2. Emmet 插件，HTML 高效工具
###1） 安装
1. 通过快捷键 `shift + ctrl + p` 打开命令面板。
2. 输入 ip 然后选择 Install Package，回车，进入 Package Control 插件的面板。
3. 输入 emmet 找到 Emmet For Sublime Text，点击就可以自动完成安装。
###2） 简介
快捷键可以看官网或 [这里](https://www.w3cplus.com/tools/emmet-cheat-sheet.html)
文件保存后，emmet 插件才生效。
输入文本后，tab 键唤起 emmet 插件。

- 输入 html 后，tab 键可以自动初始化html文件。
- 后代：>
缩写：`nav>ul>li`
- 兄弟：+
缩写：`div+p+bq`
- 上级：^
缩写：`div+div>p>span+em^bq`
- 分组：()
缩写：`div>(header>ul>li*2>a)+footer>p`
- 乘法：*
缩写：`ul>li*5`
- 自增符号：`$`
缩写：`ul>li.item$*5`
缩写：`h$[title=item$]{Header $}*3`
缩写：`ul>li.item$$$*5`

#二. 编辑用户设置
编辑 Sublime 用户设置的方法：在菜单栏通过 `preferencts->settings` 打开设置，编辑右侧的 `User` 配置信息。默认的设置如下：
```
{
  "font_size": 8,
  "ignored_packages":
  [
    "Vintage"
  ]
}
```
###1. 缩进
[参考这里](https://feliving.github.io/Sublime-Text-3-Documentation/indentation.html)

缩进设置除了自动检测之外，可以定义为全局，仅作用于指定文件类型，或者仅作用于指定文件。
可以设置 Tab 和空格自动转换。
```
{
  "font_size": 8,
  "ignored_packages":
  [
    "Vintage"
  ],
  "tab_size": 2, //每次按 tab 键的空白符长度
  "translate_tabs_to_spaces": true // 是否将 tab 键换为空格，如果为true，按 tab 键将会输入空格
}
```