title: Javascript 引用类型之 Object
date: 2016-02-17 10:54:08
updated:
tags: 前端
---

# Object

ECMAScript 中使用最多的类型就是 `Object`。虽然 `Object` 的实例不具备多少功能，但对于在应用程序中存储和传输数据而言，它们是非常理想的选择。

## 创建方式

创建 `Object` 实例的方式有两种。第一种是使用 `new` 操作符后跟 `Object` 构造函数，如下所示：

```javascript
var person = new Object();
person.name = "Nicholas";
person.age = 29;
```

另一种方式是使用对象字面量表示法。对象字面量是对象定义的一种简写形式，目的在于简化创建包含大量属性的对象的过程：

```javascript
var person = {
    name : "Nicholas",
    age : 29
};
```

使用这种对象字面量语法要求的代码量更少，而且能够给人以封装数据的感觉。

实际上，对象字面量也是向函数传递大量可选参数的首选方式。一般来讲，命名参数虽然容易处理，但在有多个可选参数的情况下就会显示不够灵活。例如：

```javascript
function doSomething(arg0, arg1, arg2, arg3, arg4) {　　
    ... 
}

doSomething('', 'foo', 5, [], false);    // 这里必须传够五个命名参数，无法跳过中间某个可选参数
```

但最好的做法是对那些必需值使用命名参数，而使用对象字面量来封装多个可选参数：

```javascript
function doSomething() {
    // 不传任何参数也能正常运行
    if (!arguments[0]) {
        return false;
    }

    // 为 undefined 的参数设置默认值
    var oArgs = arguments[0]
        arg0  = oArgs.arg0 || "",
        arg1  = oArgs.arg1 || "",
        arg2  = oArgs.arg2 || 0,
        arg3  = oArgs.arg3 || [],
        arg4  = oArgs.arg4 || false;
}

// 传入可选参数而不报错
doSomething({
    arg1: "foo",
    arg2: 5,
    arg4: false
});
```

## 属性和方法

由于在 ECMAScript 中 `Object` 是所有对象的基础，因此所有对象都具有下列这些基本的属性和方法：

|属性|描述|
|---|---|
|`constructor`|保存着用于创建当前对象的函数。|
|`isPrototypeOf(object)`|用于检查传入的对象是否是传入对象的原型。|
|`hasOwnProperty(propertyName)`|用于检查给定的属性在当前对象实例中（而不是在实例的原型中）是否存在。|
|`propertyIsEnumerable(propertyName)`|用于检查给定的属性是否能够使用 `for-in` 语句来枚举。|
|`toLocaleString()`|返回对象的字符串表示，该字符串与执行环境的地区对应。|
|`toString()`|返回对象的字符串表示。|
|`valueOf()`|返回对象的字符串、数值或布尔值表示。通常与 `toString()` 方法的返回值相同。|

例如，要检查某个对象的专有属性，可以使用 `hasOwnProperty(propertyName)` 方法进行判断：

```javascript
var obj = {foo: 'foo', bar: 'bar'};
obj.hasOwnProperty('foo')    // true
obj.hasOwnProperty('constructor')    // false
```

# 参考

* 《JavaScript 高级程序设计》
* 《JavaScript 语言精粹》