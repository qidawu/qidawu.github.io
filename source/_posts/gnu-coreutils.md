---
title: "GNU/CoreUtils 常用命令总结"
date: 2015-11-10 00:01:22
updated: 
tags: GNU/Linux
typora-root-url: ..
---

GNU/CoreUtils 是一组类 Unix 操作系统所需的基础软件包。它包含三组命令，常用的命令如 `cat`、`ls`、`rm`。学习 GNU/Linux 的第一步，就是要熟悉软件包下常用的命令。下面分别介绍这三组常用的命令：

# File utilities

## Basic operations

| 命令      | 描述                          | 备注                                       |
| ------- | --------------------------- | ---------------------------------------- |
| `cp`    | Copy files and directories  | `cp -rp` 备份目录。<br/>`-r` 递归复制目录，否则提示“略过目录‘xxx’”。<br/>`-p` 保留源文件或目录的属性（包括属主、属组、权限、修改时间等）。<br/>`-f` 强制覆盖。 |
| `mv`    | Move (rename) files         |                                          |
| `rm`    | Remove files or directories | `rm -rf` 强制递归删除文件或目录。<br/>`-r` 递归删除，将指定目录下的所有文件及子目录一并处理。<br/>`-f` 强制删除文件或目录。 |
| `ln`    | Create a link to a file     | `ln -s TARGET LINK_NAME` 创建软链接。          |
| `mkdir` | Create a directory          | `-p` 递归创建目录。                             |
| `rmdir` | Remove empty directories    | `-p` 递归删除空目录，如果目录非空会删除失败并提示：`rmdir: failed to remove 'xxx': Directory not empty` |

## Directory listing

| 命令   | 描述                              | 备注                                                         |
| ------ | --------------------------------- | ------------------------------------------------------------ |
| `ls`   | List directory contents           | `-l` 查看详细信息。<br/>`-a` 显示隐藏文件。<br/>`-d` 仅列出目录本身，而不是列出目录内的文件。列出所有子目录本身：`ls -d */`<br/>`-h` 将文件容量以人类较易读的方式（如GB、KB等）列出来。<br/>`-t` 按时间排序显示，默认为新的排在前面。<br/>`-S` 按文件容量大小排序，而不是用文件名。 |
| `dir`  | List directory contents briefly   | Exactly like `ls -C -b`                                      |
| `vdir` | List directory contents verbosely | Exactly like `ls -l -b`                                      |

## Changing file attributes

| 命令    | 描述                        | 备注                                                         |
| ------- | --------------------------- | ------------------------------------------------------------ |
| `chown` | Change file owner and group | `chown -R owner:group /there/is/a/file`<br/>`-R` 递归修改，常用于一次性更改某一目录内所有的文件、目录。目标属主必须在 `/etc/passwd`。 |
| `chgrp` | Change group ownership      | `-R` 递归修改，目标属组必须在 `/etc/group`。                 |
| `chmod` | Change access permissions   | `chmod [ugoa...][[+-=][rwxX]...][,...]`<br/>可用：`u`、`g`、`o`、`a`，可用权限：`r=4`、`w=2`、`x=1`<br/>例如：`chmod u+x` |
| `touch` | Change file timestamps      | 改变文件访问和修改时间，也可用于快速创建一个文件。           |

## Disk usage

| 命令         | 描述                                       | 备注                                       |
| ---------- | ---------------------------------------- | ---------------------------------------- |
| `df`       | Show disk free space on file systems     | `-h` 以 K，M，G 为单位，更易读的方式显示。<br/>`-i` list inode information instead of block usage |
| `du`       | Show disk usage on file systems          | `-h` 以 K，M，G 为单位，更易读的方式显示。<br/>`-s, --summarize` 汇总显示（等于 `--max-depth=0`）<br/>`-d, --max-depth=N` 显示第 N 层子目录各自的大小，常用于找出最占空间的目录。例如：`du --max-depth=1 -h ./`<br/>`--exclude=PATTERN` Exclude files that match PATTERN. |
| `stat`     | Return data about an inode               |                                          |
| `truncate` | Shrink or extend the size of a file to the specified size | -s 参数指定一个大小：K, M, G, T, P, E, Z, Y       |

![disk usage](https://wizardzines.com/comics/disk-usage/disk-usage.png)

# Text utilities

## Output of entire files

| 命令     | 描述                                                         | 备注                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `cat`    | Concatenates and prints files on the standard output         | 常用于连接并输出多个文件的内容。                             |
| `tac`    | Concatenates and prints files on the standard output in reverse | 常用于反向连接并输出多个文件的内容。                         |
| `nl`     | Numbers lines of files                                       | `-b` 指定行号的方式，主要有 `a` `t`两种：<br/>`-b a` 无论是否是空行，同样列出行号。<br/>`-b t` 默认值，不列出空行行号。 |
| `basenc` | Encode/decode data and print to standard output              | `--base64` same as `base64` program (RFC4648 section 4)<br/>`--base64url` file- and url-safe base64 (RFC4648 section 5)<br/>`--base32` same as `base32` program (RFC4648 section 6)<br/>`--base32hex` extended hex alphabet base32 (RFC4648 section 7)<br/>`--base16` hex encoding (RFC4648 section 8)<br/>`-d, --decode` decode data |
| `base64` | Encodes or decodes [Base64](https://en.wikipedia.org/wiki/Base64), and prints result to standard output |                                                              |
| `base32` | Encodes or decodes [Base32](https://en.wikipedia.org/wiki/Base32), and prints result to standard output |                                                              |

![cat](https://wizardzines.com/comics/cat/cat.png)

https://en.wikipedia.org/wiki/More_(command)

https://en.wikipedia.org/wiki/Less_(Unix)

![less](https://wizardzines.com/comics/less/less.png)

## Output of parts of files

| 命令     | 描述                                        | 备注                                                         |
| -------- | ------------------------------------------- | ------------------------------------------------------------ |
| `head`   | Output the first part of files              | 默认输出 10 行                                               |
| `tail`   | Output the last part of files               | `-n` 输出倒数 n 行（默认输出 10 行）<br/>`-f` 不停读取输出文件的最新内容，常用于实时监视日志输出，用 `Ctrl＋C` 来终止。 |
| `tailf`  |                                             | 等同于 `tail -f -n 10`                                       |
| `split`  | Split a file into pieces                    | 用于按行、按大小分割文件                                     |
| `csplit` | Split a file into context-determined pieces |                                                              |

![head&tail](https://wizardzines.com/comics/head-tail/head-tail.png)

## Operating on sorted files

| 命令   | 描述            | 备注                                   |
| ------ | --------------- | -------------------------------------- |
| `sort` | Sort text files | 详见[本文](/posts/gnu-text-utilities/) |
| `shuf` | Shuffling text  |                                        |
| `uniq` | Uniquify files  | 详见[本文](/posts/gnu-text-utilities/) |

## Operating on fields

| 命令    | 描述                                       | 备注                                   |
| ------- | ------------------------------------------ | -------------------------------------- |
| `cut`   | Print selected parts of lines              | 详见[本文](/posts/gnu-text-utilities/) |
| `paste` | Merge lines of files                       | 合并多个文件的所有行                   |
| `join`  | Joins lines of two files on a common field | 合并两个文件中相同位置的行             |

## Operating on characters

| 命令       | 描述                           | 备注                                   |
| ---------- | ------------------------------ | -------------------------------------- |
| `tr`       | Translate or delete characters | 详见[本文](/posts/gnu-text-utilities/) |
| `expand`   | Convert tabs to spaces         |                                        |
| `unexpand` | Convert spaces to tabs         |                                        |

## Summarizing files

| 命令                                                         | 描述                                                         | 备注                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------- |
| `wc`                                                         | Print the number of bytes, words, and lines in files         | 详见[本文](/posts/gnu-text-utilities/) |
| `sum`                                                        | Checksums and counts the blocks in a file                    |                                        |
| `cksum`                                                      | Checksums (IEEE Ethernet [CRC-32](https://en.wikipedia.org/wiki/Cyclic_redundancy_check)) and count the bytes in a file |                                        |
| `b2sum`                                                      | Computes and checks [BLAKE2b](https://en.wikipedia.org/wiki/BLAKE2b) message digest |                                        |
| `md5sum`                                                     | Computes and checks [MD5](https://en.wikipedia.org/wiki/MD5) message digest |                                        |
| `sha1sum`<br/>`sha224sum`<br/>`sha256sum`<br/>`sha384sum`<br/>`sha512sum` | Computes and checks [SHA-1](https://en.wikipedia.org/wiki/SHA-1)/[SHA-2](https://en.wikipedia.org/wiki/SHA-2) message digests |                                        |

https://en.wikipedia.org/wiki/Checksum

> A **checksum** is a small-sized [block](https://en.wikipedia.org/wiki/Block_(data_storage)) of data derived from another block of [digital data](https://en.wikipedia.org/wiki/Digital_data) for the purpose of [detecting errors](https://en.wikipedia.org/wiki/Error_detection) that may have been introduced during its [transmission](https://en.wikipedia.org/wiki/Telecommunication) or [storage](https://en.wikipedia.org/wiki/Computer_storage).

![checksum](/img/security/Checksum.svg)

https://en.wikipedia.org/wiki/Cryptographic_hash_function

[单向散列算法](/posts/java-cryptography-api/)

![digest](/img/security/Digest.svg)

# Shell utilities

## User information

| 命令        | 描述                                       | 备注                        |
| --------- | ---------------------------------------- | ------------------------- |
| `id`      | Print user identity                      | 显示当前用户的信息（uid、gid、groups） |
| `logname` | Print current login name                 |                           |
| `whoami`  | Print effective user ID                  |                           |
| `groups`  | Print group names a user is in           |                           |
| `users`   | Print login names of users currently logged in |                           |
| `who`     | Print who is currently logged in         |                           |

## System context

| 命令         | 描述                                       | 备注                           |
| ---------- | ---------------------------------------- | ---------------------------- |
| `date`     | Print or set system date and time        | `date +%Y-%m-%d`  2016-12-28 |
| `arch`     | Print machine hardware name              |                              |
| `nproc`    | Print the number of available processors |                              |
| `uname`    | Print system information                 |                              |
| `hostname` | Print or set system name                 |                              |
| `hostid`   | Print numeric host identifier            |                              |
| `uptime`   | Print system uptime and load             | 常用于查看系统负载                    |

## Working context

| 命令         | 描述                                       | 备注       |
| ---------- | ---------------------------------------- | -------- |
| `pwd`      | Print working directory                  | 显示当前所在目录 |
| `stty`     | Print or change terminal characteristics |          |
| `tty`      | Print file name of terminal on standard input |          |
| `printenv` | Print all or some environment variables  |          |

## Modified command invocation

| 命令        | 描述                                      | 备注   |
| --------- | --------------------------------------- | ---- |
| `nohup`   | Run a command immune to hangups         |      |
| `timeout` | Run a command with a time limit         |      |
| `env`     | Run a command in a modified environment |      |

## Process control

| 命令     | 描述                         | 备注   |
| ------ | -------------------------- | ---- |
| `kill` | Send a signal to processes |      |

## Delaying

| 命令      | 描述                         | 备注   |
| ------- | -------------------------- | ---- |
| `sleep` | Delay for a specified time |      |

## Redirection

| 命令    | 描述                                       | 备注                                       |
| ----- | ---------------------------------------- | ---------------------------------------- |
| `tee` | Redirect output to multiple files or processes | 详见[本文](/posts/gnu-text-utilities/) |

## Conditions

| 命令      | 描述                                  | 备注   |
| ------- | ----------------------------------- | ---- |
| `false` | Do nothing, unsuccessfully          |      |
| `true`  | Do nothing, successfully            |      |
| `test`  | Check file types and compare values |      |
| `expr`  | Evaluate expressions                |      |

## Printing text

| 命令     | 描述                             | 备注                                                         |
| -------- | -------------------------------- | ------------------------------------------------------------ |
| `echo`   | Print a line of text             | `-n` Do not print the trailing newline character `\n`.<br/>`-e` 开启转义字符，例如：反斜杠 `\\`、换行符 `\n` |
| `printf` | Format and print data            |                                                              |
| `yes`    | Print a string until interrupted |                                                              |

`yes` 命令小技巧，使用管道自动输入“y”进行文件强制覆盖，方法：`yes | cp 源文件 目的文件`

## Numeric operations

| 命令       | 描述                      | 备注       |
| -------- | ----------------------- | -------- |
| `seq`    | Print numeric sequences |          |
| `numfmt` | Reformat numbers        | 常用于格式化数字 |

## File name manipulation

| 命令         | 描述                                       | 备注     |
| ---------- | ---------------------------------------- | ------ |
| `basename` | Strip directory and suffix from a file name | 截取出文件名 |
| `dirname`  | Strip last file name component           | 截取出目录名 |

# zip 压缩文件查看命令

```bash
zip -rmq dist.zip dist

-q	表示不显示压缩进度状态
-r	表示子目录子文件全部压缩为zip；这部分比较重要，不然的话只有something这个文件夹被压缩，里面的没有被压缩进去
-e	表示你的压缩文件需要加密，终端会提示你输入密码的；还有种加密方法，这种是直接在命令行里做的，比如zip -r -P Password01! modudu.zip SomeDir, 就直接用Password01!来加密modudu.zip了
-m	表示压缩完删除原文件

unzip [选项] dist.zip

-x 文件列表 解压缩文件，但不包括指定的file文件。 
-v 查看压缩文件目录，但不解压。 
-t 测试文件有无损坏，但不解压。 
-d 目录 把压缩文件解到指定目录下。 
-z 只显示压缩文件的注解。 
-n 不覆盖已经存在的文件。 
-o 覆盖已存在的文件且不要求用户确认。 
-j 不重建文档的目录结构，把所有文件解压到同一目录下。
```

| 命令    | 描述 | 备注 |
| ------- | ---- | ---- |
| `zcat`  |      |      |
| `zless` |      |      |
| `zmore` |      |      |
| `zgrep` |      |      |
| `zdiff` |      |      |



# 参考

* 《[List of GNU packages](https://en.wikipedia.org/wiki/List_of_GNU_packages)》
* 《[GNU Core Utilities](https://en.wikipedia.org/wiki/GNU_Core_Utilities)》
  * 《[List of GNU Core Utilities commands](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands)》
  * 《[GNU Coreutils Manual](http://www.gnu.org/software/coreutils/manual/html_node/index.html)》