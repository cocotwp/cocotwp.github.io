---
layout: article
title: Spark Core
tags: 
- Spark
- 黑马
toc: true
---

## 第1章 RDD 详解

### 为什么需要 RDD

分布式计算需要：

- 分区控制
- Shuffle 控制
- 数据存储\序列化\发送
- 数据计算 API
- 等一系列功能

### 什么是 RDD

**定义**

`RDD`（Resilient Distributed Dataset）叫做弹性分布式数据集，是 Spark 中最基本的数据抽象，代表一个不可变、可分区、可并行计算的集合。

- Resilient：RDD 中的数据可以存储在内存中或者磁盘中
- Distributed：RDD 中的数据是分布式存储的，可用于分布式计算
- Dataset：一个数据集合，用于存放数据的

- RDD，是 Spark 中最基本的数据抽象，代表一个不可变、可分区、可并行计算的集合。
- 所有的运算以及操作都建立在 RDD 数据结构的基础之上。
- 可以认为 RDD 是分布式的列表 List 或数组 Array，抽象的数据结构，RDD 是一个抽象类（Abstract Class）和泛型（Generic Type）。

### RDD 的五大特性

1. A list of partitions（一系列的分片）
2. A function for computing each split（一个函数可计算每一个分片）
3. A list of dependencies on other RDDs（RDD 间有依赖关系：RDD1 --> RDD2 --> RDD3）
4. Optionally,a Partitioner for Key-value RDDs（可选，Key-Value 型的 RDD 可以有分区器）
5. Optionally, a list of preferred locations to compute each split on（可选，尽可能在本地运行）

<div align="center">
	<img src="https://raw.githubusercontent.com/cocotwp/cocotwp.github.io/master/assets/images/spark基础入门/RDD五大特性示例.png" alt="RDD五大特性示例" width="75%" />
</div>

## 第2章 RDD 编程入门



## 第3章 RDD 持久化

## 第4章 RDD 案例练习

## 第5章 RDD 共享变量

## 第6章 Spark 内核调度
