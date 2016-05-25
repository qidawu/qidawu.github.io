---
title: Javascript 事件委托机制
date: 2016-05-22 19:05:06
updated:
tags: JavaScript
---

本文目的：

- 能够理解事件绑定和事件委托两种机制的区别
- 能够使用原生 API 和 jQuery API 两种方式进行事件委托

# Native API

项目开发时遇到一个需求：修改页面中所有 `A` 链接的默认行为。

## 事件绑定

最开始想到了用“事件绑定”机制进行实现：

```javascript
var showMessage = function(event) {
        // 阻止 A 链接的默认行为（不进行跳转）
        event.preventDefault();
        // 仅弹窗显示链接的 href 属性
        alert(event.currentTarget.href);
    };

var links = document.getElementsByTagName('a');
for (var i = 0; i < links.length; i++) {
    links[i].addEventListener('click', showMessage, false);
}
```

这种做法的问题是：如果页面中绑定了大量的事件处理程序，将直接影响页面的整体运行性能，因为：

1. 函数即对象，对象越多，越占用内存，性能就越差。
2. 事件绑定前，必须先找到指定的 DOM 元素。而 DOM 元素查找次数越多，页面的交互就绪时间就越长。

更麻烦的是，如果页面加载完后再次插入新元素，需要再次绑定事件处理程序，灵活性差：

```javascript
var newLink = document.createElement("a");
newLink.innerHTML = 'Click Me';
newLink.href = 'http://localhost';
newLink.addEventListener('click', showMessage, false);
document.body.appendChild(newLink);
```

## 事件委托

利用事件委托机制可以同时解决上述两个问题。只需在 DOM 树中尽量最高的层次上添加一个事件处理程序，代码如下：

```javascript
var showMessage = function(event, target) {
        // 阻止 A 链接的默认行为（不进行跳转）
        event.preventDefault();
        // 仅弹窗显示链接的 href 属性
        alert(target.href);
    },
    // 递归查询指定父元素
    findTarget = function(target, tagName) {
        while (target.tagName && target.tagName !== tagName.toUpperCase()) {
            target = target.parentNode;
        }
        return (target.tagName && target.tagName === tagName.toUpperCase()) ? target : null;
    };

document.body.addEventListener('click', function(event) {
    // 间接判断 A 链接是否被点击
    var target = findTarget(event.target, 'a');
    if (target) {
        showMessage(event, target);
    }
}, false);
```

由于所有 `A` 链接都是 `body` 元素的子节点，并且它们的事件都会冒泡，因此点击事件最终会被 `body` 上添加的事件处理程序所处理。代码重构后在以下方面提升了页面性能：

- 由于 `document` 对象很快就可以访问，而且可以在页面生命周期的任何时点上为它添加事件处理程序（无需等待 `DOMContentLoaded` 或 `load` 事件），因此只要可点击的元素呈现在页面上，就可以立即具备适当的功能。
- 由于只添加一个事件处理程序，因此所需的 DOM 引用更少，整个页面占用的内存空间也更少。

此外，事件会关联到当前以及以后添加的子元素上面，可以避免反复为新元素绑定事件处理程序，可谓一劳永逸。

# jQuery API

理解了两种机制的区别后，看看如何使用 jQuery 进行最快的实现：

## on()

jQuery 1.7+ 推出了 `on()` 方法，其目的有两个：

1. 统一接口

2. 提高性能

用法如下：

* 事件绑定：`on(events,[data],fn)` ，用于替换 `bind()`
* 事件委托：`on(events,[selector],[data],fn)`  ，用于替换 `live()` 、 `delegate()` 。这里的 `[selector]` 参数很关键，起到了一个过滤器的效果，只有被选中元素的 **子元素** 才会触发事件。

用 `on()` 方法重构后的代码如下：

```javascript
$('body').on('click', 'a', function(event) {
    event.preventDefault();
    alert(event.currentTarget.href);
});
```

可见，代码重构后非常简洁，推荐使用。

# 参考

* 《[关于jQuery新的事件绑定机制on()的使用技巧](http://www.jb51.net/article/36064.htm)》
* 《[实例分析JavaScript中的事件委托和事件绑定](http://www.diguage.com/archives/71.html)》