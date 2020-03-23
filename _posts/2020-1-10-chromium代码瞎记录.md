---
layout:     post
title:      chromium
subtitle:   
date:       2020-2-4
author:     XP
header-img: img/problems.jpg
catalog: true
tags:
    - chromium
---

## Chromium ##


### 1, .mm文件是什么？里面为什么有C++语法 ###
针对mac的实现，会出现.mm文件, [详细](https://blog.csdn.net/k16643275hn/article/details/51934742)：
```
Objective-c的工程中，会存在.m、.h、.mm这三种不同后缀名的文件，它们的区别如下：

.h ：头文件，它包含类名，类继承的父类，还有方法和变量的声明。它定义的类的成员变量以及方法等等是公开的，外部是可以访问的。

.m ：实现文件，可以包含Objective-C和C代码。同时，它是对.h文件中方法的实现，外部不能访问。

.mm ：实现文件，和.m文件类似，唯一的不同点就是，除了可以包含Objective-C和C代码以外，还可以包含C++代码。仅在你的Objective-C代码中确实需要使用C++类或者特性的时候才用这种
```
