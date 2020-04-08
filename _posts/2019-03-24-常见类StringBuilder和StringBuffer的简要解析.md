---
layout:     post
title:      StringBuffer & StringBuilder 简要解析
subtitle:   关于String & StringBuffer & StringBuilder类的学习记录
date:       2019-03-24
author:     LJJ
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - java
---

# StringBuffer & StringBuilder 简要解析
前述：String & StringBuffer & StringBuilder 都是常用的与字符串相关的类，其用法各有不同，相互扩展使得需求的实现更为简洁方便，这里做一个简要的记录自己对这三个类的理解和学习过程。

### 两者由来
提到StringBuffer & StringBuilder，就不得不说一下String类的特性，即： 
> All string literals in Java programs are implemented as instances of String class.  
> Strings are constant and their values cannot be changed after they are created.  
> String buffers support mutable strings.Because String objects are immutable they can be shared. 

在Java中，String类是不可变的，其对象是线程安全的，那么字符串不能够同时被多个线程共用，字符串一旦创建将不能够被改变。我们基于String类的字符拼接删改操作都是要创建一个新的字符串，并不改变原来的字符串。  
而相对于String类来说，StringBuffer & StringBuilder都是可变的。其对象被创建后，会在堆内存new一块内存，对两者对象的修改都是直接改变heap中的值。

### 两者共性和差异
所属模块和所在包类：
- Module java.base
- Package java.lang
- public final class StringBuilder extends Object 
- 实现的接口类似

主要区别就是：
- StringBuffer是 thread-safe的。
- StringBuilder 则 with no guarantee of synchronization and  being used by a single thread,使用速度更快。

### 使用方法细节
这里主要介绍常见的方法，欲要详细了解详见  
[StringBuffer API](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/StringBuffer.html)  
[StringBuilder API](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/StringBuilder.html)

构造函数：
- StringBuilder()；tringBuilder()
构造一个字符串生成器，其中没有字符，初始容量为16个字符。

方法：
- append()；insert()
它们被重载以接受任何类型的数据。每个都有效地将给定的数据转换为字符串，然后将该字符串的字符追加或插入字符串生成器。该 append方法始终在构建器的末尾添加这些字符; 该insert方法在指定点添加字符。

### 应用实例
    public class myStringBuilder {
    	public static void main(String[] args) {
    		StringBuilder str1 = new StringBuilder("abc");
    		//注意这里的str2,str1共用一个地址
    		StringBuilder str2 = str1;
    		String str3 = "hello world";
    		StringBuilder str4 = new StringBuilder("abc");
    		
    		for(int i=0;i<str3.length();i++) {
    			str1.append(str3.charAt(i));
    		}
    		//打印使用append()方法追加后的效果
    		System.out.println(str1.toString());
    		
    		for(int i=0;i<str3.length();i++) {
    			str2.insert(2, str3.charAt(i));
    			str4.insert(2, str3.charAt(i));
    		}
    		//已经改变了的str2再次插入后的打印结果
    		System.out.println(str2.toString());
    		//对比str2的运行效果
    		System.out.println(str4.toString());
    	}
    }
    Console output:
    abchello world
    abdlrow ollehchello world
    abdlrow ollehc

