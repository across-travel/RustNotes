---
title: Java ConcurrentHashMap 简介与源码阅读
date: 2018-01-24 20:48:48
category: Java_note
tag: [Java]
toc: true
---

本文概要
* ConcurrentHashMap 简介
* ConcurrentHashMap 源码解析
    * 属性介绍
    * 方法流程介绍

### ConcurrentHashMap 简介
`java.util.concurrent`包提供了映射表的高效实现。使用复杂的算法，通过允许并发地访问数据结构的不同部分来使竞争极小化。
与大多数集合不同，`size`方法通常需要遍历。

* concurrencyLevel 预计并发操作的线程个数。也用来限制initialCapacity的最小值。
```java
public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel)
```
