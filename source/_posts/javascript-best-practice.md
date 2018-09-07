---
title: Javascript 编程最佳实践
date: 2016-03-10 22:03:01
updated:
tags: 前端
---

# 前言

本文总结出一些广受认可的编程最佳实践，用于解决特定领域的问题。

# 编程最佳实践

## 避免使用全局变量

在 JavaScript 所有的糟糕特性之中，最为糟糕的一个就是它对全局变量的依赖。JS 大神 Douglas Crockford 甚至称之为“毒瘤”。想象一下，一个全局变量可以被程序的任何部分在任意时间修改，将使得程序的行为变得极度复杂。可怕的全局变量还带来了以下问题：

1. 命名冲突
2. 代码的脆弱性
3. 难以测试

共有三种方式定义全局变量，这些方式都是我们要避免的：

```javascript
var foo = value;       // 1、在任何函数之外放置一个 var 语句
window.foo = value;    // 2、直接给全局对象添加属性
foo = value;           // 3、直接使用未经声明的变量，即隐式的全局变量。一般都是开发者忘记声明，这将导致查找 bug 非常困难
```

下面是一些解决办法：

### 零全局变量

如果你编写的是一段不会被其它脚本访问到的完全独立的脚本，可以使用一个立即执行的匿名函数来[创建私有作用域](/2016/03/09/javascript-style-guideline/#创建私有作用域)。

### 单全局变量

最小化使用全局变量的方法之一是为你的应用创建唯一一个全局变量，并将你所有的功能代码都挂载到这个全局对象上。这种做法既降低了模块之间发生冲突的可能，又能保证模块之间的正常通信。可以参考 [JavaScript 模块模式](http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html)。

目前这种单全局变量模式已经在各种流行的库中广泛使用了：

* jQuery 定义了两个全局对象，`$` 和 `jQuery`。只有在 `$` 被其它库使用了的情况下，为了避免冲突，才使用 `jQuery`。
* YUI 定义了唯一一个 `YUI` 全局对象。
* Dojo 定义了唯一一个 `dojo` 全局对象。
* ……

### 模块化

最后一种、也是最为推崇的做法是使用“模块化”方式组织代码：

* ECMAScript 6 可以使用最新的原生模块标准语法；
* 使用主流的模块框架规范，如 AMD 和 CMD（[AMD 和 CMD 的区别有哪些？](https://www.zhihu.com/question/20351507/answer/14859415)）。

## 不是你的对象不要动

JavaScript 独一无二之处在于任何东西都不是神圣不可侵犯的。默认情况下，你可以修改任何你可以触及的对象。解析器根本就不在乎这些对象是开发者定义的还是默认执行环境的一部分——只要是能访问到的对象都可以修改。如果你的代码没有创建这些对象，禁止修改它们，包括：

* 原生对象（`Object`、`Array` 等等）；
* 文档对象模型（DOM）（`document` 等等）；
* 浏览器对象模型（BOM）（`window` 等等）；
* 类库的对象（`$`、`jQuery` 等等）。

### 原则

#### 不覆盖方法

覆盖方法将会导致所有依赖该方法的代码失效：

```javascript
// 不好的写法 - 覆盖了 DOM 方法
document.getElementById = function() {
    // 任意代码
};
```

#### 不新增方法

新增方法将会导致未来潜在的命名冲突，因为一个对象此刻没有某个方法不代表它未来没有。更糟糕的是如果将来原生的方法和你新增的方法行为不一致，将会陷入一场代码维护的噩梦：

```javascript
// 不好的写法，在 DOM 对象上增加了方法
document.getElementsByClassName = function(classes) {
    // 非原生实现。
    // 该新增方法在 HTML 5 中被官方实现了，这将会导致所有依赖该方法的代码报错。
};
// 不好的写法，在原生对象上增加了方法
Array.prototype.reverseSort = function() {
    return this.sort().reverse();
};
// 不好的写法，在库对象上增加了方法
$.doSomeThing = function() {
    // 任意代码
};
```

#### 不删除方法

删除方法将会导致所有依赖该方法的代码运行时错误。对于已发布的库来说，无用的方法应该被标识位“废弃”而不是直接删掉：

```javascript
// 不好的写法 - 删除了 DOM 方法
document.getElementById = null;
```

### 解决办法

下面介绍一些解决方法：

#### 继承

如果一种类型的对象已经做到了你想要的大多数工作，那么继承它然后再新增一些功能是最好的做法。JavaScript 中有两种基本的继承形式：

* 基于对象的继承
* 基于类型的继承

例如：

```javascript
var MyError = function(message) {
    this.message = message;
};

MyError.prototype = new Error();    // 基于类型的继承，继承自原生的 Error 类
```

#### 门面模式

JavaScript 的继承有一些很大的限制，就是无法继承自 DOM 或 BOM 对象。解决办法是利用门面模式为这些已存在的对象创建一个新的接口，达到二次封装的效果。jQuery 和 YUI 的 DOM 接口都使用了门面模式。例如：

```javascript
// 自定义一个 DOM 对象包装器
var DOMWrapper = function(element) {
    this.element = element;
};

DOMWrapper.prototype = {
    constructor: DOMWrapper,
    addClass: function(className) {
        this.element.className += ' ' + className;
    },
    remove: function() {
        this.element.parentNode.removeChild(this.element);
    }
};

// 用法
var wrapper = new DOMWrapper(document.getElementById("my-div"));
// 添加一个 className
wrapper.addClass("selected");
// 删除元素
wrapper.remove();
```

## 事件处理

### 解耦事件处理

事件处理常见的问题是将事件处理程序和业务逻辑紧紧耦合在一起，降低了代码的可维护性：

```javascript
// 不好的写法
var handleClick = function(event) {
    // DOM Level 2
    event.preventDefault();
    event.stopPropagation();

    // 耦合业务逻辑
    var popup = document.getElementById("popup");
    popup.style.left = event.clientX + "px";
    popup.style.top = event.clientY + "px";
    popup.className = "reveal";
}

document.getElementById('btn-action')
            .addEventListener("click", handleClick, false);    // DOM Level 2
```

正确的做法应该是解耦事件处理程序和业务逻辑，提高代码的可维护性：

```javascript
// 好的写法

    // 事件处理程序，唯一能接触 event 对象的函数
var handleClick = function(event) {
        // DOM Level 2
        event.preventDefault();
        event.stopPropagation();

        showPopup(event.clientX, event.clientY);
    },
    // 抽取业务逻辑，与事件隔离，便于重用与测试
    showPopup = function(x, y) {
        var popup = document.getElementById("popup");
        popup.style.left = x + "px";
        popup.style.top = y + "px";
        popup.className = "reveal";
    }

document.getElementById('btn-action')
            .addEventListener("click", handleClick, false);    // DOM Level 2
```

可见，业务逻辑不应该依赖于 `event` 对象来完成功能，原因如下：

* 好的 API 一定是对于期望和依赖都是透明的，因此方法接口应该表明哪些数据是必要的。将 `event` 对象作为参数并不能告诉你 `event` 的哪些属性是有用的，用来干什么？
* 如果想测试这个方法，你必须构建一个 `event` 对象并作为参数传入。这迫使你关注方法内部实现，以确切地知道这个方法使用了哪些信息，这样才能正确地写出测试代码。

### 使用事件委托

关于“事件绑定（Event Binding）”和“事件委托（Event Delegation）”两种机制的区别在 [本文](/2016/05/22/javascript-event-delegation) 有详细的描述。简而言之，从“内存消耗”、“处理速度”、“新增元素的处理”三方面考虑，都更建议使用“事件委托”。下例演示了如何使用 jQuery 语法进行“事件委托”：

```javascript
$('#list').on('click', 'li', function() {
    //function code here.
});
```

当 `#list` 内任一 `li` 子元素被点击时，`click` 事件将冒泡到其父元素 `#list` 并触发 `#list` 的事件处理程序，即子元素的事件都委托给父元素进行处理。这种做法有利于提升性能，推荐使用。

## UI 层保持松耦合

保持 Web UI 层的松耦合，以便在以下场景中调试代码，定位问题：

* 当发生了文本或结构相关的问题，通过查找 HTML 即可定位；
* 当发生了样式相关的问题，通过查找 CSS 即可定位；
* 当发生了行为和交互相关的问题，通过查找 JavaScript 即可定位。

这种快速定位问题的能力是 Web 界面可维护性的核心关键。

### 将 JavaScript 从 CSS 中抽离

禁止使用 CSS 表达式（CSS Expression）。

```javascript
// 不好的写法
.box {
    width: expression(document.body.offsetWidth + "px");
}
```

CSS 表达式是 IE8 及更早版本中的一个特性，它允许你将 JavaScript 直接插入到 CSS 中，这样可以在 CSS 代码中直接执行运算或其它操作。但 CSS 表达式会带来两个问题：

* 性能问题
* 代码可维护性问题

### 将 CSS 从 JavaScript 中抽离

禁止在 JavaScript 脚本中直接操作 CSS 样式：

```javascript
// 不好的写法
element.style.color = 'red';
element.style.left = '10px';
element.style.cssText = 'color: red; left: 10px';
```

当需要通过 JavaScript 来操作元素样式的时候，最佳方法是操作 CSS 的 `className`：

```javascript
element.className = 'className';    // 原生方法
$(element).addClass('className');   // jQuery
```

CSS 的 `className` 应该成为 CSS 和 JavaScript 之间通信的桥梁。JavaScript 不应当直接操作 CSS 样式，以便保持和 CSS 的松耦合。

### 将 JavaScript 从 HTML 中抽离

禁止在 HTML 标签中嵌入 JavaScript 脚本：

```javascript
<!-- 不好的写法，不该直接为 HTML 标签的 on 属性挂载事件处理程序 -->
<button onclick="doSomeThing()" id="btn-action">Click Me</button>
```

这样会导致 HTML 页面和 JavaScript 脚本紧紧耦合。正确的做法应当是在外部脚本文件中添加事件处理程序：

```javascript
var doSomeThing() {  }

document.getElementById('btn-action')
            .addEventListener("click", doSomeThing, false);    // DOM Level 2
$('#btn-action').click(doSomeThing);    // jQuery
```

这种做法的优势在于，函数 `doSomeThing()` 的定义和事件处理程序的绑定都是在同一个文件中完成的。如果函数名称需要修改，则只需修改一个文件即可；如果点击发生时想额外做一些动作，也只需在一处做修改。

此外，不到迫不得已，不建议在 HTML 页面中嵌入 JavaScript 脚本：

```javascript
<!--  不好的做法 -->
<script>
    doSomeThing();
</script>
```

### 将 HTML 从 JavaScript 中抽离

不建议在 JavaScript 脚本文件中嵌入 HTML 操作：

```javascript
// 不好的做法
var div = document.getElementById('my-div');
div.innerHTML = "<h3>Error</h3><p>Invalid e-mail address.</p>";
```

这样会导致 JavaScript 脚本和 HTML 标签紧紧耦合，从而降低了代码的可维护性，增加了跟踪文本和结构性问题的复杂度。正常来说，调试上述这段标签的典型方法，应当是先去浏览器调试工具中的 DOM 树中查找，然后打开页面的 HTML 源码对比其不同。一旦 JavaScript 脚本文件中做了除简单 DOM 操作之外的事情，如渲染标签，追踪 Bug 就变得很麻烦。因为脚本和标签都耦合成一坨了，让人望而却步。

HTML 文本和标签应该只存放于一个地方：可以控制你 HTML 代码的地方。最为推崇的做法是利用 **JavaScript 模板引擎** 解决这个问题。

项目中我引入了模板引擎 [artTemplate](https://github.com/aui/artTemplate) 进行 HTML 渲染，并通过修改源码内置了两个常用的格式化工具：

* 货币格式化：[accounting.js](http://openexchangerates.github.io/accounting.js/)
* 日期格式化：datetime.js

详见 DEMO：finance-marketres-mobi\js\utility\util-demo.html

# 参考

* 《编写可维护的 JavaScript》
* 《JavaScript 高级程序设计》
* 《JavaScript 权威指南》
* 《JavaScript 语言精粹》