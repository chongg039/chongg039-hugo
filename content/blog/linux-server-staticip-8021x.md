+++
date = "2016-11-12T15:15:27-07:00"
draft = false
title = "Linux 服务器配置静态 IP 并 802.1x 拨号"
Tags = ["server","linux","8021x"]
+++
终于布好工作室的服务器了。

我写了个脚本一键部署静态ip和802.1x拨号：[ubuntu-server.sh](https://gist.github.com/chongg039/3310467e329e00de3b6b04aaae646f3c)，不想看我啰嗦的可以直接拿过来改改就能用。

先讲个小故事：

工作室项目配了HTTPS，必须要域名访问，可是又没钱，买不起性能好的云服务器怎么办？

看了看角落里有一台遗留下来的服务器，至强12核 + 32G内存 + 120G SSD，性能不错，就是它了。跑到网络中心问了问，以前这台的MAC地址学校就给分配了一个静态IP，而且现在还能用。很好呀，交了证明材料，拿回了一些信息就准备开工。

下面就是正题了，针对ubuntu server版进行说明：

在配置之前，首先得确定几件事情：

- 确保你有一个分配到的公网ip，查清楚它的网关和子网掩码，还有本地dns
- 检查服务器有几个网卡，到底是哪个网卡的MAC地址绑定了分配给你的公网ip（我们服务器是双网卡，绑定的是`enp7s0f0`，注意下文换成自己的）
- 确保网线插到了对应的网卡接口上
- 有一个可以802.1x拨号的上网账号

#### 首先尝试一下用dhcp拨号上网：

服务器版使用了`wpasupplicant`拨号：

```
sudo apt-get install wpasupplicant
```

编辑`/etc/wpa_supplicant.conf`文件:

```
# Where is the control interface located? This is the default path:
ctrl_interface=/var/run/wpa_supplicant

# Who can use the WPA frontend? Replace "0" with a group name if you
# want other users besides root to control it.
# There should be no need to chance this value for a basic configuration:
ctrl_interface_group=0

# IEEE 802.1X works with EAPOL version 2, but the version is defaults
# to 1 because of compatibility problems with a number of wireless
# access points. So we explicitly set it to version 2:
eapol_version=2

# When configuring WPA-Supplicant for use on a wired network, we don’t need to
# scan for wireless access points. See the wpa-supplicant documentation if
# you are authenticating through 802.1x on a wireless network:
ap_scan=0

network={
 ​        key_mgmt=IEEE8021X
         eap=TTLS MD5
 ​        identity="yourUsername"
         anonymous_identity="yourUsername"
 ​        password="yourPassword"
 ​        phase1="auth=MD5"
 ​        phase2="auth=PAP password=yourPassword"
 ​        eapol_flags=0
}
```

替换成你自己的账号和密码

然后修改`/etc/network/interfaces`:

```
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto enp7s0f0
iface enp7s0f0 inet dhcp

wpa-driver wired

wpa-conf /etc/wpa_supplicant.conf
```

保存，重启下网卡：

```
sudo /etc/init.d/networking restart
```

如果不能重启，提示进程占用的话就先kill wpa进程：

```
sudo killall -q wpa_supplicant
```

再重启，成功后拨号：

```
sudo wpa_supplicant -c /etc/wpa_supplicant.conf -B -D wired -i enp7s0f0
```

使用`-B`后台运行，成功后可以ping一下地址，ifconfig看看有没有分配到ip，没有的话reboot一下，一般是没问题的。

#### 配置静态ip

这里只需要修改`/etc/network/interfaces`中`enp7s0f0`部分：

```
# The primary network interface
auto enp7s0f0
iface enp7s0f0 inet static
         address 222.197.176.***
         netmask 255.255.255.***
​         network 222.197.176.128
 ​        broadcast 222.197.176.255
 		 gateway 222.197.176.***
​		 # dns-* options are implemented by the resolvconf package, if installed
		 dns-nameservers 202.112.14.***
```

理论上只需要填写`address、netmask、gateway、dns`即可，我在安装ubuntu server时联网填入了这些，自动计算出了`network`和`broadcast`。

重启网卡后再拨号一下，应该就绑定好静态ip并且能上网了，不过建议还是reboot一下。

可以装个`openssh-server`，用别的机子ssh一下这个ip试试。

#### 后记：

- 一定要确认给你的静态ip到底绑定的是哪个网卡的MAC地址，并插好了网线，否则就像我们一样绑定好了ip也上不了公网
- 我们学校要求活动中心这边的服务器使用的静态ip，必须用电信给的办公区的上网账号才能拨号，学生宿舍的账号是不行的，给的波段不一样。在配置前一定要去网络中心和电信营业厅问清楚
- 后来装回了ubuntu desktop版，发现直接在network设置里的图形化界面配置就行了，哪这么多破事。。。这充分说明了一开始我们搞不定就是用错了网卡
- 这篇文章主要为了留给工作室以后的学弟学妹们，以及像我们学校一样，仍在使用802.1x拨号的苦逼创业团队用，随意转载



