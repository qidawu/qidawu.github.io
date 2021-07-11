---
title: C/C++ 语言系列（九）面向对象编程之类、类模板、类指针
date: 2021-02-20 22:07:11
updated: 
tags: [C/C++]
typora-root-url: ..
---

# 类

类的声明：

```C++
class class_name {
  access_specifier_1:
    member1;
  access_specifier_2:
    member2;
  ...
} object_names;
```

![](/img/cpp/cpp-classes-objects.png)

## 访问修饰符

- `private` members of a class are accessible only from within other members of the same class (or from their `friend`). By default, all members of a class have private access for all its members.
- `protected` members are accessible from other members of the same class (or from their `friend`), but also from members of their derived classes.
- Finally, `public` members are accessible from anywhere where the object is visible.

## this 指针

在 C++ 中，每一个对象都能通过 `this` 指针来访问自己的地址。`this` 指针是所有成员函数的隐含参数。因此，在成员函数内部，它可以用来指向调用对象。

友元函数没有 `this` 指针，因为友元不是类的成员。只有成员函数才有 `this` 指针。

## 静态成员（static）

![](/img/cpp/cpp-static-members.png)

`static` 关键字用于修饰静态成员变量或函数。限制如下：

* 无法访问类的非静态成员变量或函数；
* 无法使用 `this` 指针。

静态成员的引用方式：`Runoob:runoob_age`

## 成员函数

有两种方式定义类的成员函数：

* 内联成员函数（*inline* member function）
* 普通成员函数（not-inline member function）

两种方式并不会导致行为上的差异，而只会导致可能的编译器优化。

```C++
// classes example
#include <iostream>
using namespace std;

class Rectangle {
    int width, height;
  public:
    // declaration of a member function within the class
    void set_values (int, int);
    // defining a member function completely within the class definition
    int area() {return width*height;}
};

// definition of a member function of a class outside the class itself.
// The scope operator (::) specifies the class to which the member being defined belongs, granting exactly the same scope properties as if this function definition was directly included within the class definition.
void Rectangle::set_values (int x, int y) {
  width = x;
  height = y;
};

int main () {
  Rectangle rect;

  // public members of object can be accessed by dot operator (.)
  rect.set_values (3,4);
  cout << "area: " << rect.area() << endl;
  return 0;
}
```

### 常成员函数

To specify that a member is a `const` member, the `const` keyword shall follow the function prototype, after the closing parenthesis for its parameters:

```C++
int get() const {return x;}        // const member function
```

## 构造函数

类的**构造函数**是类的一种特殊的成员函数，构造函数的名称与类的名称是完全相同的，并且不会返回任何类型，也不会返回` void`。它会在每次创建类的新对象时执行：

```C++
  // 调用有参构造函数
  Rectangle rect(1, 2);	// Object is being created, width=1, height=2
  // 调用默认构造函数
  Rectangle rectb;      // Object is being created
```

### 构造函数重载

> Overloading constructors
>
> Like any other function, a constructor can also be overloaded with different versions taking different parameters: with a different number of parameters and/or parameters of different types. The compiler will automatically call the one whose parameters match the arguments:

```C++
class Rectangle {
    int width, height;
  public:
    // 声明一个无参构造函数
    Rectangle();
    // 声明一个有参构造函数
    Rectangle(int, int);
};

// 定义一个无参构造函数
Rectangle::Rectangle() {
  cout << "Object is being created" << endl;
}

// 定义一个有参构造函数
Rectangle::Rectangle(int width, int height) {
  this->width = width;
  this->height = height;
  cout << "Object is being created, width=" << this->width << ", height=" << this->height << endl;
};
```

> This example introduces a special kind constructor: the *default constructor*. The *default constructor* is the constructor that takes no parameters, and it is special because it is called when an object is declared but is not initialized with any arguments. In the example above, the *default constructor* is called for `rectb`. Note how `rectb` is not even constructed with an empty set of parentheses - in fact, empty parentheses cannot be used to call the default constructor:
>
> ```C++
> Rectangle rectb;   // ok, default constructor called
> Rectangle rectc(); // oops, default constructor NOT called, empty parentheses interpreted as a function declaration
> ```
>
> This is because the empty set of parentheses would make of `rectc` a function declaration instead of an object declaration: It would be a function that takes no arguments and returns a value of type `Rectangle`.

### 在构造函数中初始化成员变量

使用构造函数初始化其他成员变量时，有下面两种方式：

> Member initialization in constructors
>
> When a constructor is used to initialize other members, these other members can be initialized directly, without resorting to statements in its body. This is done by inserting, before the constructor's body, a colon (`:`) and a list of initializations for class members. For example, consider a class with the following declaration:
>
> ```C++
> class Rectangle {
>  int width, height;
> public:
>  Rectangle(int, int);
>  int area() {return width*height;}
> };
> ```
>
> The constructor for this class could be defined, as usual, as:
>
> ```C++
> Rectangle::Rectangle(int x, int y) { width=x; height=y; }
> ```
>
> But it could also be defined using *member initialization* as:
>
> ```C++
> Rectangle::Rectangle(int x, int y) : width(x), height(y) { }
> ```
>
> Or even:
>
> ```C++
> Rectangle::Rectangle(int x, int y) : Shape(x, y) { }
> ```
>
> Note how in this last case, the constructor does nothing else than initialize its members, hence it has an empty function body.

## 析构函数（~）

类的**析构函数**是类的一种特殊的成员函数，它会在每次删除对象时执行，有助于在跳出程序前释放资源（比如关闭文件、释放内存等）。

```C++
class Rectangle {
    int width, height;
  public:
    // 构造函数
    Rectangle();
    // 析构函数
    ~Rectangle();
};

Rectangle::Rectangle(){
  cout << "Object is being created" << endl;
};

Rectangle::~Rectangle(){
  cout << "Object is being deleted" << endl;
};
```

析构函数要点：

* 析构函数名称与类的名称完全相同，前缀使用关键字 `~`；
* 一个类中只能声明一个析构函数（destructor cannot be redeclared）；
* 析构函数无参数（destructor cannot have any parameters）；
* 析构函数无返回值（destructor cannot have a return type）；
* 不可重载。

## 友元函数（friend）

类的友元函数（`friend` 关键字），有权访问类的所有私有（`private`）和保护（`protected`）成员变量。尽管友元函数在类中声明，但是**友元函数并不是类的成员函数**。

```C++
class Rectangle {
    int width, height;
  public:
    friend int getWidth(Rectangle);
    friend int getHeight(Rectangle rect) {
      return rect.height;
    }
};

int getWidth(Rectangle rect) {
  return rect.width;
}

int main () {
  Rectangle rect1(1, 2);

  cout << "width: " << getWidth(rect1) << " height: " << getHeight(rect1) << endl;  // width: 1 height: 2
  return 0;
}
```

友元函数破坏了类的封装性，实践中不建议使用。

## 重载（overload）

### 函数重载

可以声明几个功能类似的同名函数，但是这些同名函数的形式参数（指参数的个数、类型或者顺序）必须不同。

### 运算符重载

一、支持重载的运算符：

C++ allows most operators to be overloaded so that their behavior can be defined for just about any type, including classes. Here is a list of all the operators that can be overloaded:

```
+    -    *    /    =    <    >    +=   -=   *=   /=   <<   >>
<<=  >>=  ==   !=   <=   >=   ++   --   %    &    ^    !    |
~    &=   ^=   |=   &&   ||   %=   []   ()   ,    ->*  ->   new 
delete    new[]     delete[]
```

Operators are overloaded by means of `operator` functions, which are regular functions with special names: their name begins by the `operator` keyword followed by the *operator sign* that is overloaded. The syntax is: 

```C++
type operator sign (parameters) { /*... body ...*/ }
```

二、重载运算符的不同形式：

There is a table with a summary of the parameters needed for each of the different operators than can be overloaded (please, replace `@` by the operator in each case):

| Expression  | Operator                                           | Member function         | Non-member function |
| ----------- | -------------------------------------------------- | ----------------------- | ------------------- |
| `@a`        | `+ - * & ! ~ ++ --`                                | `A::operator@()`        | `operator@(A)`      |
| `a@`        | `++ --`                                            | `A::operator@(int)`     | `operator@(A,int)`  |
| `a@b`       | `+ - * / % ^ & \| < > == != <= >= << >> && \|\| ,` | `A::operator@(B)`       | `operator@(A,B)`    |
| `a@b`       | `= += -= *= /= %= ^= &= \|= <<= >>= []`            | `A::operator@(B)`       | -                   |
| `a(b,c...)` | `()`                                               | `A::operator()(B,C...)` | -                   |
| `a->b`      | `->`                                               | `A::operator->()`       | -                   |
| `(TYPE) a`  | `TYPE`                                             | `A::operator TYPE()`    | -                   |

Where `a` is an object of class `A`, `b` is an object of class `B` and `c` is an object of class `C`. `TYPE` is just any type (that operators overloads the conversion to type `TYPE`).

Notice that some operators may be overloaded in two forms: either as a member function or as a non-member function.

三、例子：

For example, *cartesian vectors* are sets of two coordinates: `x` and `y`. The addition operation of two *cartesian vectors* is defined as the addition both `x` coordinates together, and both `y` coordinates together. For example, adding the *cartesian vectors* `(3,1)` and `(1,2)` together would result in `(3+1,1+2) = (4,3)`. This could be implemented in C++ with the following code:

![Overloading operators](/img/cpp/overloading_operators.png)

The function `operator+` of class `CVector` overloads the addition operator (`+`) for that type. Once declared, this function can be called either implicitly using the operator, or explicitly using its functional name:

```C++
// called either implicitly using the operator
c = a + b;

// or explicitly using its functional name
c = a.operator+ (b);
```

Both expressions are equivalent.

四、注意点：

> Attention
>
> The operator overloads are just regular functions which can have any behavior; there is actually no requirement that the operation performed by that overload bears a relation to the mathematical or usual meaning of the operator, although it is strongly recommended. For example, a class that overloads `operator+` to actually subtract or that overloads `operator==` to fill the object with zeros, is perfectly valid, although using such a class could be challenging.

## 函数重定义（redefine）

即 Java 语言中的方法重写（rewrite）。

# 类指针

Objects can also be pointed to by pointers: Once declared, a class becomes a valid type, so it can be used as the type pointed to by a pointer. For example:

```C++
// a pointer to an object of class Rectangle.
Rectangle * prect;
```

Similarly as with plain data structures, the members of an object can be accessed directly from a pointer by using the arrow operator (`->`). Here is an example with some possible combinations:

```C++
// pointer to classes example
#include <iostream>
using namespace std;

class Rectangle {
  int width, height;
public:
  Rectangle(int x, int y) : width(x), height(y) {}
  int area(void) { return width * height; }
};

int main() {
  Rectangle obj(3, 4);
  Rectangle * foo, * bar;  // 类指针（Pointers to classes）
  foo = &obj;
  bar = new Rectangle (5, 6);  // 参考：http://www.cplusplus.com/doc/tutorial/dynamic/
  cout << "obj's area: " << obj.area() << '\n';
  cout << "*foo's area: " << foo->area() << '\n';
  cout << "*foo's area: " << (*foo).area() << '\n';
  cout << "*bar's area: " << bar->area() << '\n';    
  cout << "*bar's area: " << (*bar).area() << '\n';    
  delete bar;
  return 0;
}	
```

This example makes use of several operators to operate on objects and pointers (operators `*`, `&`, `.`, `->`). They can be interpreted as:

| expression | can be read as                                               |
| ---------- | ------------------------------------------------------------ |
| `*x`       | pointed to by `x`                                            |
| `&x`       | address of `x`                                               |
| `x.y`      | member `y` of object `x`                                     |
| `x->y`     | member `y` of object pointed to by `x`                       |
| `(*x).y`   | member `y` of object pointed to by `x` (equivalent to the previous one) |

# 模板

模板是**泛型**编程的基础，泛型编程即以一种独立于任何特定类型的方式编写代码。

## 函数模板

```C++
// function templates
#include <iostream>
using namespace std;

template <typename T>
T getmax (T a, T b)
{
  T retval;
  retval = a > b ? a : b;
  return retval;
}

int main () {
  int maxInt = getmax(100, 75);
  cout << maxInt << endl; // 100

  double maxDouble = getmax(3.3, 2.18);
  cout << maxDouble << endl;  // 3.3

  // no matching function for call to 'getmax'
  // char maxChar = getmax('a', 1.99);
  // cout << maxChar << endl;

  return 0;
}
```

## 类模板

Just like we can create *function templates*, we can also create *class templates*, allowing classes to have members that use *template parameters* as types. For example:

```c++
// class templates
#include <iostream>
using namespace std;

template <class T>
class MyPair {
    T a, b;
  public:
    MyPair (T first, T second)
      {a=first; b=second;}
    T getmax ();
};

// In case that a member function is defined outside the defintion of the class template, it shall be preceded with the template <...> prefix
template <class T>
T MyPair<T>::getmax ()
{
  T retval;
  retval = a > b ? a : b;
  return retval;
}

int main () {
  MyPair<int> myobject(100, 75);
  cout << myobject.getmax() << endl;  // 100

  MyPair<double> myfloats(3.3, 2.18);
  cout << myfloats.getmax() << endl;  // 3.3

  // implicit conversion from 'double' to 'char' changes value from 1.99 to 1
  // MyPair<char> mychars('a', 1.99);
  // cout << mychars.getmax() << endl;

  return 0;
}
```

Notice the syntax of the definition of member function `getmax`:

```C++
template <class T>
T mypair<T>::getmax () 
```

There are three `T`'s in this declaration: The first one is the template parameter. The second `T` refers to the type returned by the function. And the third `T` (the one between angle brackets) is also a requirement: It specifies that this function's template parameter is also the class template parameter.

## 模板类

模板类是类模板实例化后的一个产物。

It is possible to define a different implementation for a template when a specific type is passed as template argument. This is called a *template specialization*.

For example:

```C++
// template specialization
#include <iostream>
using namespace std;

// class template:
template <class T>
class mycontainer {
    T element;
  public:
    mycontainer (T arg) {element=arg;}
    T increase () {return ++element;}
};

// class template specialization:
template <>
class mycontainer <char> {
    char element;
  public:
    mycontainer (char arg) {element=arg;}
    char uppercase ()
    {
      if ((element>='a')&&(element<='z'))
      element+='A'-'a';
      return element;
    }
};

int main () {
  mycontainer<int> myint (7);
  mycontainer<char> mychar ('j');
  cout << myint.increase() << endl;    // 8
  cout << mychar.uppercase() << endl;  // J
  return 0;
}
```

This is the syntax used for the class template specialization:

```C++
template <>
class mycontainer <char> { ... };
```

First of all, notice that we precede the class name with `template<>` , including an empty parameter list. This is because all types are known and no template arguments are required for this specialization, but still, it is the specialization of a class template, and thus it requires to be noted as such.

But more important than this prefix, is the `<char>` specialization parameter after the class template name. This specialization parameter itself identifies the type for which the template class is being specialized (`char`). Notice the differences between the generic class template and the specialization:

```C++
template <class T> class mycontainer { ... };
template <> class mycontainer <char> { ... };
```

The first line is the *generic template*, and the second one is the *specialization*.

When we declare specializations for a template class, we must also define all its members, even those identical to the generic template class, because there is no "inheritance" of members from the generic template to the specialization.

# 参考

http://www.cplusplus.com/doc/tutorial/classes/

http://www.cplusplus.com/doc/tutorial/templates/

http://www.cplusplus.com/doc/tutorial/classes2/

https://www.cplusplus.com/doc/oldtutorial/templates/

https://www.runoob.com/cplusplus/cpp-classes-objects.html

https://www.runoob.com/cplusplus/cpp-templates.html