title: "Vim 使用总结"
date: 2015-09-17 21:50:05
updated: 
tags: GNU/Linux
---

![Vim](/img/vim/vim.png)

使用 [Vim](http://www.vim.org/) 也有好几年了，虽然这款“编辑器之神”的学习曲线非常陡峭，但一旦上手将会极大提高文本编辑效率，因此值得投入精力学习。

本文我将会从三个方面总结 Vim 的知识。

# 模式

Vim 效率之高的秘密，就在于它拥有多种“模式”。如果你已经习惯了 Windows 下的编辑器，这些模式在一开始会很违反你的使用直觉。因此学习 Vim 的第一件事，就是要习惯这些模式之间的切换。

Vim 共具有 6 种基本模式和 5 种派生模式，下面只介绍最常用的 4 个基本模式：

## 普通模式

Vim 启动后的默认模式。这正好和许多新用户期待的操作方式相反，因为大多数编辑器的默认模式为插入模式（就是一打开编辑器就可以开始码字）。

Vim 强大的编辑能力中很大部分是来自于其普通模式的命令（及组合）。在普通模式下，用户可以执行一般的编辑器命令，比如移动光标，删除文本等等。如果进一步学习各种各样的文本间移动／跳转命令和其它编辑命令，并且能够**灵活组合使用**的话，能够比那些没有模式的编辑器更加高效的进行文本编辑。

下面介绍普通模式下几类常用的快捷键：

### 移动光标

|快捷键|说明|
|---|---|
|`hjkl`|VIM allows using the cursor keys in order to move around. However, for a pure VIM experience you should stick to using 'h', 'j', 'k' and 'l'. It's considered more efficient since you don't have to move your hand from the home row when you're typing.|
|`gg`|到第一行|
|`G`|到最后一行|
|`nG`|到第 n 行|
|`0`|到行头|
|`^`|到本行第一个非 blank 字符的位置（所谓 blank 字符就是空格、tab、换行、回车等）|
|`$`|到行尾|
|`w`|到下一个单词的开头（`b` 反向）|
|`e`|到下一个单词的结尾|
|`%`|匹配括号移动，包括 () {} []（需要先把光标先移到括号上）|
|`*`|匹配光标当前所在的单词（`#` 反向）|

### 编辑命令

|快捷键|说明|
|---|---|
|`x`|Delete current character|
|`dd`|Cut current line (剪切，可以粘贴)|
|`yy`|Copy current line (yank)|
|`p`|Paste|

|快捷键|说明|
|---|---|
|`<<`|左缩进|
|`>>`|右缩进|
|`=`|自动缩进|
|`Ctrl + p`|在 Insert 模式下，自动补全…|

### 重复命令

|快捷键|说明|
|---|---|
|`.`|重复执行上一个命令|
|`n<command>`|重复执行某个命令 n 次|
|`<start position><command><end position>`|对某段起止文本执行某个命令，例如：`d`（删除）、`y`（复制）、`v`（选择）、`gU`（变大写）、`gu`（变小写）|

## 插入模式

在这个模式中，大多数按键都会向文本缓冲中插入文本。大多数新用户希望文本编辑器在编辑过程中一直保持这个模式。

使用以下快捷键进入插入模式：

|快捷键|说明|
|---|---|
|`i`|Switch to insert mode on current character|
|`a`|Switch to insert mode after current character|
|`I`|Switch to insert mode on first visible character of the current line|
|`A`|Switch to insert mode on last visible character of the current line|

## 可视模式

这个模式与普通模式比较相似。但是移动命令会扩大高亮的文本区域。高亮区域可以是字符、行或者是一块文本。当执行一个非移动命令（例如复制、删除）时，命令会被执行到这块高亮的区域上。

使用以下快捷键进入可视模式：

|快捷键|说明|
|---|---|
|`Ctrl + v`|Switch to visual block mode|
|`v`|Switch to visual character mode|
|`V`|Switch to visual line mode|

## 命令行模式

在命令行模式中可以输入命令。在命令执行完后，Vim 返回到命令行模式之前的模式，通常是普通模式。

使用以下快捷键进入命令行模式：

|快捷键|说明|
|---|---|
|`:`|执行命令（`:%s/vivian/sky/g` 替换每一行中所有 vivian 为 sky）|
|`!`|过滤命令|
|`/` 或 `?`|搜索字符串|

# 配置

使用 Vim 年月较久后总会定制一套个性化的 Vim 配置，例如截取一段常用的 `~/.vimrc` 配置：

```
set number                  " 显示行号
set cursorline              " 突出显示当前行
set ruler                   " 打开状态栏标尺
set shiftwidth=4            " 设定 << 和 >> 命令缩进时的宽度为 4
set softtabstop=4           " 使得按退格键时可以一次删掉 4 个空格
set tabstop=4               " 设定 tab 长度为 4
set nowrapscan              " 禁止在搜索到文件两端时重新搜索
set incsearch               " 输入搜索内容时就显示搜索结果
set hlsearch                " 高亮显示搜索结果
syntax on                   " 程序语法开关
inoremap jj <ESC>           " 重映射 ESCAPE 键
" 定义缩写：ab [缩写] [要替换的文字]
ab asap as soon as possible
```

另外注意，Vim 的操作记录会写入 `~/.viminfo` 。

# GVim

[GVim](http://www.vim.org) 是 Windows 版的 Vim，因为有了标准的 Windows 风格的图形界面，所以叫 G(Graphical)Vim。

GVim 的多标签切换：

|快捷键|说明|
|---|---|
|`:tabnew`|新建标签页|
|`:tabs`|显示已打开标签页的列表|
|`:tabc`|关闭当前标签页|
|`:tabn`|移动到下一个标签页|
|`:tabp`|移动到上一个标签页|
|`:tabfirst`|移动到第一个标签页|
|`:tablast`|移动到最后一个标签页|

字符集配置参考 [这里](http://sunchuanzhen.blog.51cto.com/3076506/670193)，其它小技巧参考 [这里](http://www.cnblogs.com/alphaqiu/archive/2012/04/12/2444147.html) 。

# 参考

* 《[Vim - wikipedia](https://zh.wikipedia.org/wiki/Vim)》
* 《[简明 Vim 练级攻略](http://coolshell.cn/articles/5426.html)》
* 《[如何在 Vim 中得到你最喜爱的 IDE 特性](http://coolshell.cn/articles/894.html)》
* 《[为什么 Vim 使用 HJKL 键作为方向键](http://www.oschina.net/news/28608/vim-direction-keys)》