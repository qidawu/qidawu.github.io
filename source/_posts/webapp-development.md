---
title: 移动 Web 开发要点总结
date: 2017-01-14 17:47:15
updated:
tags: 前端
---

![移动 Web 开发要点总结](/img/javascript/key_of_webapp_dev.png)

总结下其中几个要点：

# iOS 300ms 点击延时问题

为什么存在这个问题？

> 这要追溯至 2007 年初。苹果公司在发布首款 iPhone 前夕，遇到一个问题 —— 当时的网站都是为大屏幕设备所设计的。于是苹果的工程师们做了一些约定，应对 iPhone 这种小屏幕浏览 PC 端站点的问题。这当中最出名的，当属双击缩放(double tap to zoom)。
>
> 双击缩放，顾名思义，即用手指在屏幕上快速点击两次，iOS 自带的 Safari 浏览器会将网页缩放至原始比例。
>
> 那么问题来了，假设用户在 iOS Safari 里边点击了一个链接，当用户一次点击屏幕之后，浏览器并不能立刻判断用户是确实要打开这个链接，还是想要进行双击操作。
>
> 因此，iOS Safari 就等待 300 毫秒，以判断用户是否再次点击了屏幕。如果没有，就触发 click 点击事件。

According to Google:

> ... mobile browsers will wait approximately 300ms from the time that you tap the button to fire the click event. The reason for this is that the browser is waiting to see if you are actually performing a double tap.

## 验证问题

```html
<html>
    <body>
        <button>Click Me</button>
        <script type="text/javascript">
            var button = document.querySelector("button"),
                record;

            button.addEventListener("touchstart", function(){
                record = Date.now();
                console.log("button touchstart");
            });
            button.addEventListener("touchend", function(){
                var delay = Date.now() - record;
                console.log("button touchend delay: " + delay + "ms");
            });
            button.addEventListener("click", function(){
                var delay = Date.now() - record;
                console.log("button click delay: " + delay + "ms");
            });
            document.addEventListener("click", function(){
                var delay = Date.now() - record;
                console.log("document click delay: " + delay + "ms");
            });
        </script>
    </body>
</html>
```

iOS Safari 上点击后，显示结果如下：

```
button touchstart
button touchend delay: 59ms
button click delay: 310ms
document click delay: 312ms
```

可见，`click` 事件延迟了 300ms 左右。

## 找出原因

当一个用户点击屏幕的时候，会产生两个事件：`touch` 和 `click` 。`touch` 事件会首先触发，完成捕获、冒泡的事件流。同时在点击的 300ms 延时后，触发 `click` 事件。

## 解决方案

如今移动端 webapp 性能都追求与原生应用匹配，上述 iOS 单击事件 300ms 延迟，显然是不可接受的。有三个解决方案：

### 方案一

只使用 `touch` 事件，然后使用 `e.preventDefault()` 来阻止默认行为 `click`：

```html
<html>
    <body>
        <button>Click Me</button>
        <script type="text/javascript">
            var button = document.querySelector("button"),
                record;

            button.addEventListener("touchstart", function(e){
                record = Date.now();
                console.log("button touchstart");
                e.preventDefault();
            });
            button.addEventListener("touchend", function(){
                var delay = Date.now() - record;
                console.log("button touchend delay: " + delay + "ms");
            });
            button.addEventListener("click", function(){
                var delay = Date.now() - record;
                console.log("button click delay: " + delay + "ms");
            });
            document.addEventListener("click", function(){
                var delay = Date.now() - record;
                console.log("document click delay: " + delay + "ms");
            });
        </script>
    </body>
</html>
```

效果如下：

```
button touchstart
button touchend delay: 60ms
```

这种方案看起来简单易行，然而功能复杂的时候容易出问题。比如滑动加选择，会因为滑动触发 `touchend`，从而触发选择行为。所以如果本该绑定在 `click` 上的事件全部绑定到 `touchend` 事件上，就会出现问题。请看下例：

```html
<html>
    <body>
        <button>Click Me</button>
        <script type="text/javascript">
            var button = document.querySelector("button"),
                record;

            button.addEventListener("touchstart", function(){
                record = Date.now();
                console.log("button touchstart");
            });
            document.addEventListener("touchmove", function(){
                console.log("document move");
            });
            button.addEventListener("touchend", function(){
                var delay = Date.now() - record;
                console.log("button touchend delay: " + delay + "ms");
            });
            button.addEventListener("click", function(){
                var delay = Date.now() - record;
                console.log("button click delay: " + delay + "ms");
            });
            document.addEventListener("click", function(){
                var delay = Date.now() - record;
                console.log("document click delay: " + delay + "ms");
            });
        </script>
    </body>
</html>
```

当点击按钮并拖动时，显示结果如下：

```
button touchstart
document move
document move
document move
document move
document move
button touchend delay: 943ms
```

可见：

* 该例中用户可能只想拖动页面，但却被迫触发了 `touchend` 事件。所以如果本该绑定在 `click` 上的事件全部绑定到 `touchend` 事件上，就会出现问题，违背用户意图。

* 拖动行为会导致 `click` 事件不会执行，可以理解为  `touchmove` 和 `click` 是相斥的。

因此，建议用回 `click`。

### 方案二

使用 zepto.js 的 `tap` 事件，底层是 `click` 事件并去掉 300ms 延时，然而会有点击穿透的问题。

### 方案三

使用 [fastclick](https://github.com/ftlabs/fastclick)，兼容性好，用法简单，没有点透问题，如下：

```html
<html>
    <body>
        <button>Click Me</button>
        <script type="text/javascript" src="fastclick.js"></script>
        <script type="text/javascript">
            var button = document.querySelector("button"),
                record;

            button.addEventListener("touchstart", function(){
                record = Date.now();
                console.log("button touchstart");
            });
            button.addEventListener("touchend", function(){
                var delay = Date.now() - record;
                console.log("button touchend delay: " + delay + "ms");
            });
            button.addEventListener("click", function(){
                var delay = Date.now() - record;
                console.log("button click delay: " + delay + "ms");
            });
            document.addEventListener("click", function(){
                var delay = Date.now() - record;
                console.log("document click delay: " + delay + "ms");
            });
            document.addEventListener('DOMContentLoaded', function() {
                FastClick.attach(document.body);
                }, false);
        </script>
    </body>
</html>
```

效果如下：

```
button touchstart
button touchend delay: 60ms
button click delay: 60ms
document click delay: 60ms
```

## 参考

http://developer.telerik.com/featured/300-ms-click-delay-ios-8/

https://www.sitepoint.com/5-ways-prevent-300ms-click-delay-mobile-devices/

http://www.linovo.me/front/webapp-300ms.html

https://github.com/ftlabs/fastclick

https://github.com/filamentgroup/tappy/

http://labs.ft.com/2011/08/fastclick-native-like-tapping-for-touch-apps/

http://www.mamicode.com/info-detail-666685.html

https://www.jianshu.com/p/dc3bceb10dbb