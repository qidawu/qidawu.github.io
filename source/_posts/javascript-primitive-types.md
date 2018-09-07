title: Javascript 基本数据类型
date: 2016-02-11 16:25:12
updated:
tags: 前端
---

尽管 ECMAScript 是一门弱类型语言，但它的内部提供了五种基本数据类型以便开发者使用。下面分别介绍：

# 基本数据类型

## Undefined

在使用 `var` 声明变量但未对其加以初始化时，这个变量的值就是 `undefined`。

## Null

表示一个空对象指针。

## String

String 类型用于表示 16 位 Unicode 字符组成的字符序列，即字符串。字符串可以由双引号（`""`）或单引号（`''`）表示，内含转义字符。

## Boolean

虽然 Boolean 类型只有两个字面值——`true`、`false`，但 ECMAScript 中所有类型的值都有与之等价的值。下表给出了转换规则：

### 转换规则

|数据类型|转换为 `true` 的值|转换为 `false` 的值|
|---|---|---|
|Boolean|`true`|`false`|
|String|任何非空字符串|`""`（空字符串）|
|Number|任何非零数字值（包括无穷大 `Infinity`）|`0`、`NaN`|
|Object|任何对象|`null`|
|Undefined|不适用|`undefined`|

这些转换规则对于理解流控制语句（如 `if` 语句）、布尔操作符（`!`、`&&`、`||`）自动执行相应的 Boolean 转换非常重要，例如：

```javascript
if('false') {console.log('true')}
true    // 输出 true，因为进行了自动类型转换

window.hello;    // undefined，因为该成员属性不存在
var foo = window.hello || 'unknown';    // 布尔操作符 || 可以用来填充默认值
foo;    // 值为 'unknown'

window.hello.world;    // 抛出 TypeError 异常，因为尝试从 undefined 的成员属性中取值
var bar = window.hello && window.hello.world;    // 布尔操作符 && 可以用来避免该异常
bar;    // 值为 undefined
```

## Number

### 字面量

|字面量|描述|
|---|---|
|`70`|十进制的 70。|
|`-70`|十进制的负 70。|
|`070`|八进制的 56。八进制字面值的第一位必须是零(`0`)，然后是八进制数字序列(`0~7`)。|
|`0xA`|十六进制的 10。十六进制字面值的前两位必须是 `0x`，后跟任何十六进制数字(`0~9` 及 `A~F`)。|
|`3.125e7`|科学计数法，表示“3.125 乘以 10 的 7 次幂（`3.125 * Math.pow(10, 7)`）”，即 31250000。推荐使用这种简洁的方式来表示那些极大或极小的数值。|
|`3e-7`|科学计数法，表示 0.0000003。默认情况下，ECMASctipt 会将那些小数点后面带有 6 个零以上的浮点数值转换为以 e 表示法表示的数值。|
|`Infinity`|如果某次计算的结果得到了一个超出 ECMAScript 数值范围的值，那么该值将被自动转换成特殊的 `Infinity` 值。该值将无法继续参与下一次的计算，因为 `Infinity` 不是能够参与计算的数值。要想确定一个数值是不是有穷的（即是否位于最小和最大的数值之间），可以使用 `isFinite()` 函数进行判断。|
|`NaN`|非数值（Not a Number）是一个特殊的数值，用于表示一个本来要返回数值的操作数未返回数值的情况（这样就不会抛出错误了）。ECMAScript 定义了 `isNaN()` 函数，接收任何类型参数并（调用 `Number()` 函数）进行 **自动类型转换**，如果转换失败则这个参数“不是数值”。|

### 浮点数值

Number 类型使用 IEEE754 格式来表示整数和浮点数值。这种格式有个通病：浮点数值计算会产生 **舍入误差** 的问题，从而导致无法测试 **特定的** 浮点数值，例如：

```javascript
if (a + b == 0.3) { // 不要做这样的浮点测试！例如 0.1 加 0.2 的结果不是 0.3，而是 0.30000000000000004。
    alert("You got 0.3.");
}
```

可见，浮点数值的最高精度虽然有 17 位小数，但在进行算术计算时其精确度远远不如整数。因此建议先将浮点数值转换成整数值进行计算后，再转回浮点数，如此一来就能缓解这个问题。

此外，由于保存浮点数值需要的内存空间是保存整数值的 **两倍**，因此 ECMAScript 会不失时机地将浮点数值转换为整数值，例如：

```javascript
var floatNum1 = 1.;    // 小数点后面没有数字——解析为 1
var floatNum2 = 10.0;  // 浮点数值本身表示整数——解析为 10
```

### 数值范围

由于内存限制，ECMAScript 并不能保存世界上所有的数值，其限制范围下表：

|常量|描述|
|---|---|
|`Number.MIN_VALUE`|ECMAScript 能够表示的最小数值，大多数浏览器中为 `5e-324`|
|`Number.MAX_VALUE`|ECMAScript 能够表示的最大数值，大多数浏览器中为 `1.7976931348623157e+308`|

如果某次计算的结果得到了一个超出 ECMAScript 数值范围的值，那么该值将被自动转换成特殊的 `Infinity` 值。该值将无法继续参与下一次的计算，因为 `Infinity` 不是能够参与计算的数值。要想确定一个数值是不是有穷的（即是否位于最小和最大的数值之间），可以使用 `isFinite()` 函数进行判断。

### 数值转换

有 3 个函数可以把非数值转换为数值：`Number()`、`parseInt()` 和 `parseFloat()`。第一个函数可以用于任何数据类型，而另两个函数则专门用于把字符串转换成数值。
但由于 `Number()` 函数在转换字符串时比较复杂而且不够合理，因此更常用的是 `parseInt()` 和 `parseFloat()` 函数。

`parseInt()` 函数在转换字符串时，更多的是看其是否符合数值模式：

```javascript
var num1 = parseInt('  70');       // 70（忽略字符串前面的空格，直至找到第一个非空格字符）
var num2 = parseInt('blue');       // NaN（如果第一个字符不是数字字符或者负号，返回 NaN）
var num3 = parseInt("");           // NaN（转换空字符串，也返回 NaN）
var num4 = parseInt("1234blue");   // 1234（解析直至遇到一个非数字字符）
var num5 = parseInt(22.5);         // 22（小数点并不是有效的数字字符）
```

如果字符串中的第一个字符是数字字符，`parseInt()` 也能够识别出各种整数格式：

```javascript
var num6 = parseInt("70");         // 70(十进制数)
var num5 = parseInt("070");        // 存在分歧，ECMAScript 3 认为是 56 (八进制),ECMAScript 5 认为是 70 (十进制)
var num3 = parseInt("0xA");        // 10(十六进制数)
```

为了消除在使用 `parseInt()` 函数时可能导致的上述困惑，可以为这个函数提供第二个参数：转换时使用的基数（即多少进制）：

```javascript
var num1 = parseInt("10", 2);    // 2 (按二进制解析)
var num2 = parseInt("10", 8);    // 8 (按八进制解析)
var num3 = parseInt("10", 10);   // 10 (按十进制解析)
var num4 = parseInt("10", 16);   // 16 (按十六进制解析)
```

多数情况下，我们要解析的都是十进制数值，因此始终将 10 作为第二个参数是非常必要的。

# 基本包装类型

为了便于操作基本类型值，ECMAScript 还提供了以下 3 个特殊的引用类型，它们都具有与各自的基本类型相应的特殊行为。实际上，每当读取一个基本类型值的时候，后台就会**隐式地**创建一个对应的基本包装类型的对象，从而让我们能够调用一些实用方法来操作这些数据。

|字面量|包装方法|实用方法|
|---|---|---|
|`true`、`false`|`Boolean()`||
|`70` 十进制<br/>`070` 八进制<br/>`0xA` 十六进制<br/>`3.125e7` 科学计数法|`Number()`|`toFixed(fractionDigits)` 按照指定的小数位四舍五入<br/>`toExponential(fractionDigits)` 科学计数法<br/>`toPrecision(precision)`<br/>`toString(radix)` 使用指定基数（即多少进制）将数字转换为字符串|
|`""`、`''`|`String()`|`charAt()`<br/>`concat()`<br/>`substring()`<br/>`indexOf()`<br/>`toLowerCase()`<br/>`match()`<br/>......|

不建议**显式地**创建基本包装类型的对象，因为会造成 `typeof` 操作符判断不符合预期：

```javascript
typeof new Boolean(true)
"object"
typeof new Number(70)
"object"
typeof new String('')
"object"
```

# 快速类型转换

最后是一些类型转换的小技巧：

```javascript
var myVar   = "3.14159",
    str     = ""+myVar, //  to string
    int     = ~~myVar,  //  to integer
    float   = 1*myVar,  //  to float
    bool    = !!myVar,  //  to boolean - any string with length and any number except 0 are true
    array   = [myVar];  //  to array
```

# 参考

* 《JavaScript 高级程序设计》
* 《JavaScript 语言精粹》
* 《[ECMAScript 类型转换](http://www.w3school.com.cn/js/pro_js_typeconversion.asp)》
