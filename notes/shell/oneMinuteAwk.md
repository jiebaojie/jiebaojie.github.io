---
layout: post
notes: true
subtitle: "【技术博客】一分钟学awk够用（产品经理都懂了）"
comments: false
author: "58沈剑 架构师之路"
date: 2017-04-04 00:00:00

---


原文：[http://www.wtoutiao.com/p/Z61zWD.html](http://www.wtoutiao.com/p/Z61zWD.html)

*   目录
{:toc }

# 1. 什么是AWK

1.	Aho、Weinberger、Kernighan三位发明者名字首字母
2.	一个行文本处理工具

# 2. AWK基本原理

## 2.1 原理：逐行处理文件中的数据

## 2.2 语法

	awk 'pattern + {action}'

说明：

1.	单引号''是为了和shell命令区分开；
2.	大括号{}表示一个命令分组；
3.	pattern是一个过滤器，表示命中pattern的行才进行action处理；
4.	action是处理动作；
5.	使用#作为注释；

例子：显示hello.txt中的第3行至第5行

	cat hello.txt | awk 'NR==3, NR==5{print;}'
	
## 2.3 pattern说明

pattern参数可以是egrep正则中的一个，正则使用/pattern/

例子：显示hello.txt中，正则匹配hello的行

	cat hello.txt | awk '/hello/'
	
说明：

1.	pattern和action可以只有其一，但不能两者都没有；
2.	默认的action是print；

例子：显示hello.txt中，长度大于80的行号

	cat hello.txt | awk 'length($0)>80{print NR}'

# 3. 内置变量

FS 分隔符，默认是空格

NR 当前行数，从1开始

NF 当前记录字段个数

$0 当前记录

$1~$n 当前记录第n个字段

例子：显示hello.txt中的第3行至第5行的第一列与最后一列

	cat hello.txt | awk 'NR==3, NR==5{print $1,$NF}'
	
# 4. 内置函数

*	gsub(r,s)：在$0中用s代替r
*	index(s,t)：返回s中t的第一个位置
*	length(s)：s的长度
*	match(s,r)：s是否匹配r
*	split(s,a,fs)：在fs上将s分成序列a
*	substr(s,p)：返回s从p开始的子串

# 5. 操作符

## 5.1 运算符

类似于c，支持+、-、*、/、%、++、–、+=、-=等诸多操作；

## 5.2 判断符

类似于c，支持==、!=、>、=>、~（匹配于）等诸多判断操作；

# 6. 控制流程

## 6.1 BEGIN和END

BEGIN和END本质是一个pattern。

BEGIN用于awk程序开始开始前，做一些初始化的工作；

END用于awk程序结束前，做一些收尾的工作。

例子：统计字符个数

	awk '
	BEGIN
	{
	count=0;
	}
	{
	count+=length($0);
	}
	END
	{
	print count;
	}'
	
## 6.2 流程控制语句

1.	if(condition){}else{}
2.	while{}
3.	do{}while(condition);
4.	for(init;condition;step){}
5.	break/continue：如果有END，会执行END中的收尾工作

流程控制语句用法几乎与c相同。

# 7. awk与shell的交互

## 7.1 awk中使用shell中定义的变量：使用单引号即可；

	#!/bin/bash
	STR="hello"
	echo | awk '{
	print "'${STR}'";
	}'
	
## 7.2 awk中使用shell命令：使用双引号，或者system命令；

	#!/bin/bash
	echo hello | awk '{
	print $0 | "cat"
	}'
	
或者

	#!/bin/bash
	echo | awk '{
	system("date > date.txt")
	}'
	
## 7.3 awk中的变量传出至shell：没有什么好方法，老老实实用文件吧；

## 7.4 getline：awk里，从文件中读取变量到awk中

	#!/bin/bash
	echo | awk '{
	while(getline < "date.txt")
	{
	print $0;
	}
	}'