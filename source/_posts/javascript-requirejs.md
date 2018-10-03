---
title: 前端模块化框架 Require.js
date: 2016-07-05 22:29:13
updated:
tags: 前端
---

某段时期前端技术选型上使用过 Require.js 解决前端模块化的问题，下面整理了一些简单实践：

# Require.js 使用

第一步，按功能将 JS 分门别类：

- dist/
- app/
  - js/
    - require_config.js
    - entry/
      - main.js
    - module/
      - sub.js
    - lib/
      - require.js
      - zepto.js
    - util/
  - css/
  - img/
  - ......

| 本地路径      | 功能描述        |
| --------- | ----------- |
| js/entry  | 各功能主模块（主入口） |
| js/module | 各功能子模块      |
| js/lib    | 第三方库        |
| js/util   | 自定义库        |

第二步，参考《[Patterns for separating config from the main module](https://github.com/requirejs/requirejs/wiki/Patterns-for-separating-config-from-the-main-module)》将 RequireJS 配置项从各功能主模块中剥离出来，放到 `js/require_config.js` 统一管理：

```javascript
/**
 * 定义 RequireJS 全局配置
 */
var require = {
    paths: {
        mod: '/contextpath/js/module',
        wgt: '/contextpath/js/widget'
        lib: '/contextpath/js/lib',
        util: '/contextpath/js/util'
    }
};
```

第三步，在页面中引入配置文件及入口文件，注意先后顺序：

```html
<!-- 先注册配置 -->
<script src="js/require_config.js"></script>
<!-- 然后引入 require.js ，并载入主模块 main.js -->
<script src="js/lib/require.js" data-main="js/entry/main.js"></script>
```

第四步，使用 `define()` 函数编写子模块 `js/module/sub.js`。引入所需的模块，如 zepto：

```javascript
/**
 * 子模块依赖 zepto
 */
define(['lib/zepto'], function($) {
  ...
});
```

第五步，使用 `require()` 函数编写主模块 `js/entry/main.js`：

```javascript
/**
 * 主模块依赖 sub.js
 */
require(['mod/sub'], function(sub) {
  ...
});
```

通过 AMD 规范定义的两个关键函数 `define()` 和 `require()` ，我们可以很轻松的在老版本 ES5 上实现模块化功能，解决依赖关系混乱和全局变量的问题。

# Require.js 构建

需要注意的是，模块拆分之后脚本文件数量会变多，HTTP 请求也会相应增多。使用 RequireJS 的[优化工具](http://www.requirejs.org/docs/optimization.html) `r.js` 合并压缩相关联的脚本文件，可以解决这个问题。

# 参考

JavaScript 模块化技术的起源：《[A JavaScript Module Pattern](http://yuiblog.com/blog/2007/06/12/module-pattern/)》

JavaScript 模块化技术的一些高级特性：《[JavaScript Module Pattern: In-Depth](http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html)》（[中文版](http://blog.csdn.net/flybywind/article/details/8095724)）

RequireJS 的一些入门用法参考：

- 《[Javascript模块化编程（一）：模块的写法](http://www.ruanyifeng.com/blog/2012/10/javascript_module.html)》
- 《[Javascript模块化编程（二）：AMD规范](http://www.ruanyifeng.com/blog/2012/10/asynchronous_module_definition.html)》
- 《[Javascript模块化编程（三）：require.js的用法](http://www.ruanyifeng.com/blog/2012/11/require_js.html)》