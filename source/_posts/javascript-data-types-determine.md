title: Javascript 类型判断
date: 2016-02-19 19:43:32
updated:
tags: JavaScript
---

鉴于 ECMAScript 是松散类型（loosely typed）的，因此需要有一种手段来检测给定变量的数据类型——`typeof` 和 `instanceof` 操作符提供了这方面的信息：

# typeof 操作符

`typeof` 操作符可能返回下列某个字符串：

|类型字符串|描述|
|---|---|
|`undefined`|如果这个值未定义|
|`boolean`|如果这个值是布尔值|
|`string`|如果这个值是字符串|
|`number`|如果这个值是数值|
|`object`|如果这个值是对象或 `null`|
|`function`|如果这个值是函数|

例如：

```javascript
typeof undefined
"undefined"
typeof null
"object"
typeof true
"boolean"
typeof false
"boolean"
typeof ''
"string"
typeof ""
"string"
typeof 70
"number"
typeof 070
"number"
typeof 0xA
"number"
typeof 3.125e7
"number"
typeof 3e-7
"number"
typeof NaN
"number"
typeof Infinity
"number"
```

## null

由于在检测对象的值时，`typeof` 无法辨别出 `null` 与对象，因此建议使用下列这样的判断：

```javascript
var my_value = null;
if (my_value && typeof my_value === 'object') {    // null 值为 false
    // my_value 是一个对象或数组！
}
```

## NaN

`typeof` 无法辨别出 `NaN` 和数字：

```javascript
typeof NaN === 'number';    // true
```

`isNaN()` 函数可以解决这类判断问题：

```javascript
isNaN(NaN);     // true
```

## Infinity

`typeof` 无法辨别出 `Infinity` 和数字：

```javascript
typeof Infinity === 'number';    // true
```

可以自定义一个 `isNumber()` 函数用于判断数字：

```javascript
function isNumber(value) {
    return typeof value === 'number' 
        && isFinite(value);    // isFinite 函数会筛选掉 NaN 和 Infinity
}

isNumber(100);    // true
isNumber(NaN);    // false
isNumber(Infinity);    // false
```

## function

比较特殊的类型是 `function`：

```javascript
typeof function(){}
"function"
```

从技术角度讲，函数在 ECMAScript 中是对象，而不是一种数据类型。然而，函数也确实有一些特殊的属性，因此通过 `typeof` 操作符来区分函数和其他对象是有必要的。

# instanceof 操作符

`typeof` 操作符存在一个问题：在判断任何引用类型时都会返回 `"object"`，因此 ECMAScript 引入了 `instanceof` 操作符来解决这个问题：

```javascript
[] instanceof Array
true
new Date() instanceof Date
true
```

# 参考

* 《JavaScript 高级程序设计》