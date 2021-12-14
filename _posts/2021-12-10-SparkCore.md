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

### RDD 的创建

**并行化创建**（集合对象）

API：
```python
SparkContext.parallelize(_c_, _numSlices=None_)
```

代码：
```python
from pyspark import SparkConf, SparkContext

if __name__ == '__main__':
    conf = SparkConf().setAppName('myApp').setMaster('local')
    sc = SparkContext(conf=conf)

    rdd = sc.parallelize([1, 2, 3, 4, 5, 6, 7, 8, 9])  # 默认分区数
    print(rdd.getNumPartitions())

    rdd = sc.parallelize([1, 2, 3, 4, 5, 6, 7, 8, 9], 3)  # 指定分区数
    print(rdd.getNumPartitions())

    print(rdd.collect())
```

**读取外部数据源**

API：
```python
SparkContext.textFile(_name_, _minPartitions=None_, _use_unicode=True_)
```

支持本地文件、HDFS、S3协议

代码：
```python
from pyspark import SparkConf, SparkContext

if __name__ == '__main__':
    conf = SparkConf().setAppName('myApp').setMaster('local')
    sc = SparkContext(conf=conf)

    sc.setLogLevel('INFO')

    file_rdd1 = sc.textFile('./data/input/words.txt')
    print(file_rdd1.getNumPartitions())
    print(file_rdd1.collect())

    file_rdd2 = sc.textFile('./data/input/words.txt', 3)
    print(file_rdd2.getNumPartitions())

    file_rdd3 = sc.textFile('./data/input/words.txt', 100)
    print(file_rdd3.getNumPartitions())
```

API：
```python
SparkContext.wholeTextFiles(path, minPartitions=None, use_unicode=True)
```

适合读取一堆小文件\
同样支持本地文件、HDFS、S3协议

代码：
```python
from pyspark import SparkConf, SparkContext

if __name__ == '__main__':
    conf = SparkConf().setAppName('myApp').setMaster('local')
    sc = SparkContext(conf=conf)

    rdd = sc.wholeTextFiles('./data/input/tiny_files', 3)
    print(rdd.collect())
```

### RDD 算子

**算子是什么**

分布式集合对象上的 API 称为*算子*。

分成两类：
- *转换（Transformation）算子*：返回值仍旧是一个 RDD 的
- *动作（Action）算子*：返回值**不是** RDD 的

> **懒加载**：转换算子只是在构建执行计划，动作算子是一个指令让执行计划开始工作。

### 常用转换算子

#### map

将方法应用到 RDD 中的每个元素

API：
```python
RDD.map(f, preservesPartitioning=False)
```

代码：
```python
from pyspark import SparkConf, SparkContext

if __name__ == '__main__':
    conf = SparkConf().setAppName('myApp').setMaster('local')
    sc = SparkContext(conf=conf)

    rdd = sc.parallelize([1, 2, 3, 4, 5, 6, 7, 8, 9], 3)

    # def double(num):
    #     return num * 2

    # print(rdd.map(double).collect())

    print(rdd.map(lambda x: x * 2).collect())
```

#### flatMap

先进行 map 操作，再进行 **展平** 操作

API：
```python
RDD.flatMap(f, preservesPartitioning=False)
```

代码：
```python
from pyspark import SparkConf, SparkContext

if __name__ == '__main__':
    conf = SparkConf().setAppName('myApp').setMaster('local')
    sc = SparkContext(conf=conf)

    rdd = sc.parallelize(["hello spark", "hello apache", "hello replit"])
    rdd2 = rdd.flatMap(lambda x: x.split(" "))
    print(rdd2.collect())
```

#### reduceByKey

针对 KV 型 RDD，自动按照 key 分组，然后对 value 应用聚合函数。

聚合逻辑如下图：

<div align="center">
	<img src="https://raw.githubusercontent.com/cocotwp/cocotwp.github.io/master/assets/images/sparkcore/reduceByKey算子聚合逻辑.png" alt="reduceByKey算子聚合逻辑" width="50%"/>
</div>

API：
```python
RDD.reduceByKey(func, numPartitions=None, partitionFunc=<function portable_hash>)
```

代码：
```spark
from pyspark import SparkConf, SparkContext

if __name__ == '__main__':
    conf = SparkConf().setAppName('myApp').setMaster('local')
    sc = SparkContext(conf=conf)

    # 实现 WordCount
    rdd = sc.textFile('./data/input/words.txt')
    rdd1 = rdd.flatMap(lambda x: x.split(" "))
    rdd2 = rdd1.map(lambda x: (x, 1))
    rdd3 = rdd2.reduceByKey(lambda a, b: a + b)
    print(rdd3.collect())
```

#### mapValue

针对KV 型 RDD，对 value 应用 map 函数，不影响 key

API：
```python
RDD.mapValues(f)
```

代码：
```python
rdd = sc.parallelize([('a', 1), ('b', 2), ('c', 3)])
print(rdd.mapValues(lambda x: x * 2).collect())
```

#### groupBy

将 RDD 的数据进行分组

API：
```python
RDD.groupBy(f, numPartitions=None, partitionFunc=<function portable_hash>)
```

代码：
```python
rdd = sc.parallelize([('a', 1), ('b', 2), ('c', 1), ('a', 1)])
result = rdd.groupBy(lambda x: x[0])
print(result.map(lambda x: (x[0], list(x[1]))).collect())
```



## 第3章 RDD 持久化

## 第4章 RDD 案例练习

## 第5章 RDD 共享变量

## 第6章 Spark 内核调度
