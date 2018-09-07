---
title: JavaScript 事件绑定机制
date: 2016-05-08 17:53:54
updated:
tags: 前端
---

本文目的：
- 理解并能按需使用各种事件绑定 API
- 理解事件对象
- 理解事件流

# 事件绑定

事件是用户或浏览器自身执行的某种动作，例如 `onclick` 、 `onload` ，都是事件的名字。而响应某个事件的函数就叫做 **事件处理程序（Event Handlers）**。为事件绑定处理程序的方式有以下几种：

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

优点：简单、粗暴，浏览器兼容性好。

缺点：

- 存在时差问题。用户可能会在 HTML 元素一出现在页面上时，就触发相应事件，但当时的事件处理程序有可能还未具备执行条件（例如事件处理程序所在的外部脚本文件还未加载或解析完毕），此时会引发 `undefined` 错误。


- HTML 与 JavaScript 代码紧密耦合。如果要重命名事件处理程序，就要改动两个地方，容易改漏、改错。

## DOM Level 0

做法：首先获取目标 HTML 元素的引用，然后将一个事件处理程序赋值给其指定的事件属性：

```html
<input type="button" id='btn' value="Click Me" />

<script type="text/javascript">
    var btn = document.getElementById('btn');

    // 绑定事件处理程序
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

- 传统、常用、浏览器兼容性好。
- 解决了 HTML 事件处理程序的两个缺点。

缺点：一个事件只能绑定唯一一个事件处理程序。

## DOM Level 2

做法：目前最主流的写法，可以支持事件冒泡或捕获：

```javascript
<input type="button" id='btn' value="Click Me" />

<script type="text/javascript">
    var btn = document.getElementById('btn'),
        showMessage = function() {
            alert('Clicked');
            alert(this.id);    // 输出“myDiv” —— this 指向事件的目标元素
        };

    // 绑定事件处理程序。false 表示在“冒泡阶段”和“目标阶段”触发
    btn.addEventListener('click', showMessage, false);

    // 删除事件处理程序。注意，匿名函数无法移除
    btn.removeEventListener('click', showMessage, false);
</script>
```

优点：一个事件可以绑定多个事件处理程序，以绑定的顺序执行。

缺点：浏览器兼容性差，IE8 及以下版本不支持。

## IE

IE 实现了与 DOM 2 级类似的两个方法，只支持事件冒泡：

```javascript
<input type="button" id='btn' value="Click Me" />

<script type="text/javascript">
    var btn = document.getElementById('btn'),
        showMessage = function() {
            alert('Clicked');
            alert(this === window);    // 输出“true” —— 注意 this 指向 window
        };

    // 绑定事件处理程序。仅在“冒泡阶段”和“目标阶段”触发
    btn.attachEvent('onclick', showMessage);

    // 删除事件处理程序。注意，匿名函数无法移除
    btn.detachEvent('onclick', showMessage);
</script>
```

优点：一个事件可以绑定多个事件处理程序，以绑定的顺序 **逆序** 执行。

缺点：浏览器兼容性差，仅支持 IE 及 Opera。

## Cross-Browser

鉴于上述几种方式的各有优劣，为了以跨浏览器的方式处理事件，可以定义自己的  `EventUtil`：

```javascript
var EventUtil = {

    addHandler: function(element, type, handler){
        if (element.addEventListener){
            element.addEventListener(type, handler, false);
        } else if (element.attachEvent){
            element.attachEvent(“on” + type, handler);
        } else {
            element[“on” + type] = handler;
        }
    },

    removeHandler: function(element, type, handler){
        if (element.removeEventListener){
            element.removeEventListener(type, handler, false);
        } else if (element.detachEvent){
            element.detachEvent(“on” + type, handler);
        } else {
            element[“on” + type] = null;
        }
    }

};
```

# 事件对象

在触发 DOM 上的某个事件时，会产生一个事件对象 `event` ，这个对象中包含着所有与事件有关的信息。尽管触发的事件类型不同，可用属性和方法也会不同，但是所有事件都会包含下列常用成员：

| DOM Level 2         | Type     | IE             | Type    | Description                              |
| ------------------- | -------- | -------------- | ------- | ---------------------------------------- |
| `type`              | String   | `type`         | String  | 被触发的事件类型                                 |
| `eventPhase`        | Integer  | -              | -       | 调用事件处理程序的所处阶段：`1` 表示捕获阶段，`2` 表示“处于目标”，`3` 表示冒泡阶段 |
| `target`            | Element  | `srcElement`   | Element | 事件的目标元素                                  |
| `currentTarget`     | Element  | -              | -       | 当前正在处理事件的元素。如果事件处于目标元素，则 `this === currentTarget === target` |
| `stopPropagation()` | Function | `cancelBubble` | Boolean | 取消事件的进一步捕获或冒泡                            |
| `preventDefault()`  | Function | `returnValue`  | Boolean | 取消事件的默认行为                                |

# 事件流

最后总结下与事件处理程序息息相关的“事件流”。事件流是指从页面中接收事件的顺序。但有意思的是，历史上 IE 和 Netscape 开发团队居然提出了 **完全相反** 的事件流概念 —— IE 使用“**事件冒泡（Event Bubbling）**”、Netscape 使用“**事件捕获（Event Capturing）**”。下图演示了这两种事件流的区别：

![事件流（Event Flow）](/img/javascript/event-flow.png)

下表列出了四种事件绑定所使用的事件流模型：

|             | 事件冒泡 or 事件捕获？      |
| ----------- | ------------------ |
| HTML        | 取决于 IE or Netscape |
| DOM Level 0 | 取决于 IE or Netscape |
| DOM Level 2 | 事件冒泡 + 事件捕获        |
| IE          | 事件冒泡               |

下面重点讲解 DOM Level 2 事件处理程序所规定的事件流，其共包含三个阶段（其运行效果如上图从 1 到 10）：

1. 事件捕获阶段，可用于事件截获
2. 处于目标阶段
3. 事件冒泡阶段，可用于[事件委托（Event Delegation）](/2016/05/22/javascript-event-delegation)

下面这段代码演示了 DOM Level 2 的整个事件流：

```html
<!DOCTYPE html>
<html>
<body>
<input type="button" id='btn' value="Click Me" />

<script type="text/javascript">

    // 仅在“事件捕获阶段”和“处于目标阶段”触发
    document.body.addEventListener('click', function(event){
        alert(event.eventPhase + ' body');
    }, true);
  
    // 仅在“事件冒泡阶段”和“处于目标阶段”触发
    document.getElementById('btn').addEventListener('click', function(event){
        alert(event.eventPhase + ' input');
    }, false);

    // 仅在“事件冒泡阶段”和“处于目标阶段”触发
    document.addEventListener('click', function(event){
        alert(event.eventPhase + ' document');
    }, false);

</script>
</body>
</html>
```

点击 `input` 按钮，将依次输出：

```
1 body
2 input
3 document
```

可见，DOM Level 2 是同时支持事件冒泡 + 事件捕获的。

# 参考

- 《JavaScript 高级程序设计》