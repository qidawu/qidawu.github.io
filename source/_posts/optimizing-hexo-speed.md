---
title: "优化 Hexo 博客的访问速度"
date: 2015-05-17 19:19:20
updated: 
tags: 建站
description: 之前本博客是挂在 GitHub Pages 空间上的，但由于众所周知的原因访问速度一直很慢，甚至频繁收到无法访问的告警通知，实在是不堪其扰啊。因此决定迁移博客到国内的空间，并且对部分资源进行 CDN 加速。
---


之前本博客是挂在 GitHub Pages 空间上的，但由于众所周知的原因访问速度一直很慢，甚至频繁收到无法访问的告警邮件，实在是不堪其扰啊。因此决定迁移博客到国内的空间，并且对部分资源进行 CDN 加速。

# 迁移 Pages 服务

很多国内的空间都支持部署静态博客，例如 [GitCafe]()、[Coding](https://coding.net)、[SAE](http://sae.sina.com.cn/)、[七牛](http://www.qiniu.com/)、……

在此特别介绍 GitCafe，一个 GitHub 的国内版，但访问速度比 GitHub 快，其 Pages 服务免费且支持绑定自定义域名，就选它了。

如何创建 GitCafe Pages 服务？参考[这里](https://gitcafe.com/GitCafe/Help/wiki/Pages-%E7%9B%B8%E5%85%B3%E5%B8%AE%E5%8A%A9)。

如何将 Hexo 静态博客部署到 GitCafe 仓库？参考[这里](https://github.com/hexojs/hexo-deployer-git#options)，我的配置如下：

```
deploy:
  type: git
  repo:
    github: git@github.com:cynthia903/cynthia903.github.io.git,master
    coding: git@git.coding.net:cynthia903/cynthia903,coding-pages
    gitcafe: git@gitcafe.com:cynthia903/cynthia903.git,gitcafe-pages
```

注意仓库地址须使用 SSH 而不是 HTTP 协议，这样在推送代码时就无须繁琐的输入账号密码了。

运行命令即可生成站点并推送部署：

```
hexo d -g
```

# 使用 CDN 加速

利用 CDN 服务商遍布全国甚至全球的 CDN 缓存节点，可以使得各地网民迅速的访问到网站资源。对于 Hexo 这种静态博客来说，使用 CDN 进行 HTTP 网页加速尤为合适，缓存命中率极高，减轻源站的访问压力，在博客达到一定访问量时可以考虑使用这种方案。

目前来说，只需对部分静态资源进行 CDN 加速即可，使用 CDN 公共库可以满足需求。什么是 CDN 公共库？引述自 [cnbeta](http://www.cnbeta.com/articles/304469.htm)：

> CDN公共库是指将常用的JS库存放在CDN节点，以方便广大开发者直接调用。使用CDN公共库不仅可以为你节省流量，还能通过CDN加速，获得更快的访问速度。

目前国内一些比较大的 CDN 公共库：

* 百度CDN公共库：http://developer.baidu.com/wiki/index.php?title=docs/cplat/libs
* 新浪云计算CDN公共库：http://lib.sinaapp.com
* BootCDN公共库：http://www.bootcdn.cn
* 360公共库：http://libs.useso.com
* 七牛云存储 开放静态文件CDN：http://www.staticfile.org
* 又拍云JS库CDN服务：http://jscdn.upai.com
* CDNJS：http://www.cdnjs.com

由于 BootCDN 公共库的资源较全，在此推荐选用。

以 hexo 主题 jacman 为例，修改文件 `themes\jacman\layout\_partial\after_footer.ejs`，找到以下 JS 和 CSS：

```javascript
<script src="<%- config.root %>js/jquery-2.0.3.min.js"></script>
<script src="<%- config.root %>js/jquery.imagesloaded.min.js"></script>
<script src="<%- config.root %>fancybox/jquery.fancybox.pack.js"></script>
<link rel="stylesheet" href="<%- config.root %>fancybox/jquery.fancybox.css" media="screen" type="text/css">
```

替换为：

```javascript
<script src="http://cdn.bootcss.com/jquery/2.0.3/jquery.min.js"></script>
<script src="http://cdn.bootcss.com/jquery.imagesloaded/2.1.0/jquery.imagesloaded.min.js"></script>
<script src="http://cdn.bootcss.com/fancybox/2.1.5/jquery.fancybox.pack.js"></script>
<link rel="stylesheet" href="http://cdn.bootcss.com/fancybox/2.1.5/jquery.fancybox.css" media="screen" type="text/css">
```

即可生效。