title: "GNU/CoreUtils 常用命令总结"
date: 2015-11-10 00:01:22
updated: 
tags: GNU/Linux
---

GNU/CoreUtils 是一组类 Unix 操作系统所需的基础软件包。它包含三组命令，常用的命令如 `cat`、`ls`、`rm`。学习 GNU/Linux 的第一步，就是要熟悉软件包下常用的命令。下面分别介绍这三组常用的命令：

# File utilities

## Basic operations

|命令|描述|备注|
|---|---|---|
|`cp`|Copy files and directories||
|`mv`|Move (rename) files||
|`rm`|Remove files or directories||
|`ln`|Create a link to a file||
|`mkdir`|Create a directory||
|`rmdir`|Remove empty directories||

## Directory listing

|命令|描述|备注|
|---|---|---|
|`ls`|List directory contents||
|`dir`|List directory contents briefly|Exactly like `ls -C -b`|
|`vdir`|List directory contents verbosely|Exactly like `ls -l -b`|

## Changing file attributes

|命令|描述|备注|
|---|---|---|
|`chown`|Change file owner and group|chown root:root /there/is/a/file|
|`chgrp`|Change group ownership||
|`chmod`|Change access permissions||
|`touch`|Change file timestamps|可用于快速创建一个文件|

## Disk usage

|命令|描述|备注|
|---|---|---|
|`df`|Show disk free space on file systems||
|`du`|Show disk usage on file systems||
|`stat`|Return data about an inode||
|`truncate`|Shrink or extend the size of a file to the specified size|-s 参数指定一个大小：K, M, G, T, P, E, Z, Y|

# Text utilities

## Output of entire files

|命令|描述|备注|
|---|---|---|
|`cat`|Concatenates and prints files on the standard output|常用于连接并输出多个文件的内容|
|`tac`|Concatenates and prints files on the standard output in reverse|常用于反向连接并输出多个文件的内容|
|`nl`|Numbers lines of files||
|`base64`|base64 encode/decode data and print to standard output||

## Output of parts of files

|命令|描述|备注|
|---|---|---|
|`head`|Output the first part of files||
|`tail`|Output the last part of files||
|`split`|Split a file into pieces|用于按行、按大小分割文件|
|`csplit`|Split a file into context-determined pieces||

## Operating on sorted files

|命令|描述|备注|
|---|---|---|
|`sort`|Sort text files||
|`shuf`|Shuffling text||
|`uniq`|Uniquify files||

## Operating on fields

|命令|描述|备注|
|---|---|---|
|`cut`|Print selected parts of lines||
|`paste`|Merge lines of files|合并多个文件的所有行|
|`join`|Joins lines of two files on a common field|合并两个文件中相同位置的行|

## Operating on characters

|命令|描述|备注|
|---|---|---|
|`tr`|Translate or delete characters||
|`expand`|Convert tabs to spaces||
|`unexpand`|Convert spaces to tabs||

## Summarizing files

|命令|描述|备注|
|---|---|---|
|`wc`|Print the number of bytes, words, and lines in files||
|`sum`|||
|`cksum`|||
|`md5sum`|||
|`sha1sum`|||
|`sha256sum`|||
|`sha512sum`|||

## Formatting file contents

不常用。

|命令|描述|备注|
|---|---|---|
|`fmt`|Reformat paragraph text||
|`pr`|Paginate or columnate files for printing||
|`fold`|Wrap input lines to fit in specified width||

# Shell utilities

|命令|描述|备注|
|---|---|---|
|``|||
|``|||
|``|||

# Other utilities

|命令|描述|备注|
|---|---|---|
|``|||
|``|||
|``|||