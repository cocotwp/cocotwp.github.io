---
layout: article
title: Spark Core - Part III
tags: 
- Spark
- 黑马
toc: true
---

## 第6章 Spark 内核调度

### DAG

有向无环图（Directed Acyclic Graph）

Spark 的核心是根据 **RDD** 来实现的，Spark Scheduler 则为 Spark 核心实现的重要一环，其作用就是任务调度。Spark 的任务调度就是如何组织任务去处理 RDD 中每个分区的数据，根据 RDD 的依赖关系构建 `DAG`，基于 DAG 划分 `Stage`， 将每个 Stage 中的任务发到指定节点运行。基于 Spark 的任务调度原理，可以合理规划资源利用，做到尽可能用最少的资源高效地完成任务计算。

[[2021-12-15-SparkCore#RDD 的缓存\|RDD 的缓存]] 对应的 DAG：

<div align="center">
	<img src="https://raw.githubusercontent.com/cocotwp/cocotwp.github.io/master/assets/images/sparkcore/持久化DAG.png" alt="持久化DAG" width="95%" />
</div>

**结论**：1个Action = 1个DAG = 1个Job

### 宽窄依赖和阶段划分

- 窄依赖：父 RDD 的一个分区，将数据*全部*发给子 RDD 的*一个*分区（一对一、多对一）
- 宽依赖：父 RDD 的一个分区，将数据发给子 RDD 的*多个*分区（一对多），又称 `shuffle`

对于 Spark 来说，会根据 DAG，按照宽依赖划分不同的阶段\
*划分依据*：从后向前，遇到宽依赖，就划分出一个阶段，称之为 `stage`

<div align="center">
	<img src="https://raw.githubusercontent.com/cocotwp/cocotwp.github.io/master/assets/images/sparkcore/阶段划分.png" alt="阶段划分" width="80%" />
</div>

### 内存迭代计算

Spark 默认受到全局并行度的限制，除了个别算子有特殊分区情况，大部分的算子，都会遵循全局并行度的要求，来规划自己的分区数。

**Spark 为什么比 MapReduce 快**

1. 编程模型上 Spark 算子丰富
2. 算子交互上和计算上可以尽可能多的内存计算而非磁盘迭代。

### Spark 并行度

**并行度**：在同一时间内，有多少个 `task` 在同时运行

#### 全局并行度-推荐

配置文件中：
```
# conf/spark-defaults.conf
spark.default.parallelism 100
```

在客户端提交参数中：
```
spark-submit --conf "spark.default.parallelism=100"
```

在代码中设置：
```
conf = SparkConf()
conf.set("spark.default.parallelism", "100")
```

#### 针对 RDD 的并行度设置-不推荐

只能在代码中，算子：
- repartition
- coalesce
- partitionBy

#### 并行度规划建议

**结论**：设置为 CPU 总核心数的2～10倍

### Spark 的任务调度

Spark 程序的调度流程如下：
1. Driver 被构建出来
2. 构建 SparkContext（执行环境入口对象）
3. 基于 DAG Scheduler（DAG 调度器）构建逻辑 Task 分配
4. 基于 TaskScheduler（Task 调度器）将逻辑 Task 分配到各个 Executor 上，并监控它们
5. Executor 被 TaskScheduler 管理监控，听从它们的指令干活，并定期汇报进度

<div align="center">
	<img src="https://raw.githubusercontent.com/cocotwp/cocotwp.github.io/master/assets/images/sparkcore/spark调度流程图.png" alt="spark调度流程图" width="90%" />
</div>

**DAG 调度器**

工作内容：将逻辑的 DAG 图进行处理，最终得到逻辑上的`Task`划分

**Task 调度器**

工作内容：基于 DAG 调度器的产出，来规划*逻辑*的 task，应该在哪些*物理*的 executor 上运行，以及监控管理它们的运行。

