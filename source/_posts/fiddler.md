---
title: 使用 Fiddler 代理调试 APP 页面
date: 2017-02-07 22:14:58
updated:
tags: 前端
---

我们知道如何在电脑上通过 Chrome、Firefox 调试页面请求，但在手机端呢？我们可以使用 fiddler 来调试webapp。fiddler 是一个很好的调试、抓包工具。

问题：在客户端打开页面有问题，但浏览器正常；（由于客户端特定的环境，我们无法在电脑端浏览器调试定位一些问题，线上环境，更不能修改代码来调试。怎么办？ 用Fiddler 代理本地文件来调试）

解决办法： 使用Fiddler 代理调试APP页面；

# PC 端设置

## Fiddler 安装

https://www.telerik.com/download/fiddler

## Fiddler 设置

首先打开Fiddler->Tools->Fiddler Options 进行配置，配置完成后重启 Fiddler，如下图：

![Fiddler 设置](/img/fiddler/fiddler_setting.png)

## 关闭防火墙

关闭本机防火墙，避免手机无法 ping 通。

# iPhone 手机端调试

参考：http://blog.csdn.net/asmcvc/article/details/51566569

## 安装 Fiddler 证书

手机浏览器访问 172.22.31.43:8888 ，点击安装证书：

![安装 Fiddler 证书](/img/fiddler/fiddler_certificate_install_1.png)

提示警告，继续安装：

![安装 Fiddler 证书](/img/fiddler/fiddler_certificate_install_2.png)

安装完毕：

![安装 Fiddler 证书](/img/fiddler/fiddler_certificate_install_3.png)

## 证书信任设置

iOS 10 需要设置：通用 - 关于本机 - 证书信任设置 - 针对根证书启用完全信任：

![证书信任设置](/img/fiddler/fiddler_certificate_install_4.png)

## WIFI 代理设置

![WIFI 代理设置](/img/fiddler/fiddler_wifi_setting.png)

# 开始抓包调试

## 嗅探所有请求、响应

打开APP页面，可以嗅探被过滤的请求：

![嗅探器](/img/fiddler/fiddler_inspect.png)

## 解码 URL 请求参数

普通表单的显示如下：

![解码 URL 请求参数](/img/fiddler/fiddler_inspect_webforms.png)

可以使用 Send to TextWizard 进一步解码：

![解码 URL 请求参数](/img/fiddler/fiddler_inspect_webforms_2.png)

## 设置断点，修改请求、响应

Fiddler 最强大的功能莫过于设置断点了，设置好断点后，你可以修改请求头（如 cookie）、请求体、响应头、响应体。

![断点原理](/img/fiddler/break_request.png)

设置断点有两种方法：

第一种：设置全局断点

打开 Fiddler 点击 Rules-> Automatic Breakpoint -> Before Requests，可修改请求；After Responses，可修改响应。

如何消除全局断点呢？ 点击Rules-> Automatic Breakpoint ->Disabled

![设置全局断点](/img/fiddler/break_request_1.png)



第二种：设置指定断点

在命令行中输入命令: bpu www.baidu.com (这种方法只会中断 www.baidu.com)

如何消除命令呢？ 在命令行中输入命令 bpu

 

断点效果如下：

![设置指定断点](/img/fiddler/break_request_2.png)

![设置指定断点](/img/fiddler/break_request_3.png)

## 自动响应

有时候，线上客户端环境下打开的页面有bug无法用浏览器调试，则可以用fiddler代理本地文件来进行调试，非常方便。例如：

![设置自动响应](/img/fiddler/auto_response.png)

我们将线上JS文件代理为本地文件，我们可以修改本地文件，就能用客户端打开看到修改结果，非常方便，当然我们可以同时代理 引入 vconsole  来在手机端打印错误日志。

也可以将一个接口代理下来，然后新建一个json文件，和代理js文件是同样的方法，就可以修改接口的请求了。

PS：代理接口时可能会出现跨域问题，解决方法[点这里](http://www.choujindeputao.com/fiddler-cross-origin/)。

## 显示 IP

如何显示 IP？只需配置一行代码：

```
FiddlerObject.UI.lvSessions.AddBoundColumn("IP", 120, "X-HostIP");
```

![显示 IP](/img/fiddler/show_ip.png)

# 常见问题

1. 多级代理相互影响（数据库连接vpn_ssl、蓝灯、xx-net等）
2. 代理缓存
3. 左下角的“Captuing”仅用于控制电脑端抓包，不影响手机端，可以关掉，避免影响电脑端的正常上网（例如网易云音乐听歌，印象笔记同步）。
4. 开启代理后，响应体不会被 gzip，也没有响应头：Content-Encoding: gzip。可以自行用 Chrome 和 IE 对比测试。