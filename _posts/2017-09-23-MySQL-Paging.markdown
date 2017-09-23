---
layout: post
title: "MySQL数据库分页"
date: 2017-09-23 23:30:00.000000000 +00:00
tags: 他山之石
---

## 1 分页的必要性

(1) 当数据量很大时，页面上将其全部显示出来，没有实际意义.
(2) 遍历查询全部记录需要较多时间，占用较多带宽，响应客户端请求时间变长，效率降低，进而使用用户体验差。

## 2 MySQL分页原理

(1) 数据库分页需要确定以下信息：

` pageSize `——每页显示的记录数据；
` totalCount `——总共的记录数
` totalPage `——总共的页数，总页数需要向上取整，其计算公式如下：
totalPage = totalCount%pageSize==0? totalCount/pageSize:(totalCount/pageSize)+1;
` currentPage `——当前页数

(2) mysql数据库分页采用limit关键字

Select * from tableName limit m，n；
其中`m`是指记录开始的index，从0开始，表示第一条记录
`n`是指从第m+1条开始，取n条。
实际的分页中m = (currentPage-1)*pageSize; n = pageSize，如每页显示5条记录，现在查询第二页，则m=(2-1)*5，n=5；

