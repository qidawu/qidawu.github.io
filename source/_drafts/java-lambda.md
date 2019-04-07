---
title: Java lambda 表达式的总结
date: 2019-04-05 23:53:12
updated:
tags: Java
---



```java
public void test() {
    ExecutorService executor = Executors.newSingleThreadExecutor();

    // 普通类实现，适用于多个地方复用
    executor.execute(new MyRunnable());

    // 匿名内部类实现，适用于一次性使用，别的地方无法使用该类。匿名内部类的设计目的是为了方便将代码作为数据传递，常用作回调函数
    executor.execute(new Runnable() {
        @Override
        public void run() {
            System.out.println("Hello world");
        }
    });
    
    // lambda 表达式实现，简化了实现匿名内部类的冗余样板代码。和上面两个例子中传入一个实现某接口的对象不同，传入的是一个函数
    executor.execute(() -> System.out.println("Hello world"));
}

class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Hello world");
    }
}
```

