---
layout:     post
title:      "设计模式-单例模式"
subtitle:   "几种单例模式的实现"
date:       2019-08-17 17:10
author:     "LQFGH"
header-img: "img/post-bg-halting.jpg"
header-mask: 0.3
catalog:    true
tags:

  - 设计模式
---







# 单例模式集中实现方式

饿汉式：直接创建对象，不存在线程安全问题

- 直接实例化饿汉式
- 枚举式
- 静态代码块饿汉式（包含复杂实例化代码）

懒汉式：延迟创建对象

- 线程不安全
- 线程安全
- 双检模式
- 静态内部类

------



# 饿汉式

#### 直接实例化饿汉式

```java
public class Singleton1 {

    private final static Singleton1 INSTANT = new Singleton1();

    private Singleton1(){}

    public static Singleton1 getInstance(){
        return INSTANT;
    }
}
```



#### 枚举式

```java
public enum  Singleton2 {
    INSTANT
}
```



#### 静态代码块饿汉式（包含复杂实例化代码）

```java
public class Singleton3 {

    private static Singleton3 INSTANCE ;

    private Singleton3(){}

    static {
        INSTANCE = new Singleton3();
        // 这里可以写一些创建的逻辑
    }
    
    public static Singleton3 getInstance(){
        return INSTANCE;
    }
}
```



------



# 懒汉式

#### 线程不安全

```java
public class SingleTon4 {

    private static SingleTon4 INSTANCE;

    private SingleTon4(){}

    public static SingleTon4 getInstance(){
        if (INSTANCE == null) {
            INSTANCE = new SingleTon4();
        }
        return INSTANCE;
    }
}
```



#### 线程安全

```java
public class SingleTon5 {

    private static SingleTon5 INSTANCE;

    private SingleTon5(){}

    public static SingleTon5 getInstance(){
        synchronized (SingleTon5.class){
            if (INSTANCE == null) {
                INSTANCE = new SingleTon5();
            }
        }
        return INSTANCE;
    }
}

```



#### 双检模式

```java
public class SingleTon5 {

    private static SingleTon5 INSTANCE;

    private SingleTon5(){}

    public static SingleTon5 getInstance(){
        if (INSTANCE == null) {
            synchronized (SingleTon5.class){
                if (INSTANCE == null) {
                    INSTANCE = new SingleTon5();
                }
            }
        }
        return INSTANCE;
    }
}
```



#### 静态内部类

```java
public class Singleton6 {

    private Singleton6(){}

    private static class Inner{
        private final static Singleton6 INSTANCE = new Singleton6();
    }

    public static Singleton6 getInstent(){
        return Inner.INSTANCE;
    }

}
```

