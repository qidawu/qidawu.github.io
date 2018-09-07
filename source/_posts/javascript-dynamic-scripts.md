---
title: Javascript 动态加载脚本
date: 2016-04-30 21:10:01
updated:
tags: 前端
---

本文演示如何动态加载脚本。即脚本在页面加载时不存在，但将来的某一时刻通过修改 DOM 动态添加脚本，从而实现按需加载脚本。

# 加载脚本文件

```javascript
function loadScriptFile(url) {
    var script = document.createElement('script');
    script.type = 'text/javascript';
    script.src = url;

    // 在执行到这行代码将 <script> 元素添加到页面之前，不会下载指定外部文件
    document.body.appendChild(script);
}
```

# 内联脚本代码

```javascript
function loadScriptString(code) {
    var script = document.createElement('script');
    script.type = 'text/javascript';
    script.text = code;

    // 在执行到这行代码将 <script> 元素添加到页面之前，不会下载指定外部文件
    document.body.appendChild(script);
}
```

以这种方式加载的代码会在全局作用域中执行，而且当脚本执行后将立即可用。实际上，这样执行代码与在全局作用域中把相同的字符串传递给 `eval()` 是一样的。