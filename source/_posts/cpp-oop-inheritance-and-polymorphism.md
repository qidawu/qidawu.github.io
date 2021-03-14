---
title: C/C++ 语言系列（九）面向对象编程之继承与多态
date: 2021-02-25 22:07:11
updated: 
tags: [C/C++]
typora-root-url: ..
---

# 继承

```C++
// Single inheritance 单继承
class derived_class: access_specifier base_class

// Multiple inheritance 多重继承，各个基类之间用逗号分隔
class derived_class: access_specifier base_class_1, access_specifier base_class_2, ...
```

## 继承类型

继承类型通过访问修饰符 access_specifier 来指定：

| 继承类型 | 基类的 `public` 成员      | 基类的 `protected` 成员   | 基类的 `private` 成员 |
| --------------------- | ------------------------- | ------------------------- | --------------------- |
| 公有继承（`public`）  | 派生类的 `public` 成员    | 派生类的 `protected` 成员 | 无法继承              |
| 保护继承（`protected`） | 派生类的 `protected` 成员 | 派生类的 `protected` 成员 | 无法继承              |
| 私有继承（`private`） | 派生类的 `private` 成员   | 派生类的 `private` 成员   | 无法继承              |

通常使用 `public` 继承，几乎不使用 `protected` 或 `private` 继承。

> In principle, a publicly derived class inherits access to every member of a base class **except**:
>
> - its constructors and its destructor
> - its assignment operator members (operator=)
> - its friends
> - its private members

## 继承的访问控制属性

| Access                    | `public` | `protected` | `private` |
| :------------------------ | :------- | :---------- | :-------- |
| members of the same class | yes      | yes         | yes       |
| members of derived class  | yes      | yes         | no        |
| not members               | yes      | no          | no        |

## 构造、析构函数执行顺序

> Even though access to the constructors and destructor of the base class is not inherited, they are automatically called by the constructors and destructor of the derived class.
>
> Unless otherwise specified, the constructors of a derived class calls the default constructor of its base classes (i.e., the constructor taking no arguments).

继承后，执行顺序如下：

* 构造函数：先父后子
* 析构函数：先子后父

## 子类调用父类方法

`BaseClass::Function()`

# 多态

> One of the key features of class inheritance is that a pointer to a derived class is type-compatible with a pointer to its base class. *Polymorphism* is the art of taking advantage of this simple but powerful and versatile feature.

类继承的关键特性之一，就是指向派生类的指针与指向其基类的指针在类型上兼容。 多态是利用这一简单但强大而通用的功能的艺术。

下面使用指针来演示多态这一特性。

UML 类图如下：

![UML 类图](/img/cpp/class_inheritance.png)

类声明如下：

```C++
// pointers to base class
#include <iostream>
using namespace std;

class Shape {
  protected:
    int width, height;
  public:
    Shape(int width, int height) {
      this->width = width;
      this->height = height;
    }
    virtual int area() { return 0; }
};

class Rectangle : public Shape {
  public:
    Rectangle(int x, int y) : Shape(x, y) {};
    int area() { return width * height; }
};

class Triangle : public Shape {
  public:
    Triangle(int x, int y) : Shape(x, y) {};
    int area() { return width * height / 2; }
};
```

使用方式一：

```C++
int main() {
  Rectangle rect(2, 3);
  Triangle trgl(2, 3);
  
  Shape * p1 = &rect;
  Shape * p2 = &trgl;

  cout << "rectangle area is " << p1->area() << endl;  // rectangle area is 6
  cout << "triangle area is " << p2->area() << endl;   // triangle area is 3
  
  return 0;
}
```

使用方式二，参考：[Dynamic memory](http://www.cplusplus.com/doc/tutorial/dynamic/)

```C++
int main() {

  Shape * p1 = new Rectangle(2, 3);
  Shape * p2 = new Triangle(2, 3);

  cout << "rectangle area is " << p1->area() << endl;  // rectangle area is 6
  cout << "triangle area is " << p2->area() << endl;   // triangle area is 3

  delete p3;
  delete p4;
  
  return 0;
}
```

描述如下：

> Function `main` declares two pointers to `Shape` (named `p1` and `p2`). These are assigned the addresses of `rect` and `trgl`, respectively, which are objects of type `Rectangle` and `Triangle`. Such assignments are valid, since both `Rectangle` and `Triangle` are classes derived from `Shape`.
>
> Dereferencing `p1` and `p2` (with `p1->` and `p2->`) is valid and allows us to access the members of their pointed objects. For example, the following two statements would be equivalent in the previous example:
>
> ```C++
> rect.area();
> p1->area();
> ```

## 虚函数（virtual）

虚函数是在基类中使用 `virtual` 关键字声明的函数，是可以在派生类中**重定义**的成员函数。虚函数用于实现**运行时多态**：

```
+--------------------+
| Base Class         |
|   virtual function |
+---------^----------+
          |
          |class inheritance
          |
+---------+----------+
| Derived class      |
|  redefined function|
+--------------------+

运行时多态的实现手段：虚函数 + 继承 + 函数重定义
```

派生类也可以重定义基类的非虚函数，但无法通过基类的引用来访问派生类的该函数。即：如果移除上述基类中 `area` 函数声明的 `virtual` 关键字，下面的函数调用将返回 0，因为实际调用的是基类的版本：

```C++
  cout << "rectangle area is " << p1->area() << endl;  // rectangle area is 0
  cout << "triangle area is " << p2->area() << endl;   // triangle area is 0
```

> Therefore, essentially, what the `virtual` keyword does is to allow a member of a derived class with the same name as one in the base class to be appropriately called from a pointer, and more precisely when the type of the pointer is a pointer to the base class that is pointing to an object of the derived class, as in the above example.
>
> A class that declares or inherits a virtual function is called a *polymorphic class*.

注意，尽管成员之一是 `virtual` 的，但 `Sharp` 仍然是一个常规类，可以实例化对象。

## 纯虚函数（抽象类）

Classes that contain at least one *pure virtual function* are known as *abstract base classes*. The syntax of *pure virtual function* is to replace their definition by `=0` (an equal sign and a zero):

```C++
// abstract class CPolygon
class Shape {
  protected:
    int width, height;
  public:
    virtual int area () = 0;
};
```

Abstract base classes cannot be used to instantiate objects:

```C++
// 不允许使用抽象类类型 "Shape" 的对象: -- 函数 "Shape::area" 是纯虚函数
// variable type 'Shape' is an abstract class
Shape shape;
```

But an *abstract base class* is not totally useless. It can be used to create pointers to it, and take advantage of all its polymorphic abilities.

```C++
// the following pointer declarations would be valid
Shape * p1 = &rect;
Shape * p2 = &trgl;
```

Virtual members and abstract classes grant C++ polymorphic characteristics, most useful for object-oriented projects.

C++ 接口是使用**抽象类**来实现的。

# 参考

http://www.cplusplus.com/doc/tutorial/inheritance/

http://www.cplusplus.com/doc/tutorial/polymorphism/

