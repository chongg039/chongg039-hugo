+++
date = "2016-10-03T20:32:20-07:00"
draft = false
title = "使用vultr+SS+锐速科学上网"
Tags = ["server","vultr"]
+++
接手工作室后，tesths 便把原来那个已经配置好的ss账号连同服务器一块给销了。平时ss又十分必要，就向他讨要了具体的方法，自己搭了一个。这里也是把方法贴出来。

### 配置vultr主机
V家的主机算得上是性价比较高的一款了，而且有日本和新加坡的线路，相对来说延迟比较稳定。截止我购买的时候是每个新用户有20刀的返利，还是比较划算的。
#### 在 https://www.vultr.com/ 创建用户
![ ](http://7xswbj.com1.z0.glb.clouddn.com/vultr-create-account.png  "vultr-create-account")

进来后需要绑定账户。如果没有信用卡的话建议使用PayPal，而且PayPal应该是支持国内的储蓄卡绑定的，比较方便。

第一次使用需要先充值5刀，还是比较便宜的，之后会说到vultr主机的扣费机制。

在图的右边，这次我注册了一个新账号，但是很不幸的显示返利变少了。。。好吧你要是能找到优惠码的话也许还会有。。。

#### 配置主机
![ ](http://7xswbj.com1.z0.glb.clouddn.com/vultr-setting-servers.png  "vultr-setting-servers")

这是我的vultr主界面面板，可以看到已经有了两个在运行的服务器。如果开始要新建一个服务器，点击右上方的+号：

![ ](http://7xswbj.com1.z0.glb.clouddn.com/vultr-server.png  "vultr-server")
>这里选择日本的服务器，经过几天的测试发现日本的要比新加坡的线路快不少
操作系统选择自己熟悉的就好，我选的ubuntu14.04，不建议windows
如果只是为了搭建番茄的梯子，配置最低配那个就行，一个月总计才5刀，性能完全满足需求
有需要的话可以勾选IPV6，我们校区没有就不搞了

配置完后deploy就会生成相应的vultr服务器了主界面像下面这样：

![ ](http://7xswbj.com1.z0.glb.clouddn.com/vultr-server-info.png  "vultr-server-info")

显示了ip地址，初始root密码和流量、费用的使用。

#### ssh到vultr服务器
打开终端，输入
>ssh root@你的vultr主机ip

第一次验证选择'YES'，输入初始密码后便连上了vultr服务器。

>可以 `passwd root` 修改初始密码

### 安装shadowsocks-python服务器版
shadowsocks服务器有很多版本，go、python、libev、R都可以。安装方式可以手动配，我自己的ubuntu电脑就是用这种比较原始的办法配的，需要手动编写 /etc/shadowsocks.json 文件，还得写一个开机启动和后台运行脚本。不过确实比较有成就感，而且一劳永逸。

然而那天我们美工告诉我 [秋水大大](https://teddysun.com/) 写了一个自动生成脚本，感觉世界观被颠覆了。不过确实好用，这里用他的 [Shadowsocks Python版一键安装脚本](https://teddysun.com/342.html) 来安装到我们的vultr服务器中：
```
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh

chmod +x shadowsocks.sh

./shadowsocks.sh 2>&1 | tee shadowsocks.log
```
按照脚本提示输入就可以，安装完后提示如下：
```
Congratulations, shadowsocks install completed!
Your Server IP:your_server_ip
Your Server Port:your_server_port
Your Password:your_password
Your Local IP:127.0.0.1
Your Local Port:1080
Your Encryption Method:aes-256-cfb

Welcome to visit:https://teddysun.com/342.html
Enjoy it!
```
>tesths 建议我们用8388端口，说这个经他测试比较稳定。虽然知道他在满嘴跑火车，但是还是忍不住用了。。。

可以使用下面的命令操作shadowsocks：
```
启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
状态：/etc/init.d/shadowsocks status
```

现在不论你的电脑是windows、mac还是linux，使用ss客户端（自行查找下载）输入ip、端口和密码应该就可以番茄了。可以打开 https://fast.com/ 测一下速。
>说实话买到的vultr主机的ip，番茄快不快完全看人品，根本不知道这个节点速度怎样。
所以我建议多买几个主机，多用几个ip试试，因问V家的主机不是一次性收费一个月的5刀，而是按时间计费。加上他给的返利，可以多几个ip试用几天，留下稳定的那个服务器，剩下的再删掉即可。

### 配置锐速
有很多的服务器加速工具，[锐速](http://www.serverspeeder.com/) 应该是相当不错的一家，又不会无脑占用国际带宽。不过不知道官方怎么了，我们安装破解版就可以。

锐速破解版也有一键安装脚本，我参考的是 [这里](https://www.91yun.org/archives/683) 的方法。

#### 安装锐速
```
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/serverspeeder/master/serverspeeder-all.sh && bash serverspeeder-all.sh
```

>如果安装失败，一般会提示锐速支持的内核和我们的不匹配，可以 http://dl.serverspeeder.com/ls.do?m=availables 查看支持的内核

#### 更换内核
```
dpkg -l|grep linux-image
//查看当前安装的内核
apt-get install linux-image-3.13.0-96-generic linux-image-extra-3.13.0-96-generic
//安装锐速支持的内核，如果不支持，可能会选择一个比较近的版本下载
sudo apt-get purge linux-image-3.13.0-xx-generic linux-image-extra-3.13.0-xx-generic
//卸载第一步看到的不支持的内核
update-grub
//更新系统引导文件
reboot
//重启后生效
```

这样再去执行一键安装脚本，锐速就可以顺利安装了。

理论上速度应该会有所提升，而且还比较明显。

最后我只想说，脚本真是个好东西啊。。。
（默哀一下阿里的月饼员工们）