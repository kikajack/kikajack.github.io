Git 资料参考：
http://www.runoob.com/git/git-tutorial.html
GitHub 资料参考：
http://www.ruanyifeng.com/blog/2014/06/git_remote.html

所有操作都是基于 CentOS7，其他系统仅供参考。
#1. 简介
Git 和 SVN 不一样，Git 是一个开源的、分布式的版本控制系统，没有服务器端。之所以要在服务器上安装 Git，是为了建立一个基准，并通过公网环境保证 24 小时随时可以同步。
#2. 安装配置
下载地址：
https://git-scm.com/downloads
https://github.com/git/git/releases
https://www.kernel.org/pub/software/scm/git/
##一 安装
###1） Linux下
- CentOS：
CentOS 自带Git，我的 7.x 版本自带的 Git 版本为 1.8.3.1。
卸载：`yum remove git`
安装：`yum install git`
查看版本：`git --version`
- Ubuntu（没试过）：
`$ apt-get install libcurl4-gnutls-dev libexpat1-dev gettext libz-dev libssl-dev`
`$ apt-get install git-core`
或者直接：`$ apt-get install git`
###2） Windows下
下载对应的 Windows 版本软件，安装即可。
##二 配置
###1）Git 环境变量
Git 自带的 config 工具用来配置或读取相应的环境变量。
Git 的环境变量分为三种：

1. /etc/gitconfig 文件：系统中对所有用户都普遍适用的配置。若使用 git config 时用 --system 选项，读写的就是这个文件。
2. ~/.gitconfig 文件：用户目录下的配置文件只适用于该用户。若使用 git config 时用 --global 选项，读写的就是这个文件。
3. 当前项目的 Git 目录中的配置文件（也就是工作目录中的 .git/config 文件）：这里的配置仅仅针对当前项目有效。每一个级别的配置都会覆盖上层的相同配置，所以 .git/config 里的配置会覆盖 /etc/gitconfig 中的同名变量。
###2）用户信息
配置用户名和电子邮件地址：
```
$ git config --global user.name "runoob"
$ git config --global user.email test@runoob.com
```
如果用了 --global 选项，那么更改的配置文件就是位于你用户主目录下的那个，以后你所有的项目都会默认使用这里配置的用户信息。
如果要在某个特定的项目中使用其他名字或者电邮，只要去掉 --global 选项重新配置即可，新的设定保存在当前项目的 .git/config 文件里。
###3）文本编辑器
设置Git默认使用的文本编辑器, 一般可能会是 Vi 或者 Vim。如果你有其他偏好，比如 Emacs 的话，可以重新设置：:
`$ git config --global core.editor emacs`
###4）差异分析工具
还有一个比较常用的是，在解决合并冲突时使用哪种差异分析工具。比如要改用 vimdiff 的话：
$ git config --global merge.tool vimdiff
Git 可以理解 kdiff3，tkdiff，meld，xxdiff，emerge，vimdiff，gvimdiff，ecmerge，和 opendiff 等合并工具的输出信息。
也可以指定使用自己开发的工具。
###5）查看配置信息
要检查已有的配置信息，可以使用 git config --list 命令：
```
$ git config --list
```
有时候会看到重复的变量名，那就说明它们来自不同的配置文件（比如 /etc/gitconfig 和 ~/.gitconfig），不过最终 Git 实际采用的是最后一个。
这些配置我们也可以在 `~/.gitconfig` 或 `/etc/gitconfig` 看到，如下所示：
```
vim ~/.gitconfig 
```
也可以直接查阅某个环境变量的设定，只要把特定的名字跟在后面即可，像这样：
```
$ git config user.name
```
#3. Git 基本使用
如果只是自己使用，可以用 root 用户访问 Git 仓库。但多人使用时，为了保证最小权限，需要建立 Git 账户，把每个用户的 SSH 公钥放入 Git 账户的 `~/.ssh/authorized_keys` 文件，从而通过 Git 账户访问仓库。
##一 基本使用
###1）指定 Git 仓库目录
`git init` 用当前目录作为Git仓库。
`git init /new/repo`  指定目录作为Git仓库。
###2）添加新文件
`git add file` 
###3）删除文件
`git rm file`
###3）提交文件
`git commit -m "my files"`
提交之后，文件才真正的进入 Git 仓库。
`-m` 参数后加注释字符串，也可以不加 `-m` 参数，会通过编辑器来写注释。
`-a` 参数可将所有被修改或已删除的且已经被git管理的文档提交到仓库中。
###4）从远端服务器获取库
`git clone ssh://example.com/~/www/project.git`
###6）获取远端服务器的更新
`git pull`
###5）推送到服务器。
`git push ssh://example.com/~/www/project.git`
#4 GitHub
##一 简介
GitHub 是一个基于 Git 的代码托管平台，付费用户可以建私人仓库，免费用户只能使用公共仓库（任何人都可以访问）。
你只要在 GitHub 注册了账号，就可以登录进去创建仓库，不再需要在自己的服务器上安装 Git 了。
##二 使用
###1）注册
 github官网地址：https://github.com/
###2）安装客户端
下载地址：
https://git-scm.com/downloads
http://gitforwindows.org/
###3）配置
首先需要创建 SSH，可以参考[这里](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)。

1. 打开 Git Bash，进入命令行界面。
2. 执行命令 `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`，邮箱替换为注册 GitHub 用的邮箱。这个命令最终会产生一个新的 SSH Key。命令执行后，会进入一个交互式界面。
3. 输入 Key 的位置。直接回车用默认位置即可。
`Enter a file in which to save the key (/c/Users/you/.ssh/id_rsa):` 
4. 输入密码
`Enter passphrase (empty for no passphrase): [输入密码，可以为空]`
`Enter same passphrase again: [再次输入密码]`
这里就创建完成了，可以通过命令 `cat id_rsa.pub` 查看内容。
![命令细节](http://img.blog.csdn.net/20171219173350242?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
5. 在 GitHub 的个人中心，把刚创建的 Key 复制进来，注意保持格式。
![GitHub 界面](http://img.blog.csdn.net/20171219163358543?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
6. 验证，在 Git Bash 命令行界面中执行以下命令。如果没设置密码，则直接连接上，有设置密码在需要输入密码（提示是否 continue 的话，直接输入yes）。
`ssh -T git@github.com`
7. 设置用户名和邮箱地址
```
$ git config --global user.name "your name"
$ git config --global user.email "your_email@youremail.com"
```
###4）操作仓库
####1. 从 GitHub 仓库检出：
`git clone username@host:/path/to/repository`
注意这里使用的路径，是 SSH 的，可以从网站上复制下来。
![从 GitHub 网站复制仓库链接](http://img.blog.csdn.net/20171219173139096?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
这里我的完整命令是： `git clone git@github.com:kikajack/kikajack.github.io.git`
####2. 创建远程仓库：
`git remote add origin git@github.com:yourName/yourRepo.git`
加完之后进入.git，打开config，这里会多出一个remote "origin"内容，这就是刚才添加的远程地址，也可以直接修改config来配置远程地址。