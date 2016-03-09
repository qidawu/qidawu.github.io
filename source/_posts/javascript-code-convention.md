title: Javascript 前端开发规范
date: 2016-03-05 20:03:01
updated:
tags: JavaScript
---

# 前言

在团队开发中，所有的代码看起来风格一致是极其重要的，原因有以下几点：

* 任何开发者都不会在乎某个文件的作者是谁，因为所有代码排版格式看起来应当是非常一致，不该花费额外精力去理解代码逻辑并重新排版。
* 风格一致能够让人很容易识别出问题代码并发现错误。如果所有代码看起来很像，当你看到一段与众不同的代码时，很可能错误就产生在这段代码中。

当项目变得庞大时，统一的编程风格能够节省的大量时间成本。

# 编程风格

本节编程风格（Style Guideline）是用于规范单文件中的代码，使团队编程风格保持一致。

## 缩进层级

使用 **四个空格** 进行缩进，以便在所有的系统和编辑器中，文件的展现格式不会有任何差异。建议在文本编辑器中配置敲击 `Tab` 键时插入四个空格。

## 语句格式

* 始终使用分号 `;` 结尾一个语句。禁止省略分号，因为：
  * 后续使用构建工具时，可以通过自动删除多余的空格和换行来压缩代码行（代码行结尾处没有分号会导致压缩错误）。
  * 在某些情况下增进代码的性能，因为这样解析器就不必再花时间推测应该在哪里插入分号了。
  * 避免解析器错误的插入分号，导致程序报错。
* 始终使用花括号 `{}` 包住块语句，可以让编程意图更清晰，降低修改代码时出错的几率。
* 块语句使用空格间隔。

这里展示了一些例子：

```javascript
// 不好的写法，缺少花括号
if (condition)
    doSomething();

// 不好的写法，左花括号应当放在块语句中第一句代码的末尾
if (condition)
{
    doSomething();
}

// 不好的写法，缺少空格间隔
if(condition){
    doSomething();
}

// 不好的写法，缺少适当换行
if (condition) { doSomething(); }

// 不好的写法，缺少分号结尾
if (condition) {
    doSomething()
}

// 好的写法
if (condition) {
    doSomething();
}
```

## 字面量

JavaScript 中有三种[基本包装类型](/2016/02/11/javascript-primitive-types/#基本包装类型)：`Boolean`、`Number`、`String`，每种类型都代表全局作用域中的一个构造函数，并分别表示各自对应的原始值的对象。基本包装类型的主要作用是让原始值具有对象般的行为。

要避免直接使用这些基本包装类型，应该直接使用对应的字面量：

|类型|描述|
|---|---|
|布尔值|统一使用字面量 `true`、`false` 而不是构造函数 `new Boolean()`|
|数字值|统一使用[字面量](/2016/02/11/javascript-primitive-types/#字面量)，而不是构造函数 `new Number()`|
|字符串|统一使用单引号 `''`，而不是构造函数 `new String()`|
|[对象](/2016/02/17/javascript-object/)|统一使用字面量 `{}` 而不是构造函数 `new Object()`|
|[数组](/2016/02/18/javascript-array/)|统一使用字面量 `[]` 而不是构造函数 `new Array()`|

## 操作符

由于相等（`==`）和不相等（`!=`）操作符存在 **自动类型转换** 的问题，因此禁止使用。为了保持代码中数据类型的完整性，要求使用全等（`===`）和不全等（`!==`）操作符。

## 变量命名

* 变量命名使用小驼峰式（Camel Case）命名法，即以小写字母开头，后续每个单词首字母都大写。
* 常量命名使用大写字母和下划线。
* 函数变量使用前缀：`fn`。
* 函数入参使用下划线前缀：`_`。

### DOM 变量

DOM 变量使用前缀 `js-`，并中缀如下：

|前缀|描述|
|---|---|
|`ipt`|input 输入框|
|`btn`|按钮|
|`lbl`|Label|
|`chk`|CheckBox|
|`lnk`|A链接|
|`img`|图片|

### 构造函数

构造函数使用大驼峰式（Pascal Case）命名法，即以大写字母开头，后续每个单词首字母都大写。

## 变量声明

在具有块级作用域的语言中，在狭小的作用域内让变量声明和使用变量的代码尽可能彼此靠近，通常是个好的编程习惯。因此在编写 JavaScript 时常常会出现类似的惯性思维：

```javascript
for(var i = 0; i < 3; i++) {
    console.log('for 语句内，i=' + i);
}
console.log('for 语句外，i=' + i);    // 注意这里，JavaScript 没有块级作用域，因此 for 语句外仍然可以读取变量 i
```

输出如下：

```javascript
for 语句内，i=0
for 语句内，i=1
for 语句内，i=2
for 语句外，i=3    // 注意这里
```

但由于 JavaScript 中并没有块级作用域（block scope），只有函数作用域（function scope），因此函数内声明的所有变量在函数体内始终是可见的。这个特性被非正式地称为 **声明提前（hoisting）**，即 JavaScript 函数内声明的所有变量（但不涉及赋值）都被“提前”至函数顶部。这步操作是在代码开始运行之前、[JavaScript 引擎](http://www.slideshare.net/lijing00333/javascript-engine)的“预编译”阶段进行的。上述代码编译如下：

```javascript
var i;    // 变量声明提前
for(i = 0; i < 3; i++) {
    console.log('for 语句内，i=' + i);
}
console.log('for 语句外，i=' + i);
```

变量声明提前意味着：在函数内部任意地方声明变量和在函数顶部声明变量是完全一样的。为了让源代码能够非常清晰地反映出真实的变量作用域，避免潜藏错误，规范要求始终在函数顶部使用单 `var` 语句统一声明所有变量，例如：

```javascript
// 每个变量声明都独占一行
var iptUsername = $('input[name="username"]'),
    iptPwd = $('input[name="pwd"]'),
    btnLogin = $('#js-btn-login'),
    fnLogin = function() {};
```

## 函数声明

和上述变量声明提前一样，函数声明也会被 JavaScript 引擎提前（function declaration hoisting）。因此，在代码中函数的调用可以出现在函数声明之前：

```javascript
// 不好的写法
doSomeThing();

function doSomeThing() {
    console.log('Hello world!');
}
```

这段代码是可以正常运行的，因为 JavaScript 引擎将这段代码解析为：

```javascript
// 函数声明提前
function doSomeThing() {
    console.log('Hello world!');
}

doSomeThing();
```

由于 JavaScript 的这种行为会放宽函数必须 **先声明后使用** 的要求，因此会导致代码混乱。

规范要求函数始终 **先声明后使用**。

## 函数表达式

更好的办法是使用 **函数表达式** 代替函数声明：

```javascript
// 好的写法
var doSomeThing = function() {
    console.log('Hello world!');
};

doSomeThing();
```

这种形式看起来像是常规的变量赋值语句，即创建一个函数并将它赋值给变量 `doSomeThing`。这种情况下创建的函数叫做 **匿名函数（anonymous function）**（也称为 **拉姆达函数**），因为 `function` 关键字后面没有标识符，其 `name` 属性为空。

与使用函数声明的区别在于，如果执行顺序颠倒，函数调用 `doSomeThing()` 将会报错。因为函数表达式必须等到解析器执行到它所在的代码行，才会真正被解释执行：

```javascript
typeof doSomeThing === 'undefined';    // true

var doSomeThing = function() {
    console.log('Hello world!');
};
```

除此之外，函数声明与函数表达式的语法其实是等价的。尽管如此，规范仍然要求优先使用函数表达式，原因有二：

* 强制开发者 **先声明后使用** 函数，避免函数声明提升带来的混乱；
* 函数表达式更能明确表示一个包含函数的变量。要学好这门语言，理解 **函数就是对象** 是很重要的。因为函数是对象，所以它们可以像任何其它的值一样被使用。例如：
  * 函数可以保存在变量、对象和数组中；
  * 函数可以被当做 **参数** 传递给其它函数，也可以被作为函数的 **返回值**；
  * 函数可以拥有方法。

## 立即执行的函数

使用函数表达式可以声明匿名函数，并将匿名函数赋值给变量或者属性：

```javascript
var doSomeThing = function() {
    return 'doSomeThing';
};

typeof doSomeThing === 'function';    // true
```

这种匿名函数可以通过在最后加上一对圆括号 `()` 来 **立即执行并返回** 一个值给变量：

```javascript
// 不好的写法
var doSomeThing = function() {
    return 'doSomeThing';
}();

typeof doSomeThing === 'string';    // true
```

这种写法的问题在于，会让人误以为将一个匿名函数赋值给了这个变量。除非读完整段代码并看到最后一行的那对圆括号 `()`，否则你不会知道是将函数赋值给变量还是将函数的执行结果赋值给变量。这种困惑会影响代码的可读性。

为了让立即执行的函数能够被一眼看出来，可以用一对圆括号 `()` 将函数包起来。这样做并不会影响代码的执行结果，却能让人一眼就看出这是个立即执行的函数：

```javascript
// 好的写法
var doSomeThing = (function() {
    return 'doSomeThing';
})();

typeof doSomeThing === 'string';    // true
```

### 创建私有作用域

此外，还可以使用立即执行的匿名函数（immediately executed anonymous function）来创建私有作用域，从而解决全局变量污染的问题。这种函数一般是没有返回值的：

```javascript
(function() {
    var hidden_variable = 'Hello world!';    // hidden_variable 只是一个局部变量
})()
```

要注意的是在这种场景下，函数表达式外的那对圆括号 `()` 绝不能省略，因为官方的语法假定以单词 `function` 开头的语句是一个函数声明语句，而函数声明语句是无法匿名的，否则会报错。

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

如果你编写的是一段不会被其它脚本访问到的完全独立的脚本，可以使用一个立即执行的匿名函数来[创建私有作用域](/2016/03/05/javascript-code-convention/#创建私有作用域)。

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
* 否则可以使用主流的模块框架规范，如 AMD 和 CMD（[AMD 和 CMD 的区别有哪些？](https://www.zhihu.com/question/20351507/answer/14859415)）。

## 事件处理

* 由于 `click` 事件在移动端浏览器有 300 毫秒的延迟，因此要求使用 zepto.js 的 `tap` 事件替代所有 `click` 事件。

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
            .addEventListener("click", doSomeThing, false);    // DOM 2级添加事件
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
* 《JavaScript 权威指南》
* 《JavaScript 语言精粹》