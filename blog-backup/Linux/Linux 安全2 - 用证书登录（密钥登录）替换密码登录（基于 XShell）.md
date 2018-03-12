证书登录分3步实现：

1. 生成密钥（公钥与私钥，可以用 ssh 客户端生成，也可以用专用软件生成），公钥放在服务器端，私钥放在客户端。为了安全起见，私钥最好设置密码；
2. 放置公钥（Public Key）到服务器的 `~/.ssh/authorized_keys` 文件中；
3. 配置 ssh 客户端（XShell 等）使用证书登录。

注意：记得保存好密钥，如果为密钥设置了密码，要一并保存好。
[深入的知识可以参考这里](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)
#1. 密码登录和证书登录的区别
SSH 是一种网络协议，用于计算机之间的加密登录。SSH 可以用密码登录或证书登录。
SSH 允许密码登录时，每次登录 Linux 服务器时，总是有几千几万失败登录次数。安装了 fail2ban 软件后，数量下降到几百几千，但还是有隐患。密码登录，很容易受到暴力破解攻击，导致出现这种情况。
生产环境要使用证书登录，且最好只允许使用证书登录。
#2. 配置证书
##2.1 用 XShell 生产证书
生产证书的方式挺多的，大多数软件都自带这个功能，这里以 XShell 为例：
####1.  在菜单栏点击“工具”，选择“新建用户密钥生成向导”：
![这里写图片描述](http://img.blog.csdn.net/20180122114358049?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####2.  配置密钥参数，用默认的密钥类型 RSA，密钥长度 2048 位即可：
![这里写图片描述](http://img.blog.csdn.net/20180122114454327?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####3.  点 下一步 生成密钥：
![这里写图片描述](http://img.blog.csdn.net/20180122114548769?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####4.  点 下一步 输入用户的密钥名称和密码（密码最好设置，防止密钥丢失出问题）：
![这里写图片描述](http://img.blog.csdn.net/20180122114709405?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####5.  查看密钥：
通过查看属性，可以改密钥名字，密钥的密码，可以导出公钥或保存为文件。
![这里写图片描述](http://img.blog.csdn.net/20180122114848002?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
##2.2 服务端配置证书
```
# cd /root/.ssh/
# vi authorized_keys 
```
进入 `/root/.ssh/` 目录，把刚才产生的密钥对的公钥复制进 `authorized_keys` 文件就可以了。
#3. 设置 XShell 为证书登录
在会话窗口，右击会话，进入属性页面，设置 `连接->用户身份验证` 的方法和用户密钥，密码。设置完成后，登录方式选 `Public Key` 正常登录即可。注意这里需要输入的密码，是刚才设置的私钥密码，而不是服务器端的用户密码。
![这里写图片描述](http://img.blog.csdn.net/20180122135430556?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#4. 取消密码认证登录
####为了进一步增强安全，编辑 `/etc/ssh/sshd_config` 文件，禁止密码登录：
```
PasswordAuthentication no
```
![这里写图片描述](http://img.blog.csdn.net/20180122140233878?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
#### 重启 sshd 服务
`/etc/ssh/sshd_config` 文件的改动要想生效，需要重启 sshd 服务。CentOS 7 的命令如下：
```
systemctl restart sshd
```
然后再重新连接服务器时，会发现没法选择密码登录。对，用密钥和密钥的密码登录吧。
![这里写图片描述](http://img.blog.csdn.net/20180122141250510?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)