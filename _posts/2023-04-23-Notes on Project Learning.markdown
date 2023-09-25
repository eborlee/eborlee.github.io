---
layout:     post
title:      "Notes - Projects Learning | 项目学习笔记"
subtitle:   " \"\""
date:       2023-08-23 12:00:00
author:     "Yibo Li"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - C++
    - Quant
    - Python
    - Notes
---

> “”


# Projects Learning

2023-06-01

Yibo Li

yli12@stevens.edu; yiboli@link.cuhk.edu.hk

http://eborlee.github.io

## Table Of Contents

[TOC]

## Project Sources:

Async Trading Framework,  aat: https://github.com/AsyncAlgoTrading/aat

CppTrader: https://github.com/chronoxor/CppTrader/tree/master

C++ Memory Pool: https://github.com/cacay/MemoryPool

### 1) Python

##### 异步+回调的事件驱动机制：



关于是否要使用异步的事件驱动结构：

鉴于我自己的项目是一个用于中低频的python开发的，使用异步+事件驱动+事件循环的意义不是很大。

对于异步的需求仅仅在于



##### periodic定时器

##### 使用动态属性为类的方法添加属性并赋默认值

setattr(EventHandler.onTrade, "_original", 1)

对应hasattr

注意这种方式对象并不会获得类方法的属性

### 2) Speed Up: C++

**Structure：**

**Specific usages：**

编译：Pybind11

RAII智能指针：std::shared_prt & std::unique_ptr

static修饰

高频订单簿的管理：使用内存池

异常处理对性能的影响

类型别名的使用

订单簿-链表的使用

订单簿的管理：std::map