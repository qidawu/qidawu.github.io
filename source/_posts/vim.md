---
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

## 普通模式（NORMAL MODE）

Vim 启动后的默认模式。这正好和许多新用户期待的操作方式相反，因为大多数编辑器的默认模式为插入模式（就是一打开编辑器就可以开始码字）。

Vim 强大的编辑能力中很大部分是来自于其普通模式的命令（及组合）。在普通模式下，用户可以执行一般的编辑器命令，比如移动光标，删除文本等等。如果进一步学习各种各样的文本间移动／跳转命令和其它编辑命令，并且能够**灵活组合使用**的话，能够比那些没有模式的编辑器更加高效的进行文本编辑。

下面介绍普通模式下几类常用的快捷键：

### 移动命令

跨行移动：

| 快捷键    | 说明                                       |
| ------ | ---------------------------------------- |
| `hjkl` | VIM allows using the cursor keys in order to move around. However, for a pure VIM experience you should stick to using 'h', 'j', 'k' and 'l'. It's considered more efficient since you don't have to move your hand from the home row when you're typing. |
| `gg`   | 到第一行                                     |
| `G`    | 到最后一行                                    |
| `nG`   | 到第 n 行                                   |
| `%`    | 匹配括号移动，包括 () {} []（需要先把光标先移到括号上）         |
| `*`    | 匹配光标当前所在的单词（`#` 反向）                      |

当前行移动：

| 快捷键  | 说明                                       |
| ---- | ---------------------------------------- |
| `0`  | 到行头（`$` 反向）                              |
| `^`  | 到本行第一个非 blank 字符的位置（所谓 blank 字符就是空格、tab、换行、回车等） |
| `w`  | 到下一个单词的开头（`b` 反向）                        |
| `e`  | 到下一个单词的结尾                                |
| `f`  | Find next character（`F` 反向） <br/> `fi ` 到字符 i 处<br/> `4fi` 到第四个字符 i 处 |
| `t`  | Find before character（`T` 反向）            |

![Vim 当前行移动](/img/vim/vim_line_moves.jpg)

### 编辑命令

文本替换：

| 快捷键 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| `r`    | Replace current character <br/> When you need to replace only one character under your cursor, **without changing to insert mode**, use `r`. |

剪切/复制/粘贴：

| 快捷键  | 说明                                       |
| ---- | ---------------------------------------- |
| `x`  | Cut current character                    |
| `d`  | `dd` Cut current line <br/> `dt` Cut till ... |
| `y`  | `yy` Copy current line (yank) <br/> `yt` Copy till ... |
| `p`  | Paste                                    |

缩进/补全：

| 快捷键        | 说明                 |
| ---------- | ------------------ |
| `<<`       | 左缩进                |
| `>>`       | 右缩进                |
| `=`        | 自动缩进               |
| `Ctrl + p` | 在 Insert 模式下，自动补全… |

从别的编辑器里粘贴到 vim 里的代码经常由于不正常的缩进变得格式混乱，可以使用如下命令：

* 自动缩进当前行： `==`

* 全文格式化：`gg=G` ，即：

  1. gg - Goto the beginning of the file
  2. = - apply indentation
  3. G - till end of file

### 重复命令

| 快捷键                                    | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| `.`                                       | 重复执行上一个命令                                           |
| `n<command>`                              | 重复执行某个命令 n 次                                        |
| `<start position><command><end position>` | 对某段起止文本执行某个命令，例如：`d`（删除）、`y`（复制）、`v`（选择）、`gU`（变大写）、`gu`（变小写）。例如：`gg=G` |

## 插入模式（INSERT MODE）

在这个模式中，大多数按键都会向文本缓冲中插入文本。大多数新用户希望文本编辑器在编辑过程中一直保持这个模式。

使用以下快捷键进入插入模式：

### 当前行插入

| 快捷键  | 说明                                       |
| ---- | ---------------------------------------- |
| `i`  | Switch to insert mode **on current character** |
| `a`  | Switch to insert mode **after current character** |
| `I`  | Switch to insert mode **on first visible character of the current line** |
| `A`  | Switch to insert mode **on last visible character of the current line** |

### 另起一行插入

| 快捷键  | 说明                                       |
| ---- | ---------------------------------------- |
| `o`  | Switch to insert mode **after current line** |
| `O`  | Switch to insert mode **before current line** |

## 可视模式（VISUAL MODE）

这个模式与普通模式比较相似。但是移动命令会扩大高亮的文本区域。高亮区域可以是字符、行或者是一块文本。当执行一个非移动命令（例如复制、删除）时，命令会被执行到这块高亮的区域上。

使用以下快捷键进入可视模式：

| 快捷键        | 说明                              |
| ---------- | ------------------------------- |
| `Ctrl + v` | Switch to visual block mode     |
| `v`        | Switch to visual character mode |
| `V`        | Switch to visual line mode      |

## 命令行模式

在命令行模式中可以输入命令。在命令执行完后，Vim 返回到命令行模式之前的模式，通常是普通模式。

使用以下快捷键进入命令行模式：

| 快捷键     | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| `:`        | 执行命令： <br/> `:h` 帮助文档，例如查看 `s`文本替换命令（substitude）的帮助：`:h s` |
| `!`        | 过滤命令                                                     |
| `/` 或 `?` | 搜索字符串                                                   |

### 替换命令（Substitute）

```
4.2 Substitute                                          *:substitute*
                                                        *:s* *:su*
:[range]s[ubstitute]/{pattern}/{string}/[flags] [count]
                        For each line in [range] replace a match of {pattern}
                        with {string}.
                        For the {pattern} see |pattern|.
                        {string} can be a literal string, or something
                        special; see |sub-replace-special|.
                        When [range] and [count] are omitted, replace in the
                        current line only.
                        When [count] is given, replace in [count] lines,
                        starting with the last line in [range].  When [range]
                        is omitted start in the current line.
                        Also see |cmdline-ranges|.
                        See |:s_flags| for [flags].
```

`[range]` 有以下一些表示方法：

```
不写range   ：  默认为光标所在的行。
.           ：  光标所在的行。
1           ：  第一行。
$           ：  最后一行。
33          ：  第33行。
'a          ：  标记a所在的行（之前要使用ma做过标记）。
.+1         ：  当前光标所在行的下面一行。
$-1         ：  倒数第二行。（这里说明我们可以对某一行加减某个数值来
                取得相对的行）。
22,33       ：  第22～33行。
1,$         ：  第1行 到 最后一行。
1,.         ：  第1行 到 当前行。
.,$         ：  当前行 到 最后一行。
'a,'b       ：  标记a所在的行 到 标记b所在的行。

%           ：  所有行（与 1,$ 等价）。

?chapter?   ：  从当前位置向上搜索，找到的第一个chapter所在的行。（
                其中chapter可以是任何字符串或者正则表达式。
/chapter/   ：  从当前位置向下搜索，找到的第一个chapter所在的行。（
                其中chapter可以是任何字符串或者正则表达式。

注意，上面的所有用于range的表示方法都可以通过 +、- 操作来设置相对偏
移量。
```

`[flags]` 有以下一些表示方法：

```
无      ：  只对指定范围内的第一个匹配项进行替换。
g       ：  对指定范围内的所有匹配项进行替换。
c       ：  在替换前请求用户确认。
e       ：  忽略执行过程中的错误。

注意：上面的所有flags都可以组合起来使用，比如 gc 表示对指定范围内的所有匹配项进行替换，并且在每一次替换之前都会请用户确认。
```

例子：

```
1.  替换当前行中的内容：    :s/from/to/    （s即substitude）
    :s/from/to/     ：  将当前行中的第一个from，替换成to。如果当前行含有多个
                        from，则只会替换其中的第一个。
    :s/from/to/g    ：  将当前行中的所有from都替换成to。
    :s/from/to/gc   ：  将当前行中的所有from都替换成to，但是每一次替换之前都
                        会询问请求用户确认此操作。

    注意：这里的from和to都可以是任何字符串，其中from还可以是正则表达式。

2.  替换某一行的内容：      :33s/from/to/g
    :.s/from/to/g   ：  在当前行进行替换操作。
    :33s/from/to/g  ：  在第33行进行替换操作。
    :$s/from/to/g   ：  在最后一行进行替换操作。

3.  替换某些行的内容：      :10,20s/from/to/g
    :10,20s/from/to/g   ：  对第10行到第20行的内容进行替换。
    :1,$s/from/to/g     ：  对第一行到最后一行的内容进行替换（即全部文本）。
    :1,.s/from/to/g     ：  对第一行到当前行的内容进行替换。
    :.,$s/from/to/g     ：  对当前行到最后一行的内容进行替换。
    :'a,'bs/from/to/g   ：  对标记a和b之间的行（含a和b所在的行）进行替换。
                            其中a和b是之前用m命令所做的标记。

4.  替换所有行的内容：      :%s/from/to/g
    :%s/from/to/g   ：  对所有行的内容进行替换。
```

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

| 快捷键         | 说明          |
| ----------- | ----------- |
| `:tabnew`   | 新建标签页       |
| `:tabs`     | 显示已打开标签页的列表 |
| `:tabc`     | 关闭当前标签页     |
| `:tabn`     | 移动到下一个标签页   |
| `:tabp`     | 移动到上一个标签页   |
| `:tabfirst` | 移动到第一个标签页   |
| `:tablast`  | 移动到最后一个标签页  |

字符集配置参考 [这里](http://sunchuanzhen.blog.51cto.com/3076506/670193)，其它小技巧参考 [这里](http://www.cnblogs.com/alphaqiu/archive/2012/04/12/2444147.html) 。

# 参考

* 《[Vim - wikipedia](https://zh.wikipedia.org/wiki/Vim)》
* 《[简明 Vim 练级攻略](http://coolshell.cn/articles/5426.html)》
* 《[如何在 Vim 中得到你最喜爱的 IDE 特性](http://coolshell.cn/articles/894.html)》
* 《[为什么 Vim 使用 HJKL 键作为方向键](http://www.oschina.net/news/28608/vim-direction-keys)》
* 《[vim文本替换命令](https://www.cnblogs.com/wind-wang/p/5768000.html)》