##1. 改动还没有提交（commit）
`svn revert`
Windows 下，右键点击要还原的目录或文件->TortoiseSVN->Revert，在弹窗中选择需要还原的文件，确定即可。

命令行下，通过命令 `svn revert` 恢复未提交的改动：
```
svn revert 单个文件 #恢复单个文件
svn revert -R 目录 #恢复目录下所有文件
```
##2. 改动已经提交到 SVN 服务器
基本流程：svn update 更新到最新版本->svn log 查看提交日志，找要回滚的版本号->还原到指定版本->commit 提交。
###1）命令行
用 svn merge 命令进行回滚。 
####1.获取最新代码
`svn update`
####2. 查看日志
`svn log [PATH]`
查看指定文件或指定目录的 SVN 提交日志，默认查看整个项目。
####3. 比较不同版本的差异
`diff [-c M | -r N[:M]] [TARGET[@REV]...]`
`diff [-r N[:M]] --old=OLD-TGT[@OLDREV] [--new=NEW-TGT[@NEWREV]] [PATH...]`

-r 参数表示 reversion
-c 参数表示 changed 
N 表示新版本号
M 表示旧版本号

[完整文档参考这里](http://svnbook.red-bean.com/en/1.7/svn.ref.svn.c.diff.html)

`svn diff -r 28:25 [something]` ：比较第 28 和 25 版的区别。

####4. 版本变更
`svn merge sourceURL1[@N]  sourceURL2[@M]  [WCPATH]`
`svn merge sourceWCPATH1@N  sourceWCPATH2@M  [WCPATH]`
`svn merge [-c M[,N...] | -r N:M ...] SOURCE[@REV] [TARGET_WCPATH]`

[merge 参考](http://www.voidcn.com/article/p-yyfxxveg-bcu.html)
`svn merge -r 28:25 something`：将第 28 版合并到第 25 版。
####5. 提交
`svn commit -m "message here" `

###2）Windows 下通过 GUI 操作
####1. 获取最新代码
右键->SVN Update。
####2. 查看日志
右键->TortoiseSVN->Show log。
####3. 版本变更
- 恢复到指定版本
右键点击要恢复到的版本->Revert to this reversion。
- 删除提交记录至指定版本
右键点击要删除的版本->Revert chages from these revisions。
####4. 提交
右键->SVN Commit。