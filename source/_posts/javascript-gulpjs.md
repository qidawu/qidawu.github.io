---
title: 自动化构建工具 Gulp.js
date: 2016-12-05 22:35:50
updated:
tags: 前端
---

下表整理了自动化构建工具 [Gulp.js](https://www.gulpjs.com.cn/) 在实践项目中常用的插件：

| 插件                                       | 功能                                       | 备注                                       |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| [gulp-jshint](https://www.npmjs.com/package/gulp-jshint) | 检查 JavaScript 语法                         | http://jshint.com/docs/options/          |
| [gulp-uglify](https://www.npmjs.com/package/gulp-uglify) | 压缩 JavaScript                            |                                          |
| [gulp-sourcemaps](https://www.npmjs.com/package/gulp-sourcemaps) | 输出 sourcemaps                            | 部署前端之前，开发者通常会对代码进行打包压缩，这样可以减少代码大小，从而有效提高访问速度。然而，压缩代码的报错信息是很难Debug的，因为它的行号和列号已经失真。这时就需要Source Map来还原真实的出错位置了。 |
| [gulp-imagemin](https://www.npmjs.com/package/gulp-imagemin) | 压缩图片                                     | `progressive` JPEG 图像渐进式扫描；`interlaced` GIF 图像隔行扫描 |
| [gulp-rev](https://www.npmjs.com/package/gulp-rev) | 静态资源 hash                                | 在实际生产环境中，我们页面引用的静态资源的文件名都是带版本号的（非覆盖式升级），这样方便版本管理（如更新与回滚）和防止缓存。通常我们使用文件的md5编码作为版本号，生成文件指纹。 |
| [gulp-less](https://www.npmjs.com/package/gulp-less) | Less 文件编译                                | 用于引入 Less 扩展 CSS 语言，提升前端样式的开发效率。         |
| [gulp-autoprefixer](https://www.npmjs.com/package/gulp-autoprefixer) | 根据所需兼容的浏览器版本，自动补全厂商前缀                    |                                          |
| [gulp-css-base64](https://www.npmjs.com/package/gulp-css-base64/) | 将CSS 样式表中引用的图片和字体通过 base64 编码压缩合并到一起，减少文件请求数 | `maxWeightResource` 资源最大阈值，默认为 32K       |
| [gulp-ejs](https://www.npmjs.com/package/gulp-ejs) | 编译 HTML 中的 ejs 模板，可用于页面布局拆分，提升代码复用性      | http://ejs.co/                           |
| [gulp-htmlmin](https://www.npmjs.com/package/gulp-htmlmin) | 压缩 HTML                                  |                                          |
| [gulp-inline-source](https://www.npmjs.com/package/gulp-inline-source) | 将 HTML 外部引用的样式和脚本以内联的方式嵌到 HTML 文件中，减少文件请求数 |                                          |
| [gulp-if](https://www.npmjs.com/search?q=gulp-if) | 编译时动态判断                                  |                                          |
| [gulp-replace](https://www.npmjs.com/package/gulp-replace) | 编译时动态替换字符串                               | 可用于根据不同环境构建代码，解决各环境间的差异。例如正则匹配并全局替换 HTML 中的 `${web}` 变量。 |
| [gulp-tar](https://www.npmjs.com/package/gulp-tar) | 打包静态资源                                   | [tar-stream](https://github.com/mafintosh/tar-stream) |
| [gulp-mock-server](https://github.com/sanyueyu/gulp-mock-server) | API Mock Server，前后端分离后的 API 模拟利器         |                                          |
| [browser-sync](https://www.npmjs.com/package/browser-sync) | 监听本地文件变化并同步刷新浏览器，提升开发效率的利器               | https://www.browsersync.io/              |