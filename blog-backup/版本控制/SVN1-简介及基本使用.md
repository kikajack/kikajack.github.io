##1.subversion（简称SVN）
是一个集中式版本控制系统，用于团队协作。SVN的仓库在服务器上，需要在联网条件下才能提交和更新。
##2.服务器和数据存储方式
###1.服务器
SVN的运行方式有两种：独立服务器和借助Apache。SVN可以安装在Linux或Windows系统中，安装完成后自带独立的SVN服务器，可以立刻使用。
###2.数据存储方式
SVN的数据存储方式有两种：Berkeley DB（数据库）和FSFS（文件系统）。Berkeley DB方式在服务器中断时可能锁住数据。FSFS方式更常用。
##3.客户端
常用的客户端是TortoiseSVN（乌龟）。也可以用WEB方式或命令行方式访问。
##4.组件
ubversion由几个组件组成：
SVN：命令行客户端程序（常用）。
SVNversion：此工具用来显示工作拷贝的状态（用术语来说，就是当前项目的修订版本）。
SVNlook：直接查看Subversion版本库的工具。
SVNadmin：建立、调整和修复Subversion版本库的工具（重要）。
SVNdumpfilter：过滤Subversion版本库转储数据流的工具。
mod_dav_SVN：ApacheHTTP服务器的一个插件，使版本库可以通过网络访问。
SVNserve：一个单独运行的服务器程序，可以作为守护进程或由SSH调用，这是另一种使版本库，可以通过网络访问的方式（重要）。
SVNsync：一个通过网络增加镜像版本库的程序。
##5.版本控制策略
SVN：拷贝-修改-合并
多个用户可以同时阅读编辑同一个文件（每个用户在本地都有一份拷贝），A先提交编辑后的文件后，B如果提交，SVN会提示文件已过期，需要先更新为最新版本。如果B与A修改的文件位置不重叠，B可以顺利更新并提交。如果重叠，B更新时会报错‘冲突’，此时需要人工介入处理。
##6.工作流程
![这里写图片描述](http://img.blog.csdn.net/20170506221633308?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
团队协作时，为了确保版本一致，必须保证所有人用的数据都是从SVN服务器checkout到本地的，且受SVN版本控制。
基本流程是：建仓库--关联项目（原始项目文件可以删除）--各团队成员checkout到本地--正常使用（增加add，更新update，提交commit）
##7.基本使用
###1.安装SVN
不同的Linux版本，安装方式不同。以服务器常用的centOS为例，用`yum install`安装（源码安装容易导致版本依赖问题）。
####1.如果已经安装SVN，先卸载
检查是否已安装`rpm -qa subversion`
卸载旧版本：`yum remove subversion`
####2.安装
单独安装SVN：`yum install subversion`
如果需要配合Apache：`yum install httpd httpd-devel subversion mod_dav_svn mod_auth_mysql`
####3.检查是否安装成功
`svnserve --version`
如果输出版本号，则安装成功。
###2.创建SVN仓库
可以一个仓库对应一个项目，也可以一个仓库对应多个项目。每个仓库都需要单独做配置。我这里用的是一个仓库对应一个项目。
建立目录：`mkdir /home/svn/repository`
建立仓库：`svnadmin create /home/svn/repository`
进入/home/svn/repository目录，用`ll`命令查看当前目录下的内容，会发现多了conf, db,hooks, locks目录和format,README.txt文件。
###3.SVN配置
/home/svn/repository/conf下面有三个文件：authz（用户权限配置）,passwd（用户和密码）,svnserve.conf（SVN服务器配置）
####1.passwd（用户和密码）
目的：添加用户并配置密码
[users]
liming=123456
xiaohong=123456
admin=123456
####2.authz（用户分组和权限配置）
目的：为用户分组，为每个组设置可以访问哪个目录的权限
[groups]组别设置，每个组别可以放多个用户，每个组别可以设置
[/]目录权限设置，其中[/]表示所有仓库的根目录。多目录时用[仓库名:/]表示每个仓库的根目录。
```conf
[groups]
admin=admin
manager=liming,xiaohong
#组名前必须加@符号，用户名前不需要加
#admin组中的用户可以读写仓库下的所有文件和目录
[/]
@admin=rw
#manager组中的用户只可以读写仓库下的project目录
[/project1]
@manager=rw
[/project2]
liming=rw
```
如果项目人数较少，也可以不分组，直接分配权限。
```conf
[/]
admin=rw
[/project]
liming=rw
xiaohong=rw
```
注意：权限有继承关系。如果用户有父目录的某个权限，则自动获得所有子目录对应权限，除非加限制（用`*=`）。下例中，user2失去project1:/file1目录的权限：
```conf
[project1:/]
user1=rw
user2=rw
*=
[project1:/file1]
user1=rw
*=
```
####3.svnserve.conf（SVN服务器配置）
```conf
#禁止匿名访问
anon-access = none
#使授权用户有写权限
auth-access = write
#密码文件地址
password-db = /home/svn/repository/passwd
#权限文件地址
authz-db = /home/svn/repository/authz
#认证命名空间，subversion会在认证提示里显示
realm=/opt/svn/repositories
```
####4.开放svn端口
#####1.CentOS7之前的版本，防火墙用iptables（待测试）
默认是3690端口。
修改
`iptables -I INPUT -p tcp --dport 3690 -j ACCEPT`
保存
`/etc/rc.d/init.d/iptables save`
重启
`service iptables restart`
查看
`/etc/init.d/iptables status`
#####2.CentOS7 开始，防火墙用 firewalld 服务（已测试通过）
添加端口：`firewall-cmd --zone=public --permanent --add-port=3690/tcp`
重启防火墙：`firewall-cmd --reload`
查看：`firewall-cmd --list-all`

这里如果报错 `FirewallD is not running` 的话，需要先开启防火墙：`systemctl start firewalld`，再进行配置。
####5.启动SVN
#####1.单仓库：指定仓库对应路径
`svnserve -d -r /home/svn/repository/project1`
-d:守护进程
-r:svn根目录
#####2.多仓库（必须位于同一目录下）：指定所有仓库的公共父目录
`svnserve -d -r /home/svn/repository`
`netstat -ntpl`，查看3690端口是否已经在监听。
####6.设为开机自动启动
#####1.编写一个启动脚本svn_startup.sh，目录可以是/root/svn_startup.sh
```shell
#!/bin/bash
/usr/bin/svnserve -d -r /home/svn/repository
```
其中svnserve命令用绝对路径。绝对路径可以用`which svnserve`查。
#####2.然后修改该脚本的执行权限
`chmod 777 /root/svn_startup.sh`
#####3.加入自动运行
`vi /etc/rc.d/rc.local`
在末尾添加脚本的路径，如：`/root/svn_startup.sh`
#####4.重启试试
`ps -ef|grep svnserve`
如果可以看到svnserve，则设置成功。
###4.SVN使用
SVN服务器端和客户端数据的存放格式不同，在服务器端不可能找到和客户端直接对应的文件。你在SVN服务器的仓库/home/svn/repository中，找不到任何一个用SVN客户端看到的目录或文件。
####1.关联项目到SVN：import
svn import命令执行成功后，SVN客户端就可以看到对应的项目列表了。此时应将原项目文件删除，需要时重新从SVN执行checkout操作，以确保项目的版本可以关联同步。
这里用一个仓库对应多个项目，在Linux本地进行关联（SVN路径前加'file://'）。如果跨主机关联项目，SVN路径前加'svn://IP地址'。
语法：`svn import 项目绝对路径 SVN路径名 -m "日志信息"`
例如：
`svn import /home/testuser/project1 file:///home/svn/repository/project1 -m "improt project1"`
`svn import /home/testuser/project2 file:///home/svn/repository/project2 -m "improt project2"`
####2.从SVN下载项目文件并断开关联：export 
导出某一个版本的数据，导出后项目文件脱离SVN版本控制。导出文件夹下没有.svn目录。
-  从SVN服务器导出：
`svn  export  [-r 版本号]  svn://路径(目录或文件的全路径) [本地绝对路径]　--username　用户名`
- 将本地有关联的项目变为无关联的：
`svn  export  本地检出的(即带有.svn文件夹的)目录全路径  要导出的本地目录全路径`
####3.本地建立受SVN版本控制的项目副本：checkout（简写为svn co）
语法：`svn  checkout  svn://路径(目录或文件的全路径)　[本地绝对路径]  --username　用户名`
例如：`svn checkout svn://119.119.119.119/project1`
本地绝对路径为空时，将会在当前目录中建立项目副本。
####4.向版本库添加文件：add
添加：`svn add file`
例如：`svn add test.php`
####5.向版本库提交文件：commit
当文件修改过或者新添加的，要用commit命令提交到SVN服务器。
提交：`svn commit -m "日志信息" PATH`
例如：`svn commit -m "add test.php` file" test.php`
####6.更新文件至某个版本：update（简写为svn up）
执行update命令前必须确保项目或文件已checkout到本地。
更新：`svn update -r path`
注意：svn update如果后面没有目录，默认将当前目录以及子目录下的所有文件都更新到最新版本。
例如：
`svn update -r 200 test.php`(将版本库中的文件test.php还原到版本200)
`svn update test.php`(更新至版本库中最新版本。)
####7.删除已关联项目或文件：svn delete（简写为svn del/ remove/ rm）
`svn delete svn://119.119.119.119/project1`
###5.常用命令总结
<table>
    <thead>
        <tr>
            <th width="25%">命令</th>
            <th width="25%">功能</th>
            <th>语法</th>
        </tr>
    </thead>
    <tbody>
       <tr>
            <th>import</th>
            <th>关联项目到SVN</th>
            <th>svn import 项目绝对路径 SVN路径名 -m "日志信息"</th>
        </tr>
        <tr>
            <th rowspan="2">export </th>
            <th>从SVN下载项目文件并断开关联</th>
            <th>svn  export  [-r 版本号]  svn://路径(目录或文件的全路径) [本地绝对路径]　--username　用户名</th>
        </tr>
        <tr>
            <th>将本地项目断开关联</th>
            <th>svn  export  本地检出的(即带有.svn文件夹的)目录全路径  要导出的本地目录全路径</th>
        </tr>
        <tr>
            <th>checkout（简写为svn co）</th>
            <th>本地建立受SVN版本控制的项目副本</th>
            <th>svn  checkout  svn://路径(目录或文件的全路径)　[本地绝对路径]  --username　用户名 --password 密码</th>
        </tr>
        <tr>
            <th>add</th>
            <th>向版本库添加文件</th>
            <th>svn add file</th>
        </tr>
        <tr>
            <th>commit</th>
            <th>向版本库提交文件</th>
            <th>svn commit -m "日志信息" PATH</th>
        </tr>
        <tr>
            <th>update（简写为svn up）</th>
            <th>更新文件至某个版本</th>
            <th>svn update -r path</th>
        </tr>
        <tr>
            <th>svn delete（简写为svn del/ remove/ rm）</th>
            <th>删除已关联项目或文件</th>
            <th>svn delete svn://119.119.119.119/project1</th>
        </tr>
    </tbody>
</table>