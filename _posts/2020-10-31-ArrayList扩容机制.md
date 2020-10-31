---
layout: post
title: "ArrayList扩容机制"
date: 2020-10-31
tags: 源码学习
---

对于使用Java开发的同学都知道数组Array是固定长度，而List是可变长度的对象。事实上，List底层也是通过数组Array来实现的，List的动态扩容本质上就是通过创建特定长度的新数组，将老数组中的元素逐个copy过来实现的。那么ArrayList作为List的子类自然是一脉相承，那么ArrayList底层是如何动态扩容的呢？通过阅读源码便能揭开其庐山真面目。

# 1 构造器

ArrayList的底层事先定义了如下几个成员变量

```java
	/**
     * 默认初始大小为10
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 空数组实例
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * 还是空数组实例，
     * 那和EMPTY_ELEMENTDATA的区别在哪呢？该空数组实例是用于无参构造器中赋值给elementData，
     * 后续代码遇到该实例便会给elementData赋予初始默认长度10。是在jdk1.8引入的优化，1.7及之前
     * 只有EMPTY_ELEMENTDATA，优化之后，对于空的ArrayList不用在重新创建空数组了，只需要都指向			 * EMPTY_ELEMENTDATA即可
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * 存储ArrayList的元素，不可序列化
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * size标识ArrayList的元素个数，length标识数组的长度
     */
    private int size;
```

```java
 /**
    * 有参构造器
    * initialCapacity 传入初始大小，为0时，赋EMPTY_ELEMENTDATA空数组
    */
  public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     * 无参构造器，赋值DEFAULTCAPACITY_EMPTY_ELEMENTDATA，默认大小10
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 有参构造器，接受类型E的任意子类型？的集合对象
     * 
     */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```



# 2 扩容机制

ArrayList通过add方法向集合中添加元素，其源码实现如下：

```java
	/**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  
        elementData[size++] = e;
        return true;
    }
```

**ensureCapacityInternal()**方法，根据当前size进行判断是否需要扩容，判断逻辑如下源码。当ArrayList第一次添加元素时，elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA为true，此时minCapacity=DEFAULT_CAPACITY，接着继续调用ensureExplicitCapacity()方法进行判断

```java
	private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        ensureExplicitCapacity(minCapacity);
    }
```

首次添加元素，elementData为空数组实例DEFAULTCAPACITY_EMPTY_ELEMENTDATA，即length为0，接着进入grow()方法。if中大于0则表明当前添加元素之后，size会超过当前数组的length，需要扩容了；

```java
 	private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

从grow方法可以看出，ArrayList每次扩容为**原来的1.5倍**。向新的ArrayList中不断添加元素，当现有数组length的1.5倍大于``MAX_ARRAY_SIZE``时，则进入hugeCapacity()方法，如果ArrayList的minCapacity小于MAX_ARRAY_SIZE，则扩容至MAX_ARRAY_SIZE。继续添加元素，当minCapacity超过了MAX_ARRAY_SIZE，则扩容至Integer.MAX_VALUE。当达到了Integer.MAX_VALUE时，再往里添加元素就要抛异常OutOfMemoryError了；

```java
	/**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);//oldCapacity + oldCapacity/2
        if (newCapacity - minCapacity < 0) //只有第一次添加元素时，newCapacity=0，走该if
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    
```

此处有一个疑问，源码中定义的ArrayList的最大容量是``Integer.MAX_VALUE - 8``，之所以**减8**，源码中官方给的注释是``“Some VMs reserve some header words in an array”``，即部分虚拟机需要在数组中保留一些头部信息，但是为什么当ArrayList的size超过了Integer.MAX_VALUE - 8，还能继续扩容到Integer.MAX_VALUE ？

```java
	private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

