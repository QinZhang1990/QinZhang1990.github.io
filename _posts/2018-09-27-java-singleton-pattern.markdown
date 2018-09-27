---
layout: post
title: "java基础之单例模式"
date: 2018-09-15 00:00:00.000000000 +00:00
tags: java基础
---

## 1 概念

`单例模式`：确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例类，它提供全局访问的方法。单例模式是
一种对象创建型模式。

## 2 特征

+ 单例类只能有一个实例
+ 单例类必须自行创建自己的唯一的实例
+ 单例类必须给所有其他对象提供这一实例

## 3 类型

单例模式有两种类型，即`饿汉模式`和 `懒汉模式`

### 饿汉模式

```swift
public class Singleton1 {
	//构造方法私有化，不允许外部直接创建对象
	private Singleton1(){}

	//创建类的唯一实例
	private static Singleton1 instance  = new Singleton1();

	//提供一个用于获取实例的方法
	public static Singleton1 getInstance(){
		return instance;
	}
	//饿汉模式：当类被加载时，就会去创建类的唯一实例
}
```

### 懒汉模式

```swift
public class Singleton2 {
	//构造方法私有化，不允许外部直接创建对象
    private Singleton2(){}

    //声明类的唯一实例
    private static Singleton2 instance;

    //提供一个用于获取实例的方法
    public static Singleton2 getInstance(){
        if(instance == null){
            instance = new Singleton2();
        }
        return instance;
    }
    //懒汉模式：当类被加载时，并没有创建实例，而是用户在调用时才创建实例，所以，形象称之为懒汉模式
}
```

### 两种模式的对比

`饿汉模式`： 加载类时比较慢，但运行时获取对象的速度比较快，且线程安全。

`懒汉模式`： 加载类时比较快，但运行时获取对象的速度比较慢，且线程不安全。

