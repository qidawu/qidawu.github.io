---
title: C/C++ 语言系列（三）函数总结
date: 2021-02-04 22:07:11
updated: 
tags: [C/C++]
typora-root-url: ..
---

# 函数定义

函数定义形式如下：

* 函数头

  * 返回类型

  * 函数名称

  * 形式参数
    
    * 无参数函数
    
    * 有参数函数
    
      * 参数默认值（必须从右到左赋默认值）
    
        ```C
        // 函数默认值，必须从右到左
        int max2(int num1, int num2 = 100)
        {
          return num1 > num2 ? num1 : num2;
        }
        
        // 否则报编译错误：missing default argument on parameter 'num2'
        int max3(int num1 = 100, int num2)
        {
          return num1 > num2 ? num1 : num2;
        }
        ```

* 函数体

# 函数声明

函数必须**先声明后使用**：

```C
#include <iostream>
using namespace std;

// 函数必须先声明后使用
int max1(int num1, int num2);
 
int main()
{
  cout << "result is " << max1(1, 10) << endl;  // 否则报错：未定义标识符 use of undeclared identifier 'max1'
  return 0;
}

int max1(int num1, int num2)
{
  return num1 > num2 ? num1 : num2;
}
```

# 函数参数

函数的实际参数有三种传递方式：

| 调用类型 | 调用类型                                                     | 例子                          | 描述                                                         |
| -------- | :----------------------------------------------------------- | ----------------------------- | :----------------------------------------------------------- |
| 传值     | [传值调用](https://www.runoob.com/cplusplus/cpp-function-call-by-value.html) | `void swap(int x, y)`         | 把实际参数的实际值**复制**一份给形式参数。修改函数内的形式参数**对实际参数没有影响**。 |
| 传址     | [指针调用](https://www.runoob.com/cplusplus/cpp-function-call-by-pointer.html) | `void swap(int * x, int * y)` | 把实际参数的**地址**赋值给形式参数。在函数内，该指针用于访问实际参数的**地址**。这意味着，修改形式参数**会影响实际参数**。 |
| 传引用   | [引用调用](https://www.runoob.com/cplusplus/cpp-function-call-by-reference.html) | `void swap(int &x, &y)`       | 把实际参数的引用赋值给形式参数。在函数内，该引用作为实际参数的**别名**。这意味着，修改形式参数**会影响实际参数**。 |

传址与传引用的使用区别，如下：

![指针与引用的使用区别](/img/cpp/compare_with_pointer_and_reference.png)

# 函数调用

函数的调用方式：

* 嵌套调用
* 递归调用（直接递归， 间接递归）

# 参考

http://www.cplusplus.com/doc/tutorial/functions/