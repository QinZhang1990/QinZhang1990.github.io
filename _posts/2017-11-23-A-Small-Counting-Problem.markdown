---
layout: post
title: "如何计算字符串中的大写、小写字母及非字母的个数"
date: 2017-11-23 21:10:00.000000000 +00:00
tags: 他山之石
---

### 我的想法

我一开始想到的就是需要先将字符串，通过split()方法拆分为一个字符数组charArray，然后再对字符数组中的每一个字符做处理，
接下来分为三步。

#### 1 如何区分字母和非字母？

需要先定义一个字符数组alphabetArray，该字符数组中包含从a到z，和从A到Z的所有字符，然后遍历charArray中的字符是否都在
alphabetArray数组中，然后计数那些不在该数组中的元素的个数。

#### 2 如何计算大写字母和小写字母的个数

因为大、小写字母都可以转为ASCII码值，如小写a为97，小写的z为122，大写的A为65，大写的Z为90，所以判断charArray中的字符
的范围是在97到122之间，还是在65到90之间，然后分别计数。不过这种方法得先排除掉非字母的字符，然后遍历的时候取字母的ASCII值。
```swift
char   c   =   'a';
byte   b   =(byte)c;
System.out.println(b);
```

### 别人的想法

感觉我的想法，实施起来比较麻烦，网上看了下别人的想法，原来忘记了String类中的charAt()放法，该方法可以取到任意位置上的字符，
这样就不需要把字符串拆分成字符数组了；其次，字母可以直接同'a'，'z'或是'A','Z'进行比较，也不需要转换为ASCII码值，再进行比较，
具体的实现代码如下（[查看原文](http://study.163.com/course/courseLearn.htm?courseId=342010#/learn/video?lessonId=456137&courseId=342010)）：
```swift
public static void main(String[] args){
		//分别计算一个字符串中，大写的字母，小写字母和不是字母的字符的个数
		String s = "AAaABbbbEEE$@*8@%EeeOOfjFWFJLL:WW";
		int lcount=0;
		int ucount=0;
		int ocount=0;
		//开始计算每一类字符的个数
		for(int i=0,len=s.length();i<len;i++){
			char c = s.charAt(i);
			if(c>'a' && c<'z'){
				lcount++;
			}else if(c>'A' && c<'Z'){
				ucount++;
			}else{
				ocount++;
			}
		}
		System.out.println(lcount);
		System.out.println(ucount);
		System.out.println(ocount);
	}
```




