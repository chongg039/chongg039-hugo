+++
date = "2016-10-02T15:43:36-07:00"
draft = false
title = "解决 ubuntu 下键盘错位问题"
Tags = ["ubuntu","疑难杂症"]
+++
很悲催的键盘错位了，就这个系统有问题，以前也没遇到过。简而言之就是@和“互换，#打不出来等。问题不大，但是影响输入体验。
工作室的小伙伴都说我这电脑只有我会用。。。

之前在网上搜到的解决办法是在终端输入：
>sudo dpkg-reconfigure keyboard-configuration
//需要安装keyboard-configuration

然后调整键盘布局，改为English(US)或者English(UK)，这样就相当于把键盘布局重置，然后就好了。

这种办法也可以通过`sudo vim /etc/default/keyboard`手动修改配置：
> KEYBOARD CONFIGURATION FILE

> Consult the keyboard(5) manual page.

> XKBMODEL="cherrycyboard" //修改键盘类型
XKBLAYOUT="us" //修改语言类型：us,uk,cn...
XKBVARIANT=""
XKBOPTIONS=""

> BACKSPACE="guess"

**但是！！！**这样修改一会键盘就变回去了啊啊！！！一开始是一天，后来越来越快，今天连一分钟也没撑住啊啊！！！
弄得人心里很是崩溃。。。

继续看吧，有人说需要在配置完`sudo vim /etc/default/keyboard`后，终端中运行一次
>setupcon

但是输出了

>We are not on the console, the console is left unconfigured.

原来要进入文字终端，虚拟终端还不行。。。
执行完毕后，重启或者执行一次
>sudo udevadm trigger --subsystem-match=input --action=change

使得保存的配置生效。
这样一般就成功了，若是还不行，有可能是fcitx输入法的问题。
打开 Fcitx Configuration，可以在search里找到
![ ](https://c1.staticflickr.com/4/3766/33457473611_5ab96b6472_b.jpg)
像我上面选择的键盘语言是
>XKBLAYOUT="us" //修改语言类型：us,uk,cn...

需要把当前语言移动到最上面，没有的话就添加一个。

到这里就可以解决大部分问题了，至少我这里键盘输入还没有回弹到那种奇葩的状况。
那就必须秀一波@@@@@@@########""""""""""啊哈哈！
![ ](https://c1.staticflickr.com/4/3666/33457496511_69a2c3bdae_z.jpg)
---
#### 参考文档
 - [ubuntu14.04键盘错位小问题](http://www.linuxdiyf.com/linux/16832.html) 
 - [ubuntu/mint键盘错乱问题](http://www.linuxdiyf.com/linux/17450.html)
 - [Ubuntu下键盘输入错乱问题](http://blog.csdn.net/cc7756789w/article/details/50661992)  

