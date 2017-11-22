---
layout: post
title: "ibatis中CDATA标签的使用"
date: 2017-11-22 21:17:00.000000000 +00:00
tags: 他山之石
---

### 1 问题

最近在做项目的过程中用到ibatis框架，项目队友编写的DAO层sqlmapping配置文件中，有如下代码段，对这段代码中用到的CDATA，
甚是不了解，便上网百度了一番！

![](/assets/images/2017/CDATA.PNG)

### 2 为什么要用CDATA

ibatis的sqlmapping.xml文件本质上是一种XML格式的文本，XML解析器一般会处理XML文档中的所有文本。当XML元素，如`<select>`、
`<message>`等等，被解析时，XML元素内部的文本也会被解析，当元素中嵌套元素时，内层元素也会被解析，如下：

`<position><altitude>31.45</altitude><longitude>121.33</longitude></position> `

其解析方式，同如下代码一样：

```swift
<position>
    <altitude>31.45</altitude>
    <longitude>121.33</longitude>
</position>
```
所以，对于`<`、`>`、`&`等符号便不能直接存入XML，尤其是“<”，会被当做新的元素标签的开始，这样解析时便会出现问题。

### 3 如何解决

方法一：进行转译，将其转换成XML中预定义的实体，XML中预定义好了以下实体

| `&lt;` | `<` | 小于号 |
| `&gt;` | `>`| 大于号 |
| `&amp;`| `& `| 和 |
| `&apos;` | `'` | 单引号 |
| `&quot` | `"` | 双引号 |
方法二 XML解析时，不去解析带有上述特殊符号的代码片段，将其视为纯文本，XML中则是通过cdata实现的，
cdata是用来表明纯文本的，可省去转译操作，能够避免特殊符号导致XML解析出错。

### 4 使用CDATA需要注意的地方

（1）CDATA是不可以嵌套使用的，而且CDATA部件包含了字符"]]>"或者"<![CDATA[" ，将很有可能出错。
（2）在字符串“]]>”之间没有空格或者换行符。
