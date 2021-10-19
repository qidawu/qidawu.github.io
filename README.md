# hexo 版本升级

hexo 版本太老，可能会导致主题安装后 `hexo s` 运行错误，可升级解决：

1. 全局升级 hexo-cli，先 `hexo version` 查看当前版本，然后 `npm install hexo-cli -g`，再次 `hexo version` 查看是否升级成功。

2. [npm-check](https://www.npmjs.com/package/npm-check)

   > Check for outdated, incorrect, and unused dependencies.

   1. install this utility as global npm-module: `npm install npm-check -g`
   2. 运行 `npm-check`，检查系统中的插件是否有升级的，可以看到自己前面都安装了那些插件。

3. [npm-upgrade](https://www.npmjs.com/package/npm-upgrade)

   > Interactive CLI utility to easily update outdated NPM dependencies with changelogs inspection support.

   1. install this utility as global npm-module: `npm install npm-upgrade -g`
   2. 运行 `npm-upgrade`，升级系统中的插件。

4. 使用 `npm update -g` 和 `npm update --save`。

# hexo-theme-next 版本升级

A new version of NexT will be released every month. Please read the [release notes](https://github.com/next-theme/hexo-theme-next/releases) before updating the theme. You can update NexT by the following command.

Install the latest version throuth npm:

```bash
$ cd hexo-site
$ npm install hexo-theme-next@latest
```

