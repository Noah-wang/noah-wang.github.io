---
title: 在小米AX3600上安装Clash
date: 2025-01-08 20:07:14
index_img: https://clashcn.com/wp-content/uploads/2023/05/clash-300x300.png
excerpt: 一个方便自己和大家的文章
category: 开发-----记录，学习，分享
---

### 这是一个在小米AX3600路由器上安装clash插件的过程

在路由器上安装网络代理工具的好处👍是只需要安装一次，全屋设备都有了代理的网络，对于电视，VR等平台封闭的硬件来说这似乎是比较现实的唯一解。坏处⚠️就是门槛相比手机电脑上的网络代理软件比较高。

选择安装Clash需要有一些硬件准备：

Option A：软路由方案：专门的硬件+Linux系统+路由程序，所以很多人可能不愿意专门为了网络而再去准备硬件。

Option B：硬路由方案：特定的某些路由器+路由程序，这种是非常方便的方案，正好借用了路由器的功能实现代理网络。

我这里就是用的第二种方法，选择的设备是小米的AX3600，现在大概300多块

![这个就是AX3600](https://th.bing.com/th/id/R.2c5d008cf548047d7f2788d7ad611893?rik=wsC09Ex2613C4g&riu=http%3a%2f%2fimg1.mydrivers.com%2fimg%2f20200215%2fb5dca22f1b654f01980938ac460f39d0.jpg&ehk=sveu4rp6UDTuhIBMVgK1HtMMnrGaNpkH6aziRKCECkA%3d&risl=&pid=ImgRaw&r=0)

#### 为什么我选择了安装Clash呢？

我今年暑假从国内带回来了大一买的Oculus Quest 2和Xbox Series S，这些设备只有在国外的网络才能发挥全部的功能，所以暑假一直在寻找一个无痛的方案来创造一个沉浸的网络。

暑假的时候其实已经成功完成了Clash的安装和网络的部署，由于机场到期，家里扫地机器人接入的网络就没办法用了，就把路由器重置了。寒假回国之后电脑都换了一遍，这些设备又需要网络，我就又重新弄了一遍，这一次也是记录下来我遇到的问题和解决方案。

#### AX3600安装Clash的全过程

#### Step1 检查路由器的版本

检查路由器的版本是否为1.0.17，

![检查这个位置](https://github.com/Noah-wang/pictures/blob/main/picture/%E5%B0%8F%E7%B1%B3Ax3600Clash/checkingVersion.png?raw=true)

如果不是需要下载下面这个链接，安装需要的路由器版本。里面还包括了Step5要用的程序

https://pan.baidu.com/s/17LWkIHdcuhNcHFWbpLQdmA 提取码: t1gg

如果是的话，you are free to go to the next step

#### Step2 获取Stok

登陆到小米路由器管理网站：在浏览器输入 192.168.31.1 登陆进入网站

网站的地址上找到**stok=**的部分一直到**/web**复制中间的内容，这一串字符是我们需要的

#### Step3 解锁SSH

把上一步stok替换下面<STOK>输入到浏览器，正确的话会出现**{code 0}**

`http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20nvram%20set%20ssh_en%3D1%3B%20nvram%20commit%3B%20sed%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%5C%22debug%5C%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%3B%20%2Fetc%2Finit.d%2Fdropbear%20start%3B`

这一个也是一样的操作，把上一步stok替换下面<STOK>输入到浏览器，正确的话会出现**{code 0}**

`http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20echo%20-e%20'admin%5Cnadmin'%20%7C%20passwd%20root%3B`

在终端执行下面的内容

`ssh root@192.168.31.1`

⚠️但在这一步的时候我遇到了下面的问题

![ssh连接问题](https://github.com/Noah-wang/pictures/blob/main/picture/%E5%B0%8F%E7%B1%B3Ax3600Clash/step3problem.png?raw=true)

在网络上搜索后，下面的这个网站解决了

https://www.right.com.cn/FORUM/thread-8238692-1-1.html

`ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.31.1`

我问了Chatgpt，回答了关于我的这个问题

*“When you attempt to connect via SSH, the client and server negotiate which algorithms to use for encryption, key exchange, and authentication. If the server offers ssh-rsa (commonly used by older systems or devices) and your client doesn’t support it by default, the negotiation fails, resulting in the error.”*

*“当你尝试通过 SSH 进行连接时，客户端和服务器会协商使用哪种算法进行加密、密钥交换和身份验证。如果服务器提供 ssh-rsa（旧版系统或设备常用），而你的客户端默认不支持它，协商就会失败，从而导致错误"。*



**看到大大的ARE U OK就成功了**

#### Step4 备份路由器

在小米 AX3600 上执行

```
mkdir /tmp/syslogbackup/
dd if=/dev/mtd9 of=/tmp/syslogbackup/mtd9
```

随后在http://192.168.31.1/backup/log/mtd9下载备份

#### Step5 固化SSH

在Terminal里打开在百度网盘下载的fuckAX3600路径

输入`scp fuckax3600 root@192.168.31.1:/tmp`

⚠️在这一步我还是遇到了问题`ash: /usr/libexec/sftp-server: not found`

通过搜索在该网址找到了解决办法：https://forum.openwrt.org/t/ash-usr-libexec-sftp-server-not-found-when-using-scp/125772

`scp -O -oHostKeyAlgorithms=+ssh-rsa fuckax3600 root@192.168.31.1:/tmp`

然后更改系统权限：

`chmod +x /tmp/fuckax3600 `

`/tmp/fuckax3600 unlock`

系统重启之后在scp一遍程序

`scp -O -oHostKeyAlgorithms=+ssh-rsa fuckax3600 root@192.168.31.1:/tmp`

然后SSH连接到路由器执行：

`chmod +x /tmp/fuckax3600 `

`/tmp/fuckax3600 hack `

`/tmp/fuckax3600 **lock**`

然后SSH会出现一个密码，需要记录下来

#### Step6 安装ShellClash

SSH进入到小米路由器安装ShellClash，在下面这个链接里选择一个可以安装的源，直接复制粘贴地址到Terminal

`https://github.com/juewuy/ShellCrash/blob/master/README_CN.md`

等待安装完毕，随后一路输入1回车到主界面。此时ShellClash就安装完成了。

在root@xiaoqiang的用户名下输入clash就能看见菜单了，选择导入配置文件就可以把机场的订阅信息输入进去。如果没有提供订阅信息可以选择在线生成，输入机场提供链接，clash会给你生成很多的订阅地址。可以输入到浏览器看信息是否正确，选择一个链接输入就好了。我们还可以安装面板，在浏览器就可以查询到数据。我们可以选择开机启动和每日定期重启。

后面我们如果需要再次登陆进去就只需要SSH连接到路由器，输入Step5最后得到的密码，进入root@xiaoqiang，输入clash调出Clash菜单。

### 完成所有工作

此时我们的Clash就安装完毕了，其实不算很难，但每个人遇到的问题不一定一样。所以我把我的问题和解决方案都发出来提供参考，希望可以帮助了在这个过程中遇到困难的人。
