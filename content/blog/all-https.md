+++
date = "2017-03-22T22:51:09+08:00"
title = "[Hugo]使用 Cloudflare 为自定义域名的 Github 博客完成全站 HTTPS 化"
draft = false
Tags = ["Blog","HTTPS","Cloudflare"]

+++

跟风，响应 Web 世界的号召，决定把博客全站换成 HTTPS 访问，并记录下遇到的一些问题。

### Github Page

原来博客是部署在云服务器上，后来觉得没有这个必要，也不想花精力在维护上面，就重新把博客放回了 Github Page 。因此下面的文字并不针对自有服务器的博客用户进行说明。

### 图床

为了加快访问速度，原来图片等一些较大的文件都放在了 七牛云存储 上，但七牛并不提供免费的 HTTPS 流量访问，而这却是全站 HTTPS 化的硬性条件之一。

我选择了将图床迁移到了 Flickr 上，它提供了免费的 HTTPS 外链和 1T 的存储量，使用起来绰绰有余。

Mac 端迁移推荐一款应用：iPic，配合插件使用能自动识别 Markdown 文章中的图片并将其上传到新的图床。虽然我使用的时候这个功能好像挂了，但其他功能印象还是很不错。软件一年的使用费用是 38元人民币，你可以在 App Store中下载。

### SSL服务提供商：Cloudflare

作为私人博客还是希望有一个免费的 SSL 证书，最理想的 Let's Encrypt 却不能为我们的 Github 博客提供帮助。然而作为候选的 Cloudflare 则给了我们这个选择，具体方法不再阐述，你可以参考这篇博文完成配置操作：[为自定义域名的GitHub Pages添加SSL 完整方案 ](https://yicodes.com/2016/12/04/free-cloudflare-ssl-for-custom-domain/) 。

### 遇到的问题

1. 全站 HTTPS 化要求所有引用文件的来源，否则会显示网站不安全的信息，这便需要仔细选择图床等第三方文件存储提供商。

2. 我是用的是 Hugo 的 Go 语言博客框架，如果你使用的是第三方的主题文件，一定要检查其中 JS，CSS文件引入时的路径，如果是 "http://.../main.css" 开头的绝对路径是无法被引用使用的。一个可行的方案是修改为 "//.../main.css" 的相对文件路径。

3. 如果使用自定义域名代替 github.io 的二级域名，注意在 Hugo 的配置文件 config.toml 中的 baseurl 修改，否则有时仍会引用 HTTP 的文件资源。

   下面是我的配置文件：

   ```toml
   baseurl             = "https://chongg039.cn/"
   builddrafts         = false
   canonifyurls        = true
   contentdir          = "content"
   languageCode        = "zh-cn"
   layoutdir           = "layouts"
   publishdir          = "public"
   author              = "coldriver"
   title               = "coldriver的技术博客"
   theme				= "cocoa"
   pygmentsuseclasses  = true
   disqusshortname     = "coldriver" # Comment out to disable Disqus.
   pluralizelisttitles = false
   googleAnalytics     = "UA-123-45"

   [permalinks]
   fixed = ":title/"
   blog  = "blog/:slug/"

   [params]
   author                 = "coldriver的技术博客"
   cachebuster            = true                          # add the current unix timestamp in query string for cache busting css assets
   dateform               = "Jan 2, 2006"
   dateformfull           = "Mon Jan 2 2006 15:04:05 MST"
   description            = "Don't panic"
   email                  = "chongg039@gmail.com"
   # extracssfiles          = [ "/css/override.css" ]       # In your `static` directory, add/remove files as necessary.
   # faviconfile            = "img/leaf.ico"
   github                 = "//github.com/chongg039"
   highlightjs            = true
   lang                   = "en"
   # linkedin               = "//linkedin.com/in/you"
   noshowreadtime         = false                         # if true, don't show "<x> minutes read" in posts
   selfintro              = ""                            # appears in the site header when set to a non-empty string
   twitter                = "//twitter.com/uestcchongg"
   # highlightjslanguages = ["go"]                        # additional languages not included in the "common" set

   # avatar                 = "img/profile.png" # path to image in static dir e.g img/avatar.png (do not use in the same time as gravatar)
   # gravatar             = ""                # do not use in the same time as avatar

   # The following are DEPRECATED.
   gatracker              = "XYZ" # use googleAnalytics instead
   initials               = "ad"  # displayed on single post page; deprecated in v0.3.0.
   ```

4. Cloudflare 生效大概在72小时之内，我这边还是比较快的。如果你的博客显示不安全的信息，或者丢失样式文件，建议打开控制台检查一下，多半是某处文件没有使用 HTTPS 导致的。

最后建议阅读 Jerry Qu 的[关于启用 HTTPS 的一些经验分享](https://imququ.com/post/sth-about-switch-to-https.html)，对理解不同平台以及浏览器的 HTTPS 启用有很大帮助。

配置的过程很痛苦，但当你的博客上出现一把小绿锁，心里是真的舒服。

