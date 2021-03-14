---
title: C/C++ 语言系列（一）IDE 准备
date: 2021-02-01 22:07:11
updated: 
tags: [C/C++]
typora-root-url: ..
---

# VS Code 运行 C++

打开 VScode，进入 `Extensions` 模块，搜索以下扩展并安装： 

* C/C++
* C/C++ Clang Command Adapter
* C++ Intellisense
* Code Runner

安装完成后，扩展将会自动激活。此时**重启** VScode。 

重启后，打开 VScode，新建一个文件，输入如下代码： 

```C
#include <iostream>
int main() {
    std::cout << "hello, world\n";
    return 0;
}
```

保存为 `.cpp` 格式文件，点击 **运行** 按钮，观察 "Terminal" 中的成功输出：`hello, world`。至此，配置全部完成。 

 # 常见问题

cannot edit in read-only editor

![](/img/cpp/auto_guess_encoding.png)

GBK to UTF-8 乱码问题：

https://www.cnblogs.com/kingsonfu/p/11010086.html

![](/img/cpp/cannot_edit_in_read-only_editor.png)

# 参考

http://www.cplusplus.com/doc/tutorial/introduction/

[C++ 在线编译器](http://www.dooccn.com/cpp/)

[C++ Insights](https://cppinsights.io/)

[C++ Compiler Explorer](https://gcc.godbolt.org/)