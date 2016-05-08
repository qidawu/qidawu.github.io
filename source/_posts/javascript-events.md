---
title: JavaScript 事件总结
date: 2016-05-08 17:53:54
updated:
tags: JavaScript
---

本文目的：
- 理解事件流
- 理解并能按需使用各种事件处理程序
- 理解事件对象
- 理解事件委托
- 了解不同的事件类型

# 事件流

事件流描述的是从页面中接收事件的顺序。但有意思的是，IE 和 Netscape 开发团队居然提出了差不多是 **完全相反** 的事件流概念。下面以一个 HTML 页面为例，演示不同浏览器下的事件流：

```html
<!DOCTYPE html>
<html>
<body>
    <div id="myDiv">Click Me</div>
</body>
</html>
```

## 事件冒泡

IE 的事件流叫做“**事件冒泡（Event Bubbling）**”，即事件开始时由最具体的元素（文档中嵌套层次最深的那个节点）先接收，然后逐级向上传播到较为不具体的节点（文档）。

如果点击了 HTML 页面中的 `<div>` 元素， `click` 事件会按照如下顺序传播：



## 事件捕获

Netscape 的事件流叫做“**事件捕获（Event Capturing）**”，即事件开始时由不太具体的节点先接收，而最具体的节点应该最后接收。

如果点击了 HTML 页面中的 `<div>` 元素， `click` 事件会按照如下顺序传播：



# 事件处理程序

事件就是用户或浏览器自身执行的某种动作，例如 `onclick` 、 `onload` ，都是事件的名字。而响应某个事件的函数就叫做 **事件处理程序（Event Handlers）**。为事件指定处理程序的方式有以下几种：

## HTML

做法：在 HTML 元素中直接编写事件处理程序：

```html
<!-- 输出“Clicked” —— 事件处理程序中，可以直接编写 JavaScript 代码 -->
<input type="button" value="Click Me" onclick="alert('Clicked')" />

<!-- 输出“click” —— 事件处理程序中，可以直接访问事件对象 event -->
<input type="button" value="Click Me" onclick="alert('event.type')" />

<!-- 输出“Click Me” —— 事件处理程序中，this 指向事件的目标元素 -->
<input type="button" value="Click Me" onclick="alert('this.value')" />
```

其中除了可以编写 JavaScript 代码，还可以调用外部脚本：

```html
<!-- 事件处理程序中的代码在执行时，有权访问全局作用域中的任何代码 -->
<input type="button" value="Click Me" onclick="showMessage()" />

<script type="text/javascript">
    function showMessage() {
        alert("Hello world!");
    }
</script>
```

特点：上述 `onclick` 事件将自动产生一个事件处理程序（函数），例如：

```javascript
function onclick(event) {
    alert('Clicked')
}
```

优点：简单、粗暴，兼容性好。

缺点：

- 存在时差问题。用户可能会在 HTML 元素一出现在页面上时，就触发相应事件，但当时的事件处理程序有可能还未具备执行条件（例如事件处理程序所在的外部脚本文件还未加载或解析完毕），此时会引发 `undefined` 错误。


- HTML 与 JavaScript 代码紧密耦合。如果要重命名事件处理程序，就要改动两个地方，容易改漏、改错。

## DOM 0级

做法：首先获取目标 HTML 元素的引用，然后将一个事件处理程序赋值给其指定的事件属性：

```html
<input type="button" id='btn' value="Click Me" />

<script type="text/javascript">
    var btn = document.getElementById('myDiv');

    // 添加事件处理程序
    btn.onclick = function() {
        alert('Clicked');
        alert(this.id);    // 输出“myDiv” —— this 指向事件的目标元素
    }

    // 删除事件处理程序
    btn.onclick = null;
</script>
```

特点：本质上，DOM 0级事件处理程序 **等于** HTML 事件处理程序，例如：

```html
<input type="button" id='btn' value="Click Me" onclick="alert('Clicked')" />

<script type="text/javascript">
    setTimeout(function() {
        var btn = document.getElementById('btn');
        alert(typeof btn.onclick);    // 通过 HTML 的事件属性，访问其 HTML 事件处理程序，并输出其类型“function”
        btn.onclick = null;    // 几秒后，将会删除该按钮的事件处理程序
    }, 3000);
</script>
```

优点：

- 传统、常用、兼容性好。
- 解决了 HTML 事件处理程序的两个缺点。

缺点：一个事件只能绑定唯一一个事件处理程序。

## DOM 2级

## IE

## 跨浏览器



# 事件对象

# 事件委托

# 事件类型