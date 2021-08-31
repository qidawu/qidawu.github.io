---
title: C/C++ 语言系列（七）复合数据类型之结构体
date: 2021-02-16 22:07:11
updated: 
tags: [C/C++]
typora-root-url: ..
---

# 声明语法

```C
  struct type_name {
    member_type1 member_name1;
    member_type2 member_name2;
    member_type3 member_name3;
    .
    .
  } object_names;
```

# 定义用法

## 结构体对象

```C
  // declare three objects (variables) of this type (product): apple, banana, and melon.
  struct product {
    int weight;
    double price;
  } apple, banana, melon;

  // struct requires either a type_name or at least one name in object_names, but not necessarily both.
  struct {
    int weight;
    double price;
  } apple, banana, melon;

  // declare three objects (variables) of this type (product): apple, banana, and melon.
  product apple, banana, melon;
```

## 结构体数组

```C
  // because structures are types, they can also be used as the type of arrays.
  product banana[3];
```

## 结构体指针

```C
  product * p = &apple;
```
创建结构体指针之后，可以使用以下运算符访问其成员变量：

| Operator                                         | Expression | What is evaluated                       | Equivalent |
| ------------------------------------------------ | ---------- | --------------------------------------- | ---------- |
| dot operator (`.`)                               | `a.b`      | Member `b` of object `a`                |            |
| arrow operator (`->`)<br/>(dereference operator) | `a->b`     | Member `b` of object pointed to by  `a` | `(*a).b`   |

例子：

```C++
  apple.weight;  // 3
  // The arrow operator (->) is a dereference operator that is used exclusively with pointers to objects that have members. This operator serves to access the member of an object directly from its address.
  p->weight;     // 3
  // equivalent to:
  (*p).weight;   // 3
```

例子 2：

```C++
#include <iostream>
using namespace std;

struct product {
  int weight;
  double price;
};

void test() {
  product apple;
  apple.weight = 3;
  apple.price = 4;
  product * p_apple = &apple;

  cout << "weight: " << (*p_apple).weight << endl;                // weight: 3
  cout << "weight: " << p_apple->weight << endl;                  // weight: 3

  cout << "address of apple: " << p_apple << endl;                // address of apple: 0x7ffee256b5b0
  cout << "address of weight: " << &p_apple->weight << endl;      // address of weight: 0x7ffee256b5b0
  cout << "address of price: " << &p_apple->price << endl;        // address of price: 0x7ffee256b5b8
  cout << "size of weight: " << sizeof(p_apple->weight) << endl;  // size of weight: 4
  cout << "size of price: " << sizeof(p_apple->price) << endl;    // size of price: 8
}

int main() {
  test();
  return 0;
}
```

# 结构体作为函数参数

支持三种方式的传参：

* 传值
* 传引用
* 传址

```C++
#include <iostream>
using namespace std;

struct product {
  int weight;
  double price;
} lemon;

// 传值
void show(product prd) {
  cout << "weight: " << prd.weight << " price: " << prd.price << endl;
}

// 传引用
void show2(product &prd) {
  cout << "weight: " << prd.weight << " price: " << prd.price << endl;
}

// 传址
void show3(product * prd) {
  cout << "weight: " << prd->weight << " price: " << prd->price << endl;
}

void test() {
  lemon.weight = 1;
  lemon.price = 2;
  show(lemon);    // weight: 1 price: 2
  show2(lemon);   // weight: 1 price: 2
  show3(&lemon);  // weight: 1 price: 2
}

int main() {
  test();
  return 0;
}
```

# 参考

http://www.cplusplus.com/doc/tutorial/structures/