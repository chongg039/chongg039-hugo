+++
date = "2016-05-12T19:29:07-07:00"
draft = false
title = "搭建 Linux 开发环境"
Tags = ["ubuntu","nodejs"]
+++
![ubuntu下配置nodejs工作流](http://7xswbj.com1.z0.glb.clouddn.com/ubuntunodejs.jpg)

>先说说我为啥要干这事儿。
前段时间迪哥要我们用node搞一个最小化 MEAN 产品，没想到只是在搭建开发环境上windows就各种不兼容，出现了许多奇奇怪怪的bug，再加上国内关于node的开发教程确实相对比较少，没有好的学习资源。搞了virtualbox虚拟机，但是内存和性能又开始捉急。干脆换到ubuntu，一步到位，特此记录下过程。

- 系统版本：ubuntu14.04LTS
- 开发环境：nodejs4.4.4LTS， Express， Yeoman, chrome
- 写作环境：sublimetext3， haroopad（md）

>在ubuntu上搭建node和前端开发流网上大部分都有教程，我这里就不一一写出来了，只是记下搭建时应该注意的事项和几个比较好的网站供参考

### 开始搭建

####安装ubuntu14.04

没啥可说的，搞个系统盘装就行了，就是安装的时候注意盘的分区设置（自行google）并且最好选择英文。有几个可能出现的小问题：

- 如果以前安装过ubuntu系统想要删除，不能直接在系统盘下进行格式化，需要先修改win系统下的启动文件，否则就是血的教训（别问我怎么知道的）
- 有时候进入ubuntu系统时会出现配平和一个警告框，大意是警告在低电源出什么问题，这种问题就是你的显卡驱动没有更新，可以先妥协一下点ok重启进入ubuntu，或者进入命令行模式安装即可，办法自己找。当然第一次进入系统时一般不会出现这种问题，所以出于方便考虑*建议第一次进系统时就检测一下驱动并且更新*
- **还有一个大坑**，由于经常使用`sudo`命令要输入密码，总会有人偷懒想要设置自己的用户直接拥有root权限而省去这一步，网上的教程修改系统配置文件里增加自己的用户为root用户。当然这种办法是可行的。但一旦不小心在写配置文件时写错一个地方就悲剧了，你会发现你连**sudo命令都无法使用！！！**这就意味着你再也不能在图形界面下将配置文件打开恢复原样了。。。当然也不是没有办法，就是需要进入命令行模式下一顿操作。。。这里我不贴出具体的方法，因为我觉得当时人们设计这种模式是有道理的，限制对sudo命令的使用来保证系统的安全性。我们的目的还是在nodejs开发上，只要不搞linux的话我们还是老老实实`sudo`好了，不用输入密码完全可以通过`sodo -s -H`等办法解决。。。

####一些必要的配置

- `sudo apt-get update`没事就用一下
- 换源，国内的话换成阿里的源就够用了，在`system settings`里的`software&update`的`download from`目录下修改
- 搜狗输入法，做得还是不错的，需要换`fcitx`框架，官网下载deb包。装完后记得重启才能生效
>注意安装了fcitx后不要删除原有的iBus框架，否则会导致你的图形界面会发生各种异常（比如哪里的图标都可能会消失，图形界面也有可能消失哦，那样又要去命令行系统里重新搞了，iBus留着即可


####科学上网
- 工作室拿shadowsocks爬墙，还是很方便，只是换了岛国的代理后只有后台架梯子chrome还上不了墙，还得搞个switchomega。有篇@[爱探路](https://aitanlu.com/ubuntu-shadowsocks-ke-hu-duan-pei-zhi.html)博客介绍的比较详细直接转过来供参考
- 据说小飞机的账号网上很好找，没有的话用lantern也不错，官网上有linux版本提供deb包下载

####安装nodejs
- apt-get 支持安装nodejs命令，不过版本过低，还不能升级，据说是linux官方源不再更新了，然而有的框架和命令还必须要新版本的才能执行。apt-get安装的版本还不支持node命令，只能用nodejs命令替代（这就非常尴尬了）。因此，墙裂推荐去node官网下载最新的nodejs版本自行安装，麻烦一次受益一生。**推荐使用二进制文件进行安装，比较靠谱，安装过程也可以顺便熟悉一下linux命令**
- linux中国开源社区有[一篇文章](https://linux.cn/article-5766-1.html)做了比较详细的介绍，当然你需要去[nodejs官网](https://nodejs.org/en/)看当前最新的稳定版本是哪个然后下载对应的的二进制源码
- 安装完nodejs后自带了npm包，不过版本比较低，通过`sudo npm update npm -g`升级到最新版本。没事可以再`update`一下
- 注意环境配置，如g++和python2.7.x（检查系统内自带的python版本）

####配置工作流
- 正常安装Express步骤：注意高版本的要安装`express-generator`
- 正常安装Yeoman步骤：安装条件`sudo apt-get install build-essential openssl libssl-dev curl`
顺手把Ruby装了算了`sudo apt-get install ruby`以后需要啥提示的时候在装就行了
然后`sudo npm install -g yo grunt-cli bower`，`gulp`有兴趣装了也行
- 基本配置完了，顺顺当当，总归不会出现windows命令行下各种不能操作的毛病
（安装时间有点长就顺手装了个命令行版的[网易云音乐](https://github.com/darknessomi/musicbox)，说实话比想象中好用多了。就是我这里登陆是GG的，有时间再找找问题||-_-）

####安装git
- 推荐@[LittleQ](http://sjq597.github.io/2015/10/25/Ubuntu-14-04-%E5%AE%89%E8%A3%85git/)的这篇文章，比较清晰可靠，同时给配置了SSH

####配置写作环境
- 去[sublimetext官网](https://www.sublimetext.com/)下载对应的deb包安装。插件我就不介绍了，一搜一大堆
- markdown编辑器推荐[haroopad](http://pad.haroopress.com/user.html)，这是我目前在使用的编辑器，功能比较齐全，已经基本可以满足需要

####配置数据库
- 我目前在学习使用mongodb（然而SQL语言并没有学好||-_-),因为看到大部分MEAN开发都使用noSQL语言
- 安装比较简单，14.04自带源，`sudo apt-get install mongodb`即可，找不到命令的话需要`sudo apt-get update`一下
- 配置启动mongo和指定数据库目录用`mongod --dbpath “目录”`
- [这篇文章](http://www.bitscn.com/pdb/otherdb/201501/442949.html)可以稍微参考一下，mongodb安装不会出什么大问题

>到这里ubuntu下nodejs工作流基本配置完了，只是我在github上的博客还存在windows下，我想把博客的写作环境移植到这里，同时以后换电脑换系统时也需要有个参考。怎么办呢
- 找到了一个比较靠谱的@[曾梦想仗剑走天涯](http://kwangka.github.io/2015/01/17/how-to-synchronize-blog/)写的一篇文章，参考这个基本就能在任意系统上进行博客文章的写作了
- 图床我扔给七牛了，这个不用考虑
- 当然方法不唯一，你也可以参考别的方法达成这个目的。有人介绍可以通过dropbox同步更方便，有时间可以尝试一下

#### 还有一些乱七八糟的东西
- 护眼，没有防蓝光眼镜的话（有的话尽量也搞一个）装一个`f.lux`，不过这里用的是`redshift`，据说这个要好一点。参考[图灵社区](http://www.ituring.com.cn/article/211486)的一篇文章进行安装
- 截图，键盘上的Print Screen就很方便，会弹出对话框提醒你保存。放一张网易云音乐的照片
![cloudNet](http://7xswbj.com1.z0.glb.clouddn.com/cloudNet.png)
- 邮件系统，还是用Thunderbird，好用原生而且方便。系统自带不用安装，配置自己找教程（偷个懒）
- dropbox，这个没装好，有时间再来更新

### 收工！接下来就是愉快的学习和开发过程了～
----
####更新
>
2016.9.4	升级到ubuntu16.04LTS，安装搜狗中文输入可直接使用gdebi安装
2016.9.10	更换Remarkable作为MD编辑器
2016.9.26	转移工作流至阿里云服务器
2016.9.30	配置工作流到docker