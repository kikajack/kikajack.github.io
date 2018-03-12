#一. SVN钩子是啥
钩子就是由版本库的**事件**（代码提交，属性修改等）**触发**的**程序**（Shell 脚本等）。
每一个 SVN 仓库下都有一个目录`hooks`，在这里面放所有的钩子程序。其中以`.tmpl`结尾的代表是模板，可以用来参考。
```
$ ls hooks/
post-commit.tmpl          pre-revprop-change.tmpl
post-revprop-change.tmpl start-commit.tmpl
pre-commit.tmpl          
```
#二. SVN钩子类型
###1. start-commit
在提交事务产生前运行，用来判定一个用户是否有权提交。版本库传给该程序两个参数：到版本库的路径，和要进行提交的用户名。程序返回一个非零值可以在事务产生前停止该提交操作。
###2. pre-commit
在事务完成提交之前运行。版本库传递两个参数到程序：版本库的路径和正在提交的事务名称。程序返回非零值，提交会失败，事务也会删除。
###3. post-commit
在事务完成后运行，创建一个新的修订版本。可以用这个钩子来同步测试环境的代码或者发送电子邮件。版本库传给程序两个参数：到版本库的路径和被创建的新的修订版本号。退出程序会被忽略。
###4. pre-revprop-change
在对版本库修改Subversion的版本属性时运行，版本库给钩子传递四个参数：到版本库的路径，要修改属性的修订版本，经过认证的用户名和属性自身的名字。
###5. post-revprop-change
与pre-revprop-change对应。只有存在pre-revprop-change时这个脚本才会执行。当这两个钩子都存在时，post-revprop-change在修订版本属性被改变之后运行，版本库传递四个参数给该钩子：到版本库的路径，属性存在的修订版本，经过校验的产生变化的用户名，和属性自身的名字。
#三. 使用SVN钩子
最常见的用法，就是本地修改的代码提交到 SVN 仓库后，测试环境可以同步更新。本文示例中的SVN仓库和测试环境在同一台服务器。
###1. 创建post-commit文件
```
#!/bin/bash
REPOS="$1" # 这是有SVN传过来的第一个参数，仓库位置
REV="$2" # 这是有SVN传过来的第二个参数，新的版本号
export LANG=zh_CN.UTF-8
echo "Code Deployed at `date "+%Y-%m-%d %H:%M"`" >> /opt/svn/repos/project1/hooks/deploy_log # 写日志是个好习惯
/usr/bin/svn update --username username --password password /home/porject1/trunk # 不用再一次次的手动执行这句命令了
```
###2. 改权限，建日志文件
```
chmod +x post-commit
touch deploy_log
```

这样，当本地代码改动后提交到SVN后，不再需要在测试环境的代码目录中执行`SVN update`操作。SVN可以自动化这一工作了。