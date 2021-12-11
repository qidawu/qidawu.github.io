---
title: Hexo 不用图床解决图片资源问题
date: 2019-07-06 21:56:00
updated:
tags: 建站
typora-root-url: ..
---

写博客难免会引用图片资源，这里提供一种不用图床解决图片资源上传的思路：将图片资源作为源文件一并上传仓库。

# 新建 img 目录

首先，在 hexo 博客 source 目录下新建 img 目录，即：`hexo/source/img`

然后，在文章的图片引用处使用该路径即可，例如：`![example](/img/example.png)`

最后，`hexo g` 构建出 `./public` 目录，发现 `img` 在该目录之中。`hexo s` 启动服务后，确认能够成功引用图片。

# 解决 Typora 实时预览

通过上述方法能够解决部署后图片引用问题，但带来一个新的问题就是 Typora 无法实时预览。解决办法：在文章顶部加上 `typora-root-url: ..`

![hexo_with_img](/img/hexo/hexo_with_img.png)

可以将该路径加入到 hexo 模板之中，这样每次 `hexo n` 新建文稿都会带上该配置：

![hexo_with_img_2](/img/hexo/hexo_with_img_2.png)

配置如下：

```
---
title: {{ title }}
date: {{ date }}
updated: {{ updated }}
tags:
typora-root-url: ..
---
```

