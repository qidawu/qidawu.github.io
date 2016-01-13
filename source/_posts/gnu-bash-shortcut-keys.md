title: "GUN/Bash 系列之快捷键总结"
date: 2015-04-19 14:34:30
updated: 
tags: GNU/Linux
description: "下面总结的是 GNU/Bash 中部分最实用的命令名称及默认情况下它们关联的快捷键。更多快捷键请参考 GNU/Bash 文档的 Readline 库。"
---

下面总结的是 GNU/Bash 中部分最实用的命令名称及默认情况下它们关联的快捷键。更多快捷键请参考 GNU/Bash 文档的 Readline 库。

# 移动（Commands for Moving）

|Shortcut|Command name|Description|
|---|---|---|
|`Ctrl + A`|beginning-of-line|Move to the start of the current line.|
|`Ctrl + E`|end-of-line|Move to the end of the line.|
|`Ctrl + F`|forward-char|Move forward a character.|
|`Ctrl + B`|backward-char|Move back a character.|
|`Alt + F`|forward-word|Move forward to the end of the next word. Words are composed of alphanumeric characters (letters and digits).|
|`Alt + B`|backward-word|Move back to the start of the current or previous word. Words are composed of alphanumeric characters (letters and digits).|
|`Ctrl + L`|clear-screen|Clear the screen leaving the current line at the top of the screen. With an argument, refresh the current line without clearing the screen.|

# 操纵历史行（Commands for Manipulating the History）

|Shortcut|Command name|Description|
|---|---|---|
|`Ctrl + P`|previous-history|Fetch the previous command from the history list, moving back in the list.|
|`Ctrl + N`|next-history|Fetch the next command from the history list, moving forward in the list.|
|`Ctrl + R`|reverse-search-history|Search backward starting at the current line and moving `up' through the history as necessary. This is an incremental search.|
|`Alt + Ctrl + Y`|yank-nth-arg|Insert the first argument to the previous command (usually the second word on the previous line) at point. With an argument n, insert the nth word from the previous command (the words in the previous command begin with word 0). A negative argument inserts the nth word from the end of the previous command. Once the argument n is computed, the argument is extracted as if the "!n" history expansion had been specified.|

# 改变文本（Commands for Changing Text）

|Shortcut|Command name|Description|
|---|---|---|
|`Ctrl + D`|delete-char|Delete the character at point. If point is at the beginning of the line, there are no characters in the line, and the last character typed was not bound to delete-char, then return EOF.|

# 剪切和粘贴（Killing and Yanking）

|Shortcut|Command name|Description|
|---|---|---|
|`Ctrl + K`|kill-line|Kill the text from point to the end of the line.|
|`Ctrl + U`|backward-kill-line|Kill backward from point to the beginning of the line.|
|`Alt + D`|kill-word|Kill from point to the end of the current word, or if between words, to the end of the next word. Word boundaries are the same as those used by forward-word.|
|`Ctrl + W`|backward-kill-word|Kill the word behind point, using white space as a word boundary. The killed text is saved on the kill-ring.|
|`Ctrl + Y`|yank|Yank the top of the kill ring into the buffer at point.|

# 补全（Completing）

|Shortcut|Command name|Description|
|---|---|---|
|`TAB`|complete|Attempt to perform completion on the text before point. Bash attempts completion treating the text as a variable (if the text begins with $), username (if the text begins with ~), hostname (if the text begins with @), or command (including aliases and functions) in turn. If none of these produces a match, filename completion is attempted.|

# 参考

《[GNU/Bash Readline Commands](http://www.gnu.org/software/bash/manual/bashref.html#Bindable-Readline-Commands)》