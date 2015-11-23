title: "Linux 系统主要目录介绍"
date: 2015-11-22 23:20:27
updated: 
tags: GNU/Linux
---

从 Windows 转到 Linux 的初学者，往往对 Linux 的目录结构感到无所适从。本文介绍的 FHS 标准，是理解这些目录结构的关键。

# FHS 标准

FHS（Filesystem Hierarchy Standard，文件系统层次结构标准）采用树形结构组织文件，并定义了 Linux 系统中主要目录的用途、所需要的最小构成的文件和目录，同时还给出了例外处理与矛盾处理。多数 Linux 版本采用这种目录组织形式，类似于 Windows 操作系统中 C 盘的文件目录。

事实上，FHS 针对目录树结构仅定义出两层目录（`/` 及 `/usr`、`/var`）底下应该放置什么数据，下面分别介绍这些目录：

## / (root)

在 FHS 标准中，所有的文件和目录都必须出现在根目录 `/` 下，即使它们存储在不同的存储设备或网络主机中。此外还要求根目录 `/` 下必须要有以下目录或符号链接（symbolic links）：

|目录|描述|备注|
|---|---|---|
|`/etc`|Host-specific system configuration|系统配置文件|
|`/dev`|Device files|设备文件|
|`/bin`|Essential command binaries (for use by all users)|重要的执行文件|
|`/sbin`|Essential system binaries (for use by root)|重要的系统执行文件|
|`/lib`|Essential shared libraries and kernel modules|执行文件所需的函数库与内核所需的模块。`/bin` 和 `/sbin` 中二进制文件必要的函数库|
|`/boot`|Static files of the boot loader (include kenerl file、drivers)|系统开机文件|
|`/media`|Mount point for removeable media||
|`/mnt`|Mount point for mounting a filesystem temporarily (include hard disk、U disk、CD、DVD…)||
|`/opt`|Add-on application software packages||
|`/srv`|Data for services provided by this system||
|`/tmp`|Temporary files|临时文件|

注意：
* 由于根目录 `/` 与**开机、还原、系统修复**等操作有关，而开机过程中**仅有根目录会被挂载，其它分区则是在开机完成之后才会持续进行挂载**，因此，根目录下与开机过程有关的目录（即上表前六个目录）不能够与根目录分开到不同分区。
* 由于 FHS 的目录结构已经提供了足够的灵活性，因此标准要求，应用程序禁止在根目录下创建新的子目录，理由如下：
 * 这会额外占用根目录所在分区的空间，但系统管理员基于性能与安全考虑，会希望保持该分区小而简（small and simple）；
 * It evades whatever discipline the system administrator may have set up for distributing standard file hierarchies across mountable volumes.

## /usr (unix software resource)

该目录与软件安装/执行有关。

* FHS 建议所有软件开发者，应该将他们的数据合理的分别放置到这个目录下的子目录，而**不要自行建立该软件自己独立的目录**。
* 由于所有系统默认的软件（distribution 发布者提供的软件）都会放置到 `/usr` 下，因此这个目录有点类似 Windows 系统的 `C:\Windows\` 和 `C:\Program files\` 这两个目录的综合体，系统刚安装完毕时，这个目录会占用最多的硬盘容量。
* 用户后续安装的应用程序，也建议放置到 `/usr` 下。

## /var (variable)

该目录与系统运作过程有关。

如果 `/usr` 是系统安装时会占用较大硬盘容量的目录，那么 `/var` 则是在系统运行时才会渐渐占用硬盘容量的目录。 `/var` 目录主要针对常态性变动的文件，包括缓存（cache）、登录文件（log file）以及某些软件运行所产生的日志文件，因此这个目录会越来越大，建议单独挂载分区。

# 总结

由于根目录所在分区的容量有限，因此像 `/usr`、`/var`、`/home` 这种大目录最好不要与根目录放在同一个分区内，而是建议单独挂载分区。如此一来不但可以提高系统性能，根目录所在的文件系统也不容易发生问题。

# 参考

《[Unix目录结构的来历](http://www.ruanyifeng.com/blog/2012/02/a_history_of_unix_directory_structure.html)》
《[Linux 的文件权限与目录配置](http://vbird.dic.ksu.edu.tw/linux_basic/0210filepermission.php)》
《[FHS 官方文档](http://www.pathname.com/fhs/)》
《[FHS 2.3 官方文档](http://www.pathname.com/fhs/pub/fhs-2.3.html)》