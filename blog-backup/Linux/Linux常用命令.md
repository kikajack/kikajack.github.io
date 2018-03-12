Linux命令分为两类：内建命令（shell自带，如cd、pwd）和外部命令（独立的可执行文件，文件名即命令名，如ls、mv、ps，通过shell内置的环境变量$PATH指定的路径查找，可用which查看命令所在路径），用type可以查看具体的命令类型（`type cd`）。


###1.系统信息
1. date
按指定格式查看或设置系统时间
2. /proc目录
Linux的伪文件系统，只占内存。可以在运行时访问内核内部数据结构、改变内核设置。以文件系统的方式为访问系统内核数据的操作提供接口
`cat /proc/version 查看系统版本`
`cat /proc/devices 已经加载的设备并分类`
`cat /proc/cpuinfo 显示CPU info的信息 `

###2.登录登出，关机重启
1. `shutdown -h now 关闭系统`
`shutdown -h hours:minutes & 按预定时间关闭系统`
`shutdown -c 取消按预定时间关闭系统`
2. reboot 重启

###3.文件，目录
1. cd
改变工作目录。可以用绝对路径（/home/root）或相对路径（./当前目录，../上层目录，-之前的目录）。
2. pwd
显示当前工作目录。
3. ls
查看目录中的文件。
`ls -l 显示文件和目录的详细资料`
`ls -a 显示隐藏文件 `
4. rm
删除文件
5. rmdir
删除目录
`rm -f file1 删除一个叫做 'file1' 的文件'`
`rmdir dir1 删除一个叫做 'dir1' 的目录'`
`rm -rf dir1 删除一个叫做 'dir1' 的目录并同时删除其内容`
`rm -rf dir1 dir2 同时删除两个目录及它们的内容 `
6. mkdir
新建目录
`mkdir dir1 dir2 同时创建两个目录`
`mkdir -p /tmp/dir1/dir2 创建一个目录树 `
6. mv
重命名或移动
`mv a.txt b.txt  将a.txt 改名为b.txt`
`mv    /usr/lib/*    /001 将 /usr/lib/下所有的东西移到/001/中`
`mv    /usr/lib/    /001 将lib和其内部的所有东西移到/001/中。 此后，/usr里不再有lib; /001里有lib/及其原有的东西`
`mv dir1 new_dir 重命名/移动 一个目录 `
6. cp
`cp file1 file2 复制一个文件 `
`cp -a dir1 dir2 复制一个目录 `
7. ln
建立硬链接或符号链接。
`ln -s 源文件 目标文件 `
`ln -s file1 lnk1 创建一个指向文件或目录的软链接 `
8. touch
更新文件时间戳，新建文件
` touch -t 0712250000 file1 修改一个文件或目录的时间戳 - (YYMMDDhhmm) `
11. find
文件搜索。
`find   -name april* 在当前目录下查找以april开始的文件`
12. locate
`locate \*.ps 寻找以 '.ps' 结尾的文件 - 先运行 'updatedb' 命令`
13. whereis
`whereis halt 显示一个二进制文件、源码或man的位置`
14. which
`which halt 显示一个二进制文件或可执行文件的完整路径 `
15. 查看文件内容
`cat file1 从第一个字节开始正向查看文件的内容`
`tac file1 从最后一行开始反向查看一个文件的内容`
`more file1 查看一个长文件的内容`
`less file1 类似于 'more' 命令，但是它允许在文件中和正向操作一样的反向操作`
`head -2 file1 查看一个文件的前两行`
`tail -2 file1 查看一个文件的最后两行`
`tail -f /var/log/messages 实时查看被添加到一个文件中的内容 `

###4.磁盘
1. mount
挂载文件系统
`mount /dev/hda2 /mnt/hda2 挂载一个叫做hda2的盘 - 确定目录 '/ mnt/hda2' 已经存在`
`umount /dev/hda2 卸载一个叫做hda2的盘 - 先从挂载点 '/ mnt/hda2' 退出 `
2. df
磁盘空间占用情况
`df -h 显示已经挂载的分区列表`
3. du
du的英文原义为“disk usage”，含义为显示磁盘空间的使用情况，统计目录（或文件）所占磁盘空间的大小。该命令的功能是逐级进入指定目录的每一个子目录并显示该目录占用文件系统数据块（1024字节）的情况。若没有给出指定目录，则对当前目录进行统计。
`du 列出各目录所占的磁盘空间，但不详细列出每个文件所占的空间`
4. fdisk
划分磁盘分区
`#fdisk /dev/had  使用/dev/had作为默认的分区设备`
5. badblocks -v /dev/hda1 检查磁盘hda1上的坏磁块
fsck /dev/hda1 修复/检查hda1磁盘上linux文件系统的完整性 
fsck.ext3 /dev/hda1 修复/检查hda1磁盘上ext3文件系统的完整性 
6. 备份
dump -0aj -f /tmp/home0.bak /home 制作一个 '/home' 目录的完整备份
dump -1aj -f /tmp/home0.bak /home 制作一个 '/home' 目录的交互式备份 

###5.用户，群组
`groupadd group_name 创建一个新用户组`
`groupdel group_name 删除一个用户组`
`groupmod -n new_group_name old_group_name 重命名一个用户组`
`useradd -c "Name Surname " -g admin -d /home/user1 -s /bin/bash user1 创建一个属于 "admin" 用户组的用户`
`useradd user1 创建一个新用户`
`userdel -r user1 删除一个用户 ( '-r' 排除主目录)`
`usermod -c "User FTP" -g system -d /ftp/user1 -s /bin/nologin user1 修改用户属性`
`passwd 修改口令`
`passwd user1 修改一个用户的口令 (只允许root执行) `
`chown user1 file1 改变一个文件的所有人属性 `
`chgrp group1 file1 改变文件的群组 `
`chmod 755 /home/root/work 改变权限`

###6.压缩
`gunzip file1.gz 解压一个叫做 'file1.gz'的文件`
`gzip file1 压缩一个叫做 'file1'的文件 `
`tar -cvfj archive.tar.bz2 dir1 创建一个bzip2格式的压缩包`
`tar -xvfj archive.tar.bz2 解压一个bzip2格式的压缩包 `
`tar -cvfz archive.tar.gz dir1 创建一个gzip格式的压缩包 `
`tar -xvfz archive.tar.gz 解压一个gzip格式的压缩包 `

###7.网络
1. 网络 - （以太网和WIFI无线）
ifconfig eth0 显示一个以太网卡的配置
ifup eth0 启用一个 'eth0' 网络设备
ifdown eth0 禁用一个 'eth0' 网络设备
ifconfig eth0 192.168.1.1 netmask 255.255.255.0 控制IP地址
ifconfig eth0 promisc 设置 'eth0' 成混杂模式以嗅探数据包 (sniffing)
dhclient eth0 以dhcp模式启用 'eth0'
route -n show routing table 
netstat 
tcpdump 
host
whois

###8.系统服务
1. 查看Linux启动的服务
chkconfig --list 查询出所有当前运行的服务
chkconfig --list atd  查询atd服务的当前状态
2. 停止所有服务并且在下次系统启动时不再启动，如下所示：
chkconfig --levels 12345 NetworkManager off
如果想查看当前处于运行状态的服务，用如下语句过滤即可
chkconfig --list |grep on
3. 如果只是想当前的设置状态有效，在系统重启动后即不生效的话，可以用如下命令停止服务
service sshd stop
4. Linux定时任务有:cron、anacron、at。
cron是服务名称，crond是后台进程，crontab则是定制好的计划任务表。
查看crond服务是否运行：
pgrep crond或/sbin/service crond status或ps -elf|grep crond|grep -v "grep"
crond服务操作命令:
/sbin/service crond start //启动服务  
/sbin/service crond stop //关闭服务  
/sbin/service crond restart //重启服务  
/sbin/service crond reload //重新载入配置
配置定时任务：cron有两个配置文件，一个是一个全局配置文件（/etc/crontab），是针对系统任务的；一组是crontab命令生成的配置文件（/var/spool/cron下的文件），是针对某个用户的.定时任务配置到任意一个中都可以。
查看全局配置文件配置情况: cat /etc/crontab
查看用户下的定时任务:crontab -l或cat /var/spool/cron/用户名
查看用户下的定时任务:crontab -l或cat /var/spool/cron/用户名
crontab任务配置基本格式：
>`*　　*　 *　 *　 *　　command`
分钟(0-59)　小时(0-23)　日期(1-31)　月份(1-12)　星期(0-6,0代表星期天)　 命令
第1列表示分钟1～59 每分钟用*或者 */1表示
第2列表示小时1～23（0表示0点）
第3列表示日期1～31
第4列表示月份1～12
第5列标识号星期0～6（0表示星期天）
第6列要运行的命令
在以上任何值中，星号（*）可以用来代表所有有效的值。譬如，月份值中的星号意味着在满足其它制约条件后每月都执行该命令。
整数间的短线（-）指定一个整数范围。譬如，1-4 意味着整数 1、2、3、4。
用逗号（,）隔开的一系列值指定一个列表。譬如，3, 4, 6, 8 标明这四个指定的整数。
正斜线（/）可以用来指定间隔频率。在范围后加上 /<integer> 意味着在范围内可以跳过 integer。譬如，0-59/2 可以用来在分钟字段定义每两分钟。间隔频率值还可以和星号一起使用。例如，*/3 的值可以用在月份字段中表示每三个月运行一次任务。
开头为井号（#）的行是注释，不会被处理。 
`0 1 * * * /home/testuser/test.sh每天晚上1点调用/home/testuser/test.sh`
`*/10 * * * * /home/testuser/test.sh每10钟调用一次/home/testuser/test.sh`
`30 21 * * * /usr/local/etc/rc.d/lighttpd restart上面的例子表示每晚的21:30重启`apache。`
`45 4 1,10,22 * * /usr/local/etc/rc.d/lighttpd restart上面的例子表示每月1、10、22日的4 : 45重启apache。`
`10 1 * * 6,0 /usr/local/etc/rc.d/lighttpd restart上面的例子表示每周六、周日的1 : 10重启apache。`
`0,30 18-23 * * * /usr/local/etc/rc.d/lighttpd restart上面的例子表示在每天18 : 00至23 : 00之间每隔30分钟重启apache。`
`0 23 * * 6 /usr/local/etc/rc.d/lighttpd restart上面的例子表示每星期六的11 : 00 pm重启apache。`
`* */1 * * * /usr/local/etc/rc.d/lighttpd restart每一小时重启apache`
`* 23-7/1 * * * /usr/local/etc/rc.d/lighttpd restart晚上11点到早上7点之间，每隔一小时重启apache`
`0 11 4 * mon-wed /usr/local/etc/rc.d/lighttpd restart每月的4号与每周一到周三的11点重启apache`
`0 4 1 jan * /usr/local/etc/rc.d/lighttpd restart一月一号的4点重启apache`
`*/30 * * * * /usr/sbin/ntpdate 210.72.145.44每半小时同步一下时间`
配置用户定时任务的语法：
crontab [-u user]file
crontab [-u user] [-l| -r | -e][-i]
参数与说明：
crontab -u//设定某个用户的cron服务
crontab -l//列出某个用户cron服务的详细内容
crontab -r//删除没个用户的cron服务
crontab -e//编辑某个用户的cron服务





###9.
4. ps命令
该命令用于将某个时间点的进程运行情况选取下来并输出，process之意
5. kill命令
该命令用于向某个工作（%jobnumber）或者是某个PID（数字）传送一个信号，它通常与ps和jobs命令一起使用
6. time命令
该命令用于测算一个命令（即程序）的执行时间.在程序或命令运行结束后，在最后输出了三个时间，它们分别是：
user：用户CPU时间，命令执行完成花费的用户CPU时间，即命令在用户态中执行时间总和；
system：系统CPU时间，命令执行完成花费的系统CPU时间，即命令在核心态中执行时间总和；
real：实际时间，从command命令行开始执行到运行终止的消逝时间；
`time ./process`
`time ps aux`