---
layout: post
title: "Java中toString()方法的使用"
date: 2017-11-20 20:30:00.000000000 +00:00
tags: 他山之石
---

### 1 Java中toString()方法的来源

toString()方法来自java.lang包中的Object类，由于该类是所有类的父类，所以每新建一个java类都会拥有toString()方法,
该方法的作用是返回一个代表对象的字符串。当子类没有重写该方法时，便会采用父类中的实现。

### 2 toString()方法的使用

在编写java程序的过程中，我们总是喜欢用System.out.println()方法打印某个对象或是关键变量，本质上该方法会调用
toString()方法，当子类未重写该方法时，此时该方法的输出，如下：

```swift
/*新建Student*/
package com.paTech;
public class Student {
	String name;
	String gender;
	int age;
}

/*主程序，main方法中打印该类的实例*/
package com.paTech;
public class MainApp {
	public static void main(String[] args){
		Student s1 = new Student();
		System.out.println(s1);
	}
}

输出结果：com.paTech.Student@1db9742
```
乍一看这个结果，并不明白是什么意思，通过查询JDK API文档，可以看到有如下这段话:

`The toString method for class Object returns a string consisting of the name of the class of which the　object is an instance, the at-sign character ＇@＇, and the unsigned hexadecimal representation of the　hash code of the object. In other words, this method returns a string equal to the value of:`

`getClass().getName() + '@' + Integer.toHexString(hashCode())`

看完就明白了，上边的结果，即类名@hashCode，往往这样的结果并没有实际的意义，所以，我们需要重写toString()方法。

### 3 重写toString()方法

在Student类中重写该方法，如下：
```swift
public class Student {
	String name;
	String gender;
	int age;

	@Override
	public String toString() {
		return "重写之后的toString()方法";
	}
}
```
再在主程序中打印输出结果，便得到`重写之后的toString()方法`

当我们在主程序中尝试打印输出`System.out.println("test")`，便会得到`test`，这是怎么回事呢？为什么输出的就是test
原因是java中的String类已经重写了toString()方法，查阅JDK API 文档，String类的方法已包括toString()，如下图：

![](/assets/images/2017/String_Class.PNG)