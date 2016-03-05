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

本节编程风格（Style Guideline）是用于规约单文件中代码的规划。

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

### 循环

始终使用 `for` 循环遍历数组成员，使用 `for in` 循环遍历对象属性，两者不能混用。

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

和上述变量声明提前一样，函数声明也会被 JavaScript 引擎提前。因此，在代码中函数的调用可以出现在函数声明之前：

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

由于 JavaScript 的这种行为，因此规范要求始终先声明函数然后再使用。

但更好的办法是使用 **函数表达式** 代替函数声明：

```javascript
var doSomeThing = function() {
    console.log('Hello world!');
}

doSomeThing();
```

这种写法的好处在于，如果顺序颠倒，函数调用将会报错。因为函数表达式必须等到解析器执行到它所在的代码行，才会真正被解释执行：

```javascript
typeof doSomeThing === 'undefined';    // true

var doSomeThing = function() {
    console.log('Hello world!');
};
```

除此之外，函数声明与函数表达式的语法其实是等价的。

## 立即执行的函数

使用 **函数表达式** 可以声明匿名函数，并将匿名函数赋值给变量或者属性：

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

为了让立即执行的函数能够被一眼看出来，可以用一对圆括号 `()` 将函数包起来，这并不会影响代码的执行结果：

```javascript
// 好的写法，一眼就能看出是立即执行的函数
var doSomeThing = (function() {
    return 'doSomeThing';
})();

typeof doSomeThing === 'string';    // true
```

## 全局变量

* 禁止省略 `var` 操作符定义变量，以避免污染全局变量。

## 操作符

由于相等（`==`）和不相等（`!=`）操作符存在 **自动类型转换** 的问题，为了保持代码中数据类型的完整性，推荐使用全等（`===`）和不全等（`!==`）操作符。

## 字面量

### 字符串

字符串统一单引号 `''`。

### 对象

对象创建统一使用字面量 `{}` 而不是构造函数 `new Object()`。

### 数组

数组创建统一使用字面量 `[]` 而不是构造函数 `new Array()`。

## 事件

* 由于 `click` 事件在移动端浏览器有 300 毫秒的延迟，因此要求使用 zepto.js 的 `tap` 事件替代所有 `click` 事件。

# 编程最佳实践

## HTML 渲染

* Ajax 请求统一返回 `ResponseVO<T>`
* 使用模板引擎 [artTemplate](https://github.com/aui/artTemplate) 进行 HTML 渲染，内置了几个格式化工具：
  * 货币格式化：[accounting.js](http://openexchangerates.github.io/accounting.js/)
  * 日期格式化：datetime.js
  * 算数对象（用于执行数学任务）：[Math](http://www.w3school.com.cn/jsref/jsref_obj_math.asp)

详见 DEMO：finance-marketres-mobi\js\utility\util-demo.html

# 参考

《编写可维护的 JavaScript》
《JavaScript 权威指南》