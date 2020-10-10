---
layout: post
title: "反射"
date: 2018-09-15 00:00:00.000000000 +00:00
tags: java基础
---

## 1 概念

**静态加载与动态加载**

编译时刻加载类是静态加载类，运行时刻加载类是动态加载类

new创建对象时，是静态加载类，在编译时刻就要加载所有可能用到的类

可以通过class.forName()动态加载类，代码如下：

```swift
Class c = class.forName("全限定类名");
通过类类型，创建该类对象
A a = (A) c.newInstance();
```
在设计时，对于功能性的类，尽量使用动态加载。



## 2 java集合泛型的本质

Java中集合的泛型，是防止错误输入的，只在编译阶段有效，绕过编译就无效了.

举例代码如下：

```swift
ArrayList list1 = new ArrayList();
ArrayList<String> list2 = new ArrayList<String>();
list2.add("test");
list2.add(2);//添加int类型会报错

//反射的操作都是都是程序编译之后的操作
Class class1 = list1.getClass();
Class class2 = list2.getClass();
System.out.println(class1 == class2); //返回true
```
上述结果返回`true`说明：java集合在编译时去泛型化的

可以通过反射的方法像集合list2中添加非String类型的元素，代码如下：

```swift
try {
    Method m = class2.getMethod("add", Object.class);
    m.invoke(list2, 100);
    System.out.println(list2);//返回 [test, 100]

} catch (Exception e) {
    e.printStackTrace();
}
```



