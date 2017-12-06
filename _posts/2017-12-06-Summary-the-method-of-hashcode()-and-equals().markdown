---
layout: post
title: "Hashcode()和Equals()方法的总结"
date: 2017-12-06 22:15:00.000000000 +00:00
tags: 他山之石
---

### hashcode()方法存在的原因

    hashcode()和equals()均属于基类Object中的方法，所以java中任意一个类都会从Object基类中继承这两个方法。然而，这两个方法在java
容器类，即Set、List和Map，中有着十分重要的应用，其中Set类型的容器，要求容器中不能包含有重复的元素，那么这就要求再对set类型的
容器添加元素时，要检查容器中是否存在该元素，那么问题来了，如何检查呢？很简单，即通过比较，将容器中已有的元素逐个同待加入的元素
进行比较，比较的方法也是现成的，即采用**重写**之后的equals()方法。通过x.equals(y),判断x是否等于y。这种方法的缺陷却也十分明显
，当元素较少时可以采用逐个比较，但当容器中有多个元素，比如10000个，那么难道要调用10000次equals()方法吗？毫无疑问，效率低下！！！
那该如何解决这种问题呢？这时hashcode()的作用就凸显出来了。

       java中采用哈希算法来存储hashset和hashmap，哈希算法对于集合元素的存储，定位查找效率十分高，及存储时，会对每一个元素调用
hashcode()方法，计算出哈希值，然后将该元素存储在其哈希值所对应的区域，这样当加入新的元素时先检查其哈希值对应的存储区域是否已经
存在元素，若没有就把该元素放进来；若有，再调用equals()方法，判断二者是否相等，这样只需要调用一次equals()方法，大大地提高存储效率。

### hashcode()和equals()的相爱相杀

（1）不同的对象可能具有相同的hashcode

Java API中，Object的hashcode()方法描述如下:
```swift
1) If two objects are equal according to the equals(Object) method, then calling the hashCode method on each of the two
   objects must produce the same integer result.

2) It is not required that if two objects are unequal according to the equals(java.lang.Object) method, then calling the
   hashCode method on each of the two objects must produce distinct integer results. However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables.
```
第二点，Java官方文档明确地说明——不要求两个不同的对象在调用hashcode()方法时，一定返回不同的hashcode值，即两个不同的对象可能拥有
相同的hashcode值，为什么呢？原因我理解的也不是很深入，有待深究，一种解释是因为Java的hash值生成机制有问题。

第一点，就很明确地规定了，两个对象equals()相等时，那么他们一定具有相等的hashcode的值，那么反过来，当两个对象的hashcode值不等时
那么这两个对象一定是不等的，可以用字典来形象解释这种情景。

hashcode就是字典里前几页的索引项，而equals(）是索引对应的内容，相同的两个词组必定对应着同一个索引，而同一个索引未必对应的相同
词组。如`自`作为索引时，其对应的词组可以是在同一页中的`自己`和`自发`。

（2） 重写equals()方法，必须重写hashcode()方法

当一个对象调用为重写的hashcode(）方法，返回的结果本质上是这个对象在堆内存中存放的地址，而调用未重写的equals()方法，本质上是在
调用 `==`，本质上还是返回对象的内存地址，由于java中new出来的对象都会分配不同的内存地址，所以当对象调用未重写的equals()方法时，
永远只会返回false。所以，要比较两个对象是否相等，就要重写equals()方法，来比较两个对象的内容是否相等。此时，如果不重写hashcode()
方法，那么调用此方法时必定返回false，这样就会出现两个对象equals相等，而hashcode()的却不相等的情况，这是违反了java的规定。

### 关于哈希算法的更进一步研究

