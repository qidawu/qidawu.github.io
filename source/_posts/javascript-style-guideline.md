---
title: Javascript 编程风格规范
date: 2016-03-09 22:03:01
updated:
tags: JavaScript
---

# 前言

在团队开发中，所有的代码看起来风格一致是极其重要的，原因有以下几点：

* 任何开发者都不会在乎某个文件的作者是谁，因为所有代码排版格式看起来应当是非常一致，不该花费额外精力去理解代码逻辑并重新排版。
* 风格一致能够让人很容易识别出问题代码并发现错误。如果所有代码看起来很像，当你看到一段与众不同的代码时，很可能错误就产生在这段代码中。

当项目变得庞大时，统一的编程风格能够节省的大量时间成本。

# 基本编程风格

本节编程风格（Style Guideline）是用于规范单文件中的代码，使团队编程风格保持一致。

## 缩进层级

每一行的层级由 **四个空格** 组成，避免使用制表符（Tab）进行缩进，以便在所有的系统和编辑器中，文件的展现格式不会有任何差异。建议在文本编辑器中配置敲击 `Tab` 键时插入四个空格。

```bash
// 好的写法
if (true) {
    doSomething();
}
```

## 行的长度

每行长度不应该超过 80 个字符。如果一行多于 80 个字符，应当在一个**运算符（逗号、加号等）后换行**。下一行应当**增加两级缩进（8 个字符）**。

```bash
// 好的写法
doSomething(arg1, arg2, arg3, arg4,
        arg5);

// 不好的写法：第二行只有 4 个空格的缩进
doSomething(arg1, arg2, arg3, arg4,
    arg5);

// 不好的写法：在运算符之前换行
doSomething(arg1, arg2, arg3, arg4
        , arg5);
```

## 语句格式

* 始终使用分号 `;` 结束一个语句。禁止省略分号，因为：
  * 后续使用构建工具时，可以通过自动删除多余的空格和换行来压缩代码行（代码行结尾处没有分号会导致压缩错误）。
  * 在某些情况下增进代码的性能，因为这样解析器就不必再花时间推测应该在哪里插入分号了。
  * 避免解析器错误的插入分号，导致程序报错。
* 始终使用花括号 `{}` 包住块语句，可以让编程意图更清晰，降低修改代码时出错的几率。

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

// 不好的写法，缺少适当的换行
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

## 操作符间隔

二元操作符（如赋值、逻辑运算）前后必须使用一个空格来保持表达式的整洁。

```javascript
// 好的写法
var found = (value[i] === item);

// 不好的写法：丢失了空格
var found=(value[i]===item);
```

## 注释声明

注释有时候可以用于给一段代码声明额外的信息。这些声明的格式如下：

| 注释声明     | 描述                                       |
| -------- | ---------------------------------------- |
| `TODO`   | 说明代码还未完成。此时应当描述下一步要做的事情。                 |
| `HACK`   | 说明代码实现走了一个捷径。此时应当描述为何使用 hack 的原因。这也可能表明该问题可能会有更好的解决方法。 |
| `FIXME`  | 说明代码是有问题的需要尽快修复。此时应当描述问题出在哪里，或者提供解决方案。   |
| `REVIEW` | 说明代码任何可能的改动都需要评审。                        |

注释声明可以用于单行或多行注释，例如：

```javascript
// TODO: 我希望找到一种效率更快的实现方式
doSomething();

/*
 * HACK: 不得不针对 IE 做的特殊处理。我计划后续有时间时
 * 重写这部分。这些代码可能需要在 v1.2 版本之后替换掉。
 */
if (document.all) {
    doSomething();
}
```

## 变量命名

* 变量命名使用小驼峰式（Camel Case）命名法，即以小写字母开头，后续每个单词首字母都大写。
* 常量命名使用大写字母和下划线。
* 私有属性、方法使用下划线前缀：`_`。

### 常量

所有字母大写，不同单词之间用单个下划线 `_` 分隔。

### 构造函数

构造函数使用大驼峰式（Pascal Case）命名法，即以大写字母开头，后续每个单词首字母都大写。

### 函数变量

函数变量使用前缀：`fn`。

### DOM 变量

* class：使用全小写字母 + 中划线的形式命名。如果该类是用于在 JS 中引用的，还需要添加前缀 `js-`。注意用于 JS 的类严禁用于样式文件中引用。
* id：使用小驼峰命名，并添加前缀如下：

| 前缀    | 描述        |
| ----- | --------- |
| `ipt` | input 输入框 |
| `btn` | 按钮        |
| `lbl` | Label     |
| `chk` | CheckBox  |
| `lnk` | A链接       |
| `img` | 图片        |

## 禁止使用的

### 包装类型

JavaScript 中有三种[基本包装类型](/2016/02/11/javascript-primitive-types/#基本包装类型)：`Boolean`、`Number`、`String`，每种类型都代表全局作用域中的一个构造函数，并分别表示各自对应的原始值的对象。基本包装类型的主要作用是让原始值具有对象般的行为。

禁止使用这些基本包装类型声明变量，应该直接使用对应的字面量：

| 类型                                   | 描述                                       | 注意项                  |
| ------------------------------------ | ---------------------------------------- | -------------------- |
| 布尔值                                  | 统一使用字面量 `true`、`false` 而不是构造函数 `new Boolean()` |                      |
| 数字值                                  | 统一使用[字面量](/2016/02/11/javascript-primitive-types/#字面量)，而不是构造函数 `new Number()` | 避免使用八进制字面量           |
| 字符串                                  | 统一使用单引号 `''`，而不是构造函数 `new String()`      | 避免在字符串中使用斜杠 `\` 另起一行 |
| [对象](/2016/02/17/javascript-object/) | 统一使用字面量 `{}` 而不是构造函数 `new Object()`      |                      |
| [数组](/2016/02/18/javascript-array/)  | 统一使用字面量 `[]` 而不是构造函数 `new Array()`       |                      |

### 等号操作符

由于相等（`==`）和不相等（`!=`）操作符存在 **自动类型转换** 的问题，因此禁止使用。为了保持代码中数据类型的完整性，要求使用全等（`===`）和不全等（`!==`）操作符。

### 代码执行

`setTimeout()`、`setInterval()` 函数中的回调代码禁止使用字符串格式。

`eval()` 函数禁止使用。

## 空链接跳转

常用的三种空链接跳转：

```javascript
#
javascript:void(0);
javascript:;    // 推荐这种
```

# 进阶编程风格

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
// 每个变量声明都独占一行，同时注意每行的缩进
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

此外，还可以使用[立即执行的匿名函数（immediately executed anonymous function）](http://www.zuojj.com/archives/631.html)来创建私有作用域，从而解决全局变量污染的问题。这种函数一般是没有返回值的：

```javascript
(function() {
    var hidden_variable = 'Hello world!';    // hidden_variable 只是一个局部变量
})()
```

要注意的是在这种场景下，函数表达式外的那对圆括号 `()` 绝不能省略，因为官方的语法假定以单词 `function` 开头的语句是一个函数声明语句，而函数声明语句是无法匿名的，否则会报错。

# 参考

* 《编写可维护的 JavaScript》
* 《JavaScript 高级程序设计》
* 《JavaScript 权威指南》
* 《JavaScript 语言精粹》