+++
date = "2016-12-08T12:09:31-07:00"
draft = false
title = "使用 nodejs 模拟登陆电子科大信息门户并抓取信息"
Tags = ["nodejs","crawler"]
+++
学校高层做了个公众号，把信息门户弄到微信上的浏览器上。本着干掉学校高层的一贯行事风格，觉得是时候搞一个正规的公众号来抢学校生意了。

结果没想到并不是很顺利，信息门户登录有两个重定向页面，自己也对模拟请求服务器不是很熟悉，一步一步好歹是扒到了想要的数据，也浪费了好长时间。先拿出来了个命令行版本，还停留在 es5，准备熬过去考试月后全部重构成 es6，再加上流程控制，放到公众号上造福社会。

使用到的 npm 库：

- request：模拟 HTTP 请求
- cheerio：解析页面
- readline-sync：控制命令行会话
- chalk：美化输出
- log4js：管理日志

难度主要是模拟登陆到所需页面和解析相关内容上，封装好了接口，以后就方便使用了。

### 分析报文

为了完成模拟登陆这一目的，先尝试获取登录主界面`http://portal.uestc.edu.cn/`的内容。分析一波登陆过程中的 HTTP 行为：

![HTTPAnalysis](http://ohroncry6.bkt.clouddn.com/HTTPAnalysis.png)

可以发现在正文`portal.uestc.edu.cn/`前有两个页面，分别打开看一看：

`login?service=http%3A%2F%2Fportal.uestc.edu.cn%2F`页面：

![](http://ohroncry6.bkt.clouddn.com/3021.png)

`?ticket=ST-199833-BbDvk6q57cmNFtJkBJNL1481032281383-zATy-cas`页面：

![](http://ohroncry6.bkt.clouddn.com/3022.png)

最终页面：

![](http://ohroncry6.bkt.clouddn.com/200.png)

理论上是获取到最终页面的`JSESSIONID`，这是服务器用来唯一标识用户信息的`session`。带着这个 cookie 去访问其他相关页面，就能获取到对应数据。

那么怎么解决重定向问题呢？

查到 request 方法自身是支持跟随10次页面重定向的，但是我们现在需要禁止重定向，单独访问并获取每个302界面内 header 中的数据。就可以修改 request 的默认 Redirect 方法，禁止重定向跳转或者设置最大跳转次数：

```javascript
var request = request.defaults({followRedirect: false});
//OR ‘maxRedirects = 1’
```

有了条件我们就可以来分析怎么一步步获取我们想要的最终数据了（是的我用了分别 request 每个页面这样一个最蠢的办法 ...），以第一个302页面为例详细说明如何操作。

#### 第一个302页面

第一个页面是一个重定向页面，发送 POST 报文获取到了`Response-Headers`里的`Set-Cookie`:

- CASPRIVACY
- iPlanetDirectoryPro
- CASTGC

这里面有些是我们需要的，获取他们就要分析`Request-Headers`报文。请求头报文里的 Cookie 有两个值：

- route
- JSESSIONID_ids1

这个是怎么生成的？

分析发现这个`JSESSIONID_ids1`是浏览器在访问到 UESTC 信息门户登录界面
`http://idas.uestc.edu.cn/authserver/login?service=http://portal.uestc.edu.cn/index.portal`
时服务器返回的表示用户信息的一个 Cookie 。那么就很简单了，由于访问登录界面使用的是 GET 方法，使用`request`即可获取：

```javascript
var request = require('request');
var setCookies， Cookies;

const loginUrl = 'http://idas.uestc.edu.cn/authserver/login?service=http://eams.uestc.edu.cn/eams/home.action';

request(loginUrl, function (err, response, body) {
	if (!err && response.statusCode == 200) {
      	setCookies = response.headers['set-cookie'];
		Cookies = setCookies[0] + "; " + setCookies[1];
      	return Cookies;
	} else { 
		console.log("暂时无法访问信息门户，请稍后重试");
		return err;
	}
});
```

这样便得到了`Cookies = route + JSESSIONID_ids1`。

下一步要获取 form 中的内容，最为 POST 的表单随同 cookie 一起发出。表单中有：

- username
- password
- lt
- dllt
- execution
- _eventId
- rmShown

前两个是用户名和密码，那后面的又是什么？

听说15年学校系统进行了一次改版，在学号和密码之外又添加了验证码（如果你多输错几次密码应该就能看到）。网上对这种验证码的处理方式有很多，不过我觉得对学校这种系统还用不上一些图像识别的办法。

事实上也确实如此，直接用`cheerio`解析我们的`loginUrl`界面，获取后填入就行：

```javascript
var request = require('request');
var cheerio = require('cheerio');
var setCookies, Cookies, lt, dllt, execution, _eventId, rmShown;

const loginUrl = 'http://idas.uestc.edu.cn/authserver/login?service=http://eams.uestc.edu.cn/eams/home.action';

request(loginUrl, function (err, response, body) {
	if (!err && response.statusCode == 200) {  
	  	var $ = cheerio.load(body);
		lt = $('[name=lt]').attr('value');
		dllt = $('[name=dllt]').attr('value');
		execution = $('[name=execution]').attr('value');
		_eventId = $('[name=_eventId]').attr('value');
		rmShown = $('[name=rmShown]').attr('value');
		// cookies	  
	  	setCookies = response.headers['set-cookie'];
		Cookies = setCookies[0] + "; " + setCookies[1];
	} else { 
		console.log("暂时无法访问信息门户，请稍后重试");
		// return err;
	}
});
```

现在需要的都有了，就可以模拟第一次的 POST 请求了：

```javascript
/* 只写出部分功能 */
var loginOption = {
	url: loginUrl,
	method: 'POST',
    headers: {	
        'Accept':'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
        'Accept-Encoding':'gzip, deflate, sdch',
        'Accept-Language':'zh-CN,zh;q=0.8',
        'Cache-Control':'no-cache',
        'Connection':'keep-alive',
        'Cookie':Cookies,
        'Host':'idas.uestc.edu.cn',
        'Origin':'http://idas.uestc.edu.cn',
        'Pragma':'no-cache',
        'Referer':'http://idas.uestc.edu.cn/authserver/login?service=http://eams.uestc.edu.cn/eams/home.action',
        'Upgrade-Insecure-Requests':1,
        'User-Agent':'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.98 Safari/537.36'
    },
	form: {
		username: userName,
		password: password, // 这里可以先替换个人学号密码使用
		lt: lt,
		dllt: dllt,
		execution: execution,
		_eventId: _eventId,
		rmShown: rmShown
	}
};

request(loginOption, function (err, res, body) {
    if (!err && res.statusCode == 302) {
        // 获取第二个302地址
        redirectLocation = res.headers['location'];
        // 获取所需要的 cookie
        redirectCookies = res.headers['set-cookie'][1]; // iPlanetDirectoryPro
    } else {
        console.log("登录失败！请检查学号和密码重试");
        //return err;
  	}
});
```

打印结果看看应该就可以得到所需要的 Cookie，第二张图的`Reauest-Headers`中需要的是`iPlanetDirectoryPro`，我们也只获取这个就可以了。

通过以上步骤就可以获取到想要的第一个302页面的信息，带着这个信息访问第二个302页面也可以获取到相关信息。最后获取到最终界面的内容。

下面不贴出具体代码，只是写一写思想。

#### 第二个302页面

获得了第一个302页面的`iPlanetDirectoryPro`和重定向地址`redirectLocation`，就可以进行 GET 请求，获取所需要的 cookie 了：

```
MOD_AUTH_CAS=MOD_AUTH_ST-150807-EmrjontiArvbt7tLFeDp1481041468406-eVTU-cas
```

这里有个别的方法，刚才获取的重定向地址`redirectLocation`中：

```
http://portal.uestc.edu.cn/index.portal?ticket=ST-150807-EmrjontiArvbt7tLFeDp1481041468406-eVTU-cas
```

观察到存在相同部分，因此第二步重定向页面请求可以省略，直接截取、拼接字符串也是可以获得第二步的 cookie，但这实际上是取巧了 。

#### 最终200页面

最终页面的 HTTP 请求是当时困扰我好长时间的，为什么这么说呢？先来看一下它的请求头 cookie 有什么：

- route
- JSESSIONID
- iPlanetDirectoryPro
- MOD_AUTH_CAS

对比一下响应头 cookie：

- JSESSIONID

很显然两个`JSESSIONID`内容不一样的嘛！我要获取响应头的`JSESSIONID`，为什么我请求头也要发送一个名称一样的 cookie，而且在前面的请求响应报文中也没有出现好吗！于是我尝试了这些办法：

- 在登录界面的 body 里查找
- 寻找`JSESSIONID_ids1`和`JSESSIONID`之间的关系
- 模拟服务器端`JSESSIONID`的生成算法（感觉真是没救了...）

终于在某一天躺在椅子上思考人生的时候回想起了我获取登录页面`JSESSIONID_ids1`的过程：

> 我为什么不不带 cookie 请求一下第二个302页面呢？

试试试！果然，在 request 方法禁止重定向的情况下，浏览器虽然打不开这个页面，但是服务器确实返回了`route`和`JSESSIONID`	。带着这些重新请求，就能得到最后的`Response JSESSIONID`和`200-body`内容。

**事后仔细回想这个问题，其实是我陷入了两个思维误区：**

- 一个是我认为只要是名为`JSESSIONID`的 cookie，都应该是带有用户 POST 信息的，与用户数据库产生联系的 cookie 。


- 再一个就是浏览器无法直接打开的302界面，即不能从 chrome 控制台观测到有无 HTTP 报文产生，不代表真的不能单独获取服务器返回的 header 数据。

解决了之后真的是心情愉悦～

### 获取相关内容 

进入信息门户的“课程管理”的地址是
`http://eams.uestc.edu.cn/eams/home!childmenus.action?menu.id=844`
将它的 HTTP 请求替换掉上面的200页面的请求，下一步就可以获取到相应的内容。

这里就以获取课程表为例详细说明。

#### 获取课表

地址是
`http://eams.uestc.edu.cn/eams/courseTableForStd!courseTable.action`
从控制台看出需要一个 POST 请求，发送的表单含有：

- ignoreHead
- setting.kind
- startWeek
- semester.id
- ids

cookie 有：`JSESSIONID, semester.id, iPlanetDirectoryPro` 。可以看出表单中含有的前面都是常量，处理的粗糙一些的话就可以直接输入。只有一个`ids`，应该是学校用来标示学生身份的唯一码（事实上也确实如此），这个怎么需要获取呢？

凭经验来看`ids`肯定是在前文的响应 body 里服务器已经返回的数据。果然，在请求这个地址之前已经请求了一个地址：`http://eams.uestc.edu.cn/eams/courseTableForStd.action?_=1481077999393`

![ids](http://ohroncry6.bkt.clouddn.com/ids.png)

解析这个 Response，就可以得到对应的ids值。

**这里面的`_=1481077999393`可能是个时间戳一类而的东西（猜的），每次访问都是会变的，实际测试的时候应该用什么都可以，都会转到自己所需的内容。**

获取到了对应的 form 内容，POST 之后用正则解析就得到了想要的课程表。但 UESTC 信息门户的课程表是用页面脚本加载到表格里去展示，而我们用模拟请求的方法就比较麻烦了。因此就先统一拿出来所有的课程信息，暂时没有想到很好的解决办法。

获取成绩和期末考试安排都是类似的方法，难度都不高。

### 放一下效果：

主界面：

![](http://ohroncry6.bkt.clouddn.com/main.png)

课程表：

![](http://ohroncry6.bkt.clouddn.com/courses.png)

考试安排：

![](http://ohroncry6.bkt.clouddn.com/eams.png)

成绩垃圾就打上码了：

![](http://ohroncry6.bkt.clouddn.com/gradesAll.png)

虽然效果粗糙了一些，但就结果而言还是好的，达到了目的。

### 几个注意的问题

- 有些地址中的 '/' 、':' 被转义了，使用时注意修改回来
- 有时返回的信息是乱码，开始以为是 Shell 的中文解码问题，最后才发现是请求头里`gzip`压缩问题，去掉就可以了

### 后面的打算

考试月还是比较仓促的，也没有认真把这个东西做好。准备过年腾出时间来重写一遍：

- 全部使用 es6 语法，拿 React 或者 vue 套壳
- 加入 d3 或者 echart 库美化视觉输出
- 再做成公众号，干倒高层

整个工程放在了我的  [Github](https://github.com/chongg039/uestcLogin) 上，写的烂了点，不过踩坑时的思想还是可以借鉴的 ，觉得有用就给个 Star 吧。

最后祝大家考试月顺利～