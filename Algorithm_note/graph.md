---
title: 图（Graph）
date: 2015-08-12 15:08:17
category: Algorithm
tag: [algorithm]
---
是由顶点的有穷非空集合和顶点之间边的集合组成，通常表示为：G(V,E)，其中G表示一个图，V是图G中顶点的集合，E是图G中边的集合

在图中数据元素，称之为**顶点（Vertex)** ；图结构中，不允许没有顶点

图中，任意两个顶点之间都有可能有关系，顶点之间的逻辑关系用**边**来表示，边集可以是空的

边可以有方向，**有向边称为弧**；无向边可以表示为(A,D)或(D,A)

都是无向边，称为无向图；都为有向边，称为有向图

## 图的邻接矩阵（Adjacency Matrix）
存储方式是用两个数组来表示图。一个一维数组存储图中顶点信息，一个二维数组（称为邻接矩阵）存储图中的边或弧的信息。

顺序存储结构在预先分配内存时，可能造成存储空间的浪费。可以考虑对边或弧使用链式存储的方式来避免浪费。

数组与链表相结合的存储方法称为**邻接表（Adjacency List）**

### 邻接表处理方法：

1. 图中顶点用一个一维数组存储。对于顶点数组中，每个数据元素还需要存储指向第一个邻接点的指针，便于查找该顶点的边信息。

2. 图中每个顶点vi的所有邻接点构成一个线性表，由于邻接点的个数不定，所以用单链表存储。无向图称为顶点vi的边表，有向图称为顶点vi作为弧尾的出边表。