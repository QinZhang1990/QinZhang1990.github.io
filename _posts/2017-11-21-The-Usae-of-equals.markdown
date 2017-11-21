---
layout: post
title: "Java中equals方法的使用"
date: 2017-11-21 22:00:00.000000000 +00:00
tags: 他山之石
---

### 1 equals(Object obj)方法的来源

equals(Object obj)方法定义在java.lang.Object类中，由于Object类是所有java类的父类,所以任何一个子类都会继承该
方法。

### 2 两个对象的比较

新建Student类,如下：
```swift
package com.paTech;
public class Student {
	String name;
	String gender;
	int age;

	public Student() {
		super();
	}
	public Student(String name, String gender, int age) {
		this.name = name;
		this.gender = gender;
		this.age = age;
	}

	@Override
	public String toString() {
		return "Student [name=" + name + ", gender=" + gender + ", age=" + age + "]";
	}
}
```
实例化，两个Student的实例,s1和s2，并进行比较，如下:
```swift
public class MainApp {
	public static void main(String[] args){
		Student s1 = new Student("zhangsan","male", 20);
        Student s2 = new Student("zhangsan","male", 20);
		System.out.println(s1 == s2);
	}
}
```
最后，返回的结果为: `false`

两个对象的内容明明就是一样的，为什么返回的却是false，原因就在于`==`比较的是引用地址，而非对象的内容，如下图：

![](/assets/images/2017/reference_compare.PNG)

从图中可以看出，s1和s2作为对象的引用，分别指向不同的地址，所以永远都会相等。而要比较内容是否相等，应该采用equals
方法。

### 2 equals(Object obj)方法的使用

当我们用equals方法代替上述的==时，如下代码:
```swift
public class MainApp {
	public static void main(String[] args){
		Student s1 = new Student("zhangsan","male", 20);
		Student s2 = new Student("zhangsan","male", 20);
		System.out.println(s1.equals(s2));
	}
}
```
返回的结果依然是`false`，这又是为何呢？

查阅JDK API文档中java.lang.Object中对equals方法的定义，能找到如下一段描述：

`The equals method for class Object implements the most discriminating possible equivalence relation on objects; that is, for any non-null reference values x and y, this method returns true if and only if x and y refer to the same object (x == y has the value true).`

当且仅当引用x和y指向同一个对象时，Object类中的equals方法才会返回true，即当子类中没有对equals方法进行重写时，`x.equals(y)`，
本质上和`x==y`是一样的，所以，结果再次返回false，也就不足为奇了。

### 3 重写equals方法

在子类Student中重写equals方法，如下代码：
```swift
public class Student {
	String name;
	String gender;
	int age;

	public Student() {
		super();
	}
	public Student(String name, String gender, int age) {
		this.name = name;
		this.gender = gender;
		this.age = age;
	}

	@Override
	public String toString() {
		return "Student [name=" + name + ", gender=" + gender + ", age=" + age + "]";
	}

	@Override
	public boolean equals(Object obj) {
		if(obj == null) return false;
		else{
			if(obj instanceof Student){
				Student s = (Student) obj;
				if(s.name==this.name && s.gender==this.gender && s.age==this.age){
					return true;
				}
			}
		}
		return false;
	}
}
```
重写equals方法之后，再次运行结果返回`true`



