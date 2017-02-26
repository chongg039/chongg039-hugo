+++
date = "2016-09-18T09:37:45-07:00"
draft = false
title = "使用ngrok实现内网穿透"
Tags = ["ngrok","server"]
+++
今天想用nodejs写一个微信公众号服务，很尴尬的发现自己的腾讯云学生服务器已经三个月没续费被收回了。工作室的这台服务器又不是很想动，于是就想到前一段时间看的一个实现内网穿透的工具ngrok，花了一段时间部署在了自己的ubuntu上。又化了两块钱买了个域名，准备以后作为工作室测试来用。

这里感谢下[Jerry Qu](https://imququ.com/post/self-hosted-ngrokd.html) 提供的文章，方法很不错，省去了自己爬坑的麻烦。

## 准备阶段

#### 下载ngrok
由于ngrok是用Go语言开发的，先得安装好必要的环境：

```
sudo apt-get install build-essential golang mercurial git
```

接下来从源码安装，原来的官网好像早就挂了，不过开发者们已经把ngrok开源了，可以从github上获得：

```
git clone https://github.com/tutumcloud/ngrok.git ngrok
cd ngrok
```

#### 替换证书
源码中有证书，但需要自己生成一个，并把原来那个替换掉：

```
NGROK_DOMAIN="cmweb.top"
//注意上面的域名地址替换成你自己的
//下面依次运行
openssl genrsa -out base.key 2048
openssl req -new -x509 -nodes -key base.key -days 10000 -subj "/CN=>$NGROK_DOMAIN" -out base.pem
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr
openssl x509 -req -in server.csr -CA base.pem -CAkey base.key -CAcreateserial -days 10000 -out server.crt

cp base.pem assets/client/tls/ngrokroot.crt

```
#### 编译ngrok

```
sudo make release-server release-client
```

成功的标志是ngrok/bin目录下应有ngrok、ngrokd两个文件。

### 开始部署

#### 服务器端
就是前面生成的ngrokd程序，指定证书、域名和端口启动：

```
sudo ./bin/ngrokd -tlsKey=server.key -tlsCrt=server.crt -domain="cmweb.top" -httpAddr=":8081" -httpsAddr=":8082"
```

注意修改为自己的域名，httpAddr、httpsAddr 分别是 ngrok 用来转发 http、https 服务的端口，可以随意指定(但要注意这个域名一定是公网可访问到的)。

这里将我的`cmweb.top`域名做了泛解析，我指定了一个子域 http://pub.cmweb.top:8081 并访问它，发现页面显示：

```
Tunnel pub.cmweb.top:8081 not found
```

说明服务器端已经好了，剩下的就是部署客户端了。

#### 客户端
客户端就是生成的ngrok，这里参考了[Jerry Qu](https://imququ.com/post/self-hosted-ngrokd.html)的方法，在ngrok/bin下写了一个启动配置文件 ngrok.cfg：

```
server_addr: cmweb.top:4443
trust_host_root_certs: false
```

4443 端口是服务端ngrokd 开的用来跟客户端通讯的端口，并可通过 -tunnelAddr=":xxx" 指定。

### 运行
指定子域、要转发的协议和端口、配置文件，在服务器端运行后，运行客户端：

```
./ngrok -subdomain pub -proto=http -config=ngrok.cfg 80
```

这里貌似只支持本机的80端口进行映射，试了其他的端口会报错，而且nodejs的微信API也只支持80端口，毕竟我们的主要目的不是为了研究这个，具体原因也就没有深究。

现在就大功告成了我把一个测试用例`app.listen(80)`上，发现公网可以通过 http://pub.cmweb.top:8081 进行访问了，也就是说实现了内网到公网的一个映射。

### 结语
文章基本是对[Jerry Qu](https://imququ.com/post/self-hosted-ngrokd.html)方法的一次操作纪录，没什么原创性，但是可以学到很多知识（也省去了私人开发项目购买服务器的钱...）。总之以后本机开发的项目就不用劳心费神的再放到服务器上辛辛苦苦的去调试了。

这也算是一次成功的实践，也不用麻烦的去捣鼓路由器。

补充：发现了一个集成好了的东西：[sunny-ngrok](www.ngrok.cc) ，有时间大家可以研究一下。