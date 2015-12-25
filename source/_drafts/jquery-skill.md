title: jQuery 常用技巧
tags: JavaScript
---

# 基础

## 创建 jQuery 空对象

创建一个新的jQuery对象有时开销会比较大。不过你可以先创建一个空对象，然后通过 `add()` 填充它。

```
var container = $([]);
container.add(another_element);
```

## 创建 jQuery DOM 对象

```
var span = $('<span>', {'class' : 'green', 'text' : '-1,000'});
```

## 克隆一个对象

使用 `.clone()` 方法克隆 JavaScript 中的 DOM 对象：

```
// Clone the DIV
var cloned = $('#yourdivID').clone();
```

```
function handleFundItems(funds) {
     $.each(funds, function(key, val){
            var obj = $("#fund-item").clone();
           $(obj).show();
           $( "#fund-item").after(obj);
     });
}

```

## 缓存 jQuery 的结果

如果你没有别的选择，只能使用DOM选择器，那么你应该缓存jQuery的结果。例如：

```
var selectedListItem = $('li[data-selected="true"]a')
```

现在，jQuery的结果已经被缓存到变量“selectedListItem”，该变量可以多次使用而不会影响性能。

## 把你的代码变成 jQuery 插件 

如果你希望将你的jQuery代码封装成一个jQuery插件，以便以后重用，你可以通过以下代码来创建：

```
function($) {
    $.fn.yourPluginName = function() {
        // Your code goes here
        return this;
    } ;
})(jQuery);
```

## Tab 切换

```
var triggers = container.find('.tab-triggers li' )
var panels = container.find('.tab-panels' ).children();
triggers.click(function() {
     var _this = $( this);
     var index = triggers.index(_this);
     _this.addClass('cur').siblings().removeClass( 'cur');
     panels.eq(index).show().siblings().hide();
});
```

# 其它

## 本地存储

Local storage是一个用于在客户端上存储信息的API。使用时，你只需将你的数据作为localStorage全局对象的一个属性：

```
localStorage.someData = "This data is going to persist across page refreshes and browser restarts";
```

旧的浏览器不支持该 API，不过有各种 [jQuery 插件](http://plugins.jquery.com/plugin-tags/localstorage)可以作为替代方案。这些插件在 localStorage 不可用时提供了其他存储方案。下面是一个例子： 

```
// Check if "key" exists in the storage.
var value = $.jStorage.get("key");
if(!value){
// if not - load the data from the server
value = load_data_from_server();
// and save it
$.jStorage.set("key",value);
```

## 货币格式化

[accounting.js](http://openexchangerates.github.io/accounting.js/)