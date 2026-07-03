# Module 1: 流处理基础与 Flink 核心原理

> **难度**: ⭐⭐ | **预计学习时长**: 6–8 小时 | **前置要求**: Java 8+ 基础、Maven 基本使用
>
> *"理解流处理不是学会一个框架，而是换一种看待数据的方式。"*

---

## 目录

1. [为什么我们需要流处理？](#11-为什么我们需要流处理)
2. [批处理 vs 流处理：本质区别](#12-批处理-vs-流处理本质区别)
3. [Apache Flink 的诞生与设计哲学](#13-apache-flink-的诞生与设计哲学)
4. [Flink 架构全景](#14-flink-架构全景)
5. [核心概念深度解析](#15-核心概念深度解析)
6. [你的第一个 Flink 程序](#16-你的第一个-flink-程序)
7. [Flink 程序骨架完全拆解](#17-flink-程序骨架完全拆解)
8. [DataStream API 核心操作详解](#18-datastream-api-核心操作详解)
9. [并行度与执行图](#19-并行度与执行图)
10. [常见问题与调试技巧](#110-常见问题与调试技巧)
11. [模块练习](#111-模块练习)
12. [延伸阅读与参考](#112-延伸阅读与参考)

---

## 1.1 为什么我们需要流处理？

### 1.1.1 数据正在变成"流"

想象一下十年前的互联网：用户每天访问几次网站，产生几条日志。数据以"天"为单位积累，每晚跑一个批处理作业统计 PV/UV，第二天早上看报表，完全够用。

但今天：

- **一个中等规模的电商**，每秒产生数万条用户行为事件（点击、加购、下单、支付）
- **一个 IoT 平台**，数十万台设备每秒钟上报传感器数据
- **一个金融交易系统**，需要在毫秒级检测到异常交易并阻断
- **一个短视频平台**，需要实时推荐、实时审核、实时统计播放量

这些数据不是"攒够了再处理"，而是**源源不断地到来**。等你攒够一小时的批处理，欺诈交易已经完成了，爆款视频的热度已经过去了，设备故障已经造成损失了。

> **流处理的本质**：在数据到达的那一刻就进行处理，在延迟和正确性之间找到平衡。

### 1.1.2 实时不等于"快一点点"

很多人对流处理有一个误解：流处理就是把批处理做得更快一点。

这是错的。

批处理的核心假设是：**数据是静止的、完整的、有界的**。你可以排序、可以全局聚合、可以多次遍历。

流处理面对的是**无界数据流（Unbounded Stream）**——数据永不停歇，你永远看不到"全部数据"。这带来了一系列根本性的挑战：

| 挑战 | 批处理的视角 | 流处理的视角 |
|------|-----------|-----------|
| **数据完整性** | 所有数据都已就绪 | 数据永远在来的路上 |
| **时间语义** | 处理时间 = 事件时间 | 必须区分事件时间和处理时间 |
| **容错** | 失败重跑整批即可 | 必须支持增量状态的精确恢复 |
| **结果输出** | 最终一次性输出 | 持续输出、可修正 |

### 1.1.3 流处理不是替代批处理

一个常见的误区是"流处理要取代批处理"。实际上：

- **批处理**仍然非常适合：历史数据分析、离线报表、大规模 ETL、机器学习模型训练
- **流处理**更适合：实时监控、实时推荐、实时风控、实时 ETL

Flink 的独特之处在于它提出了**"流批一体"**（Unified Stream-Batch Processing）的理念：同一套 API、同一套执行引擎、同一套语义，既可以处理流也可以处理批。

---

## 1.2 批处理 vs 流处理：本质区别

### 1.2.1 有界 vs 无界

```
批处理的数据集:
┌─────────────────────────────────────┐
│  [========== 有界数据集 ==========]  │
│  所有数据已知，大小固定               │
│  可以排序、可以全局聚合               │
└─────────────────────────────────────┘

流处理的数据流:
┌─────────────────────────────────────┐
│  → → → → → → → → → → → → → → → → │
│  数据永无止境，持续到来               │
│  只能增量处理，无法等待"全部"         │
└─────────────────────────────────────┘
```

### 1.2.2 三种处理模型对比

```
┌─────────────────────────────────────────────────────────────┐
│                    三种流处理模型                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  模型1: 原生流处理 (Native Streaming)                        │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  数据到达 → 立即处理 → 输出结果                        │   │
│  │  Flink, Kafka Streams, Storm (后期版本)              │   │
│  │  ✅ 真正的逐条处理                                    │   │
│  │  ✅ 低延迟（毫秒级）                                  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  模型2: 微批处理 (Micro-Batching)                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  数据到达 → 攒成小批次 → 批量处理 → 输出结果           │   │
│  │  Spark Streaming (DStream), Spark Structured Streaming│   │
│  │  ⚠️ 延迟取决于批次间隔（秒级）                        │   │
│  │  ⚠️ 难以实现真正的低延迟                              │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  模型3: 连续处理 (Continuous Processing)                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Spark Structured Streaming 的连续模式               │   │
│  │  ⚠️ 实验性功能，生产环境使用较少                      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1.2.3 时间语义：为什么流处理的时间如此重要？

在批处理中，"时间"很简单——数据到了，处理一下，记录处理的时间戳。但在流处理中，有三层时间：

```
事件时间 (Event Time)
    ↓ 数据产生的时间（用户点击按钮的那一刻）
    ↓ 最准确，但最难处理——因为数据可能乱序、延迟到达

摄取时间 (Ingestion Time)
    ↓ 数据进入 Flink 的时间
    ↓ 中等准确，不受机器处理速度影响

处理时间 (Processing Time)
    ↓ 算子实际处理数据的时间
    ↓ 最简单，但最不准确——机器卡顿会导致时间漂移
```

**示例场景**：用户 10:00:00 点击下单，由于网络抖动，数据 10:00:05 才到达 Kafka，Flink 在 10:00:06 处理这条数据。

- 事件时间 = 10:00:00
- 摄取时间 = 10:00:05
- 处理时间 = 10:00:06

Flink 的核心优势之一就是**原生支持事件时间处理**，配合 Watermark 机制可以正确处理乱序数据和延迟数据。这一点 Spark Streaming 在很长时间内都难以做到。

---

## 1.3 Apache Flink 的诞生与设计哲学

### 1.3.1 从 Stratosphere 到 Flink

Flink 的故事始于 2009 年的柏林理工大学研究项目 **Stratosphere**。研究者发现当时的大数据框架（主要是 Hadoop MapReduce）在处理迭代算法和流式数据时存在根本性缺陷。

关键发展时间线：

```
2009  Stratosphere 研究项目启动（柏林理工大学）
2010  首次发布，专注于迭代数据处理
2014  更名为 Apache Flink，进入 Apache 孵化器
2014.12 成为 Apache 顶级项目
2015  Flink 1.0 发布，流批一体 API 稳定
2016  Flink 1.1 发布，Table API 初版
2017  Flink 1.3 发布，增量 Checkpoint
2019  Flink 1.9 发布，阿里巴巴 Blink 合并
2020  Flink 1.11 发布，原生 Kubernetes 支持
2023  Flink 1.18 发布，自适应调度器
2024  Flink 1.20 发布，当前稳定版本
```

### 1.3.2 Flink 的设计哲学

Flink 的设计深受学术研究和工业实践的双重影响，形成了几个核心设计原则：

#### 原则1: 真正的流处理引擎

Flink 从底层就是为一个接一个的事件设计的，而不是"把流切成批"。这意味着：
- 每条数据独立处理，没有"攒批"的开销
- 延迟可以做到毫秒级
- 状态管理是增量式的，不是全量快照

#### 原则2: 精确一次（Exactly-Once）语义

"精确一次"不是说每条数据只被处理一次（这在分布式系统中是不可能的），而是说：**即使在故障恢复后，系统的最终状态与没有发生故障时完全一致**。

Flink 通过 **Checkpoint** 机制（基于 Chandy-Lamport 分布式快照算法）实现这一点。我们在 Module 4 会深入讲解。

#### 原则3: 有状态的流处理

Flink 不只是"把数据从 A 传到 B"，它可以在流中**维护和查询状态**。例如：
- 统计每个用户的累计消费金额（需要在状态中保存累计值）
- 检测用户是否在一分钟内连续点击了三次（需要在状态中保存点击历史）
- 实时 join 两个流（需要在状态中保存一个流的记录等待匹配）

状态管理是 Flink 区别于"简单流处理框架"的关键。

#### 原则4: 流批一体

Flink 的 DataStream API 既可以处理无界流，也可以处理有界数据集。批处理在 Flink 中被视为"有限的流"——流是批的超集，而不是批是流的近似。

---

## 1.4 Flink 架构全景

### 1.4.1 整体架构

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Flink 生态系统                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │   DataStream │  │   Table API  │  │   DataSet    │  │    SQL       │    │
│  │     API      │  │  (流批一体)   │  │  (已废弃)    │  │  (流批一体)   │    │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘    │
│         │                 │                  │                 │           │
│         └─────────────────┴──────────────────┴─────────────────┘           │
│                                   │                                        │
│                    ┌──────────────▼──────────────┐                         │
│                    │       Flink Runtime        │                         │
│                    │   (统一流批执行引擎)        │                         │
│                    └──────────────┬──────────────┘                         │
│                                   │                                        │
│  ┌────────────────────────────────┴────────────────────────────────┐      │
│  │                        部署层                                    │      │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │      │
│  │  │Standalone│ │   YARN   │ │Kubernetes│ │   AWS    │  ...     │      │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │      │
│  └─────────────────────────────────────────────────────────────────┘      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.4.2 运行时架构：JobManager 与 TaskManager

Flink 采用经典的 **Master-Worker** 架构：

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Flink 运行时架构                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                        JobManager (主节点)                           │   │
│  │                                                                     │   │
│  │   ┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐     │   │
│  │   │  Dispatcher  │  │ ResourceManager  │  │   JobMaster      │     │   │
│  │   │              │  │                  │  │  (每个作业一个)   │     │   │
│  │   │ • 接收作业提交 │  │ • 管理集群资源   │  │                  │     │   │
│  │   │ • 启动 JobMaster    │ • 申请/释放 TM   │  │ • 生成执行图     │     │   │
│  │   │ • 提供 REST API   │ • 协调不同部署模式│  │ • 调度任务       │     │   │
│  │   └──────────────┘  └──────────────────┘  │ • 协调 Checkpoint │     │   │
│  │                                            └──────────────────┘     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                              心跳 / RPC / 网络                                   │
│                                    │                                        │
│  ┌─────────────────────────────────▼─────────────────────────────────────┐ │
│  │                    TaskManager (工作节点) × N                          │ │
│  │                                                                       │ │
│  │   ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐     │ │
│  │   │ Task Slot  │  │ Task Slot  │  │ Task Slot  │  │ Task Slot  │     │ │
│  │   │  [Subtask] │  │  [Subtask] │  │  [Subtask] │  │  [Subtask] │     │ │
│  │   │  [Subtask] │  │  [Subtask] │  │            │  │            │     │ │
│  │   └────────────┘  └────────────┘  └────────────┘  └────────────┘     │ │
│  │                                                                       │ │
│  │   每个 TaskManager 包含:                                               │ │
│  │   • 一组 Task Slot（资源隔离单元）                                      │ │
│  │   • 本地状态存储 (State Backend: Memory/RocksDB)                       │ │
│  │   • 网络缓冲池 (Network Buffer Pool，用于数据交换)                     │ │
│  │   • JVM 堆内存 / 托管内存                                              │ │
│  │                                                                       │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### JobManager 详解

JobManager 是整个集群的大脑，主要职责：

1. **调度器（Scheduler）**：将作业的执行图（JobGraph）拆分为可并行执行的任务，分配到各个 TaskManager 的 Slot 上
2. **检查点协调器（Checkpoint Coordinator）**：周期性触发分布式快照，协调所有算子保存状态
3. **资源管理**：与 ResourceManager 配合，申请或释放 TaskManager 资源

生产环境中，JobManager 通常部署多个实例（HA 模式），通过 ZooKeeper 或 Kubernetes 实现 Leader 选举。

#### TaskManager 详解

TaskManager 是实际干活的工作节点：

1. **Task Slot**：TaskManager 上的资源隔离单元。每个 Slot 拥有独立的内存资源，可以执行一个或多个 subtask（如果它们被 chain 在一起）。
2. **状态后端（State Backend）**：TaskManager 负责维护算子的本地状态。Flink 支持三种状态后端：
   - MemoryStateBackend：状态存在 JVM 堆内存中（测试用，不适合生产）
   - FsStateBackend：状态存在 TaskManager 内存，异步快照到文件系统
   - RocksDBStateBackend：状态存在嵌入式 RocksDB 中，适合大状态场景
3. **网络层**：TaskManager 之间通过 Netty 进行数据交换。每个 TM 维护一个 Network Buffer Pool，用于缓存上下游算子之间的数据。

### 1.4.3 数据流转：从 Source 到 Sink

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  Source  │────▶│  Map     │────▶│ KeyBy    │────▶│ Window   │────▶│  Sink    │
│ (数据源)  │     │ (映射)   │     │ (分组)   │     │ (窗口)   │     │ (输出)   │
└──────────┘     └──────────┘     └──────────┘     └──────────┘     └──────────┘
      │               │               │               │               │
      ▼               ▼               ▼               ▼               ▼
   读取数据        数据转换        重分区          时间窗口聚合     结果输出
   Kafka/         x -> f(x)       相同key的        按时间范围       打印/文件/
   Socket/                        数据路由到        统计数据         Kafka/
   Collection                     同一实例                         JDBC...
```

---

## 1.5 核心概念深度解析

### 1.5.1 DataStream：Flink 的核心抽象

```java
// DataStream 是什么？
DataStream<Event> stream = ...;

// 它代表一个不可变的、分布式的、无界（或有界）的数据元素序列
// 每个元素都有一个类型（这里是 Event）
// 数据在流中从左向右流动，经过一系列转换
```

DataStream 与 Java Stream 的区别：

| 特性 | Java Stream | Flink DataStream |
|------|------------|------------------|
| 数据量 | 单机内存可容纳 | 分布式，无上限 |
| 执行模型 | 即时执行（eager） | 惰性执行（lazy） |
| 容错 | 无 | Checkpoint 精确恢复 |
| 状态 | 无 | 内置状态管理 |
| 并行 | 单线程 | 可配置并行度 |

### 1.5.2 Transformation：对数据流的变换

Transformation 是 Flink 中对 DataStream 的操作。关键理解：**Transformation 只是定义了"做什么"，并不立即执行**。

```java
// 这段代码不会立即运行！它只是构建了一个执行计划。
DataStream<Result> result = input
    .map(new MyMapper())        // Transformation 1: 映射
    .filter(new MyFilter())     // Transformation 2: 过滤
    .keyBy(...)                 // Transformation 3: 分组
    .window(...)                // Transformation 4: 窗口
    .aggregate(...);            // Transformation 5: 聚合

// 只有这里才真正提交到集群运行
env.execute("My Job");
```

### 1.5.3 Operator 与 Subtask

- **Operator**：转换逻辑的定义。例如 `map()`、`filter()` 对应的逻辑。
- **Subtask**：Operator 的并行实例。如果一个 map 算子的并行度是 4，那么它会有 4 个 subtask 同时运行。

```
Operator (逻辑定义)
    │
    ├── 并行度 = 4
    │
    ├── Subtask 0  (运行在 TaskManager-1, Slot-0)
    ├── Subtask 1  (运行在 TaskManager-1, Slot-1)
    ├── Subtask 2  (运行在 TaskManager-2, Slot-0)
    └── Subtask 3  (运行在 TaskManager-2, Slot-1)
```

### 1.5.4 Parallelism：并行度

并行度决定了算子同时运行的实例数。

```java
// 设置全局默认并行度
env.setParallelism(4);

// 为单个算子设置并行度
stream.map(...).setParallelism(2);
```

并行度的优先级（从高到低）：
1. 算子级别设置 `.setParallelism(n)`
2. 执行环境设置 `env.setParallelism(n)`
3. 集群配置文件 `flink-conf.yaml` 中的 `parallelism.default`
4. 默认 = 1

### 1.5.5 Task Slot 资源分配

```
一个 TaskManager 有 3 个 Slot 的示例:

┌────────────────────────────────────────────────────┐
│                  TaskManager (JVM)                  │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐│
│  │   Slot 0     │ │   Slot 1     │ │   Slot 2     ││
│  │  ┌────────┐  │ │  ┌────────┐  │ │  ┌────────┐  ││
│  │  │Subtask │  │ │  │Subtask │  │ │  │Subtask │  ││
│  │  │  (map) │  │ │  │(filter)│  │ │  │(reduce)│  ││
│  │  └────────┘  │ │  └────────┘  │ │  └────────┘  ││
│  │  ┌────────┐  │ │              │ │              ││
│  │  │Subtask │  │ │              │ │              ││
│  │  │ (sink) │  │ │              │ │              ││
│  │  └────────┘  │ │              │ │              ││
│  └──────────────┘ └──────────────┘ └──────────────┘│
│                                                     │
│  Slot 共享：同一个 Slot 可以执行多个被 chain 在一起的算子 │
└────────────────────────────────────────────────────┘
```

### 1.5.6 State：有状态计算的核心

Flink 中的状态分两种：

**Keyed State**：与特定 key 绑定的状态。只能在 `KeyedStream` 上使用（即 `keyBy()` 之后）。

```java
// 每个用户有自己的计数器
stream.keyBy(event -> event.userId)
      .process(new KeyedProcessFunction<String, Event, Result>() {
          private ValueState<Long> counter;  // 每个 userId 一个独立的 counter
          
          @Override
          public void open(Configuration parameters) {
              counter = getRuntimeContext().getState(
                  new ValueStateDescriptor<>("counter", Types.LONG)
              );
          }
          
          @Override
          public void processElement(Event event, Context ctx, Collector<Result> out) 
                  throws Exception {
              Long current = counter.value();
              if (current == null) current = 0L;
              current++;
              counter.update(current);
              out.collect(new Result(event.userId, current));
          }
      });
```

**Operator State**：与算子实例绑定的状态。不依赖于 key，每个 subtask 有一份。

### 1.5.7 Checkpoint：分布式快照

Checkpoint 是 Flink 容错机制的核心。它的原理基于 **Chandy-Lamport 分布式快照算法**：

```
Checkpoint 过程:

1. Checkpoint Coordinator 向所有 Source 注入 Barrier（屏障）
   Source ──Barrier────────────────────────────▶

2. Source 收到 Barrier 后，保存自己的状态，向下游转发 Barrier
   Source ──Barrier──┬─────────────────────────▶
                     │ 保存状态 snapshot-1
                     ▼
                  [状态存储]

3. 每个算子收到所有输入流的 Barrier 后，保存状态，向下游转发
   
   Map ──┬──Barrier──┬─────────────────────────▶  收到上游所有 Barrier
         │           │                              ↓
         │        保存状态                         保存状态
         │           │                              ↓
         ▼           ▼                           转发 Barrier
      [状态存储]  [状态存储]

4. 当所有算子都确认状态保存完成，这次 Checkpoint 完成
   
   如果作业失败，从最新的 Checkpoint 恢复，所有算子回滚到保存的状态
```

---

## 1.6 你的第一个 Flink 程序

### 1.6.1 环境准备

你不需要下载 Flink 二进制包或启动集群。通过 Maven 依赖即可在本地运行 Flink 程序。

项目目录结构：

```
/root/flink-study/projects/
├── pom.xml                          # Maven 配置文件
└── src/
    └── main/
        └── java/
            └── com/example/module01/
                ├── WordCount.java       # 完整版 (需要 Socket)
                ├── WordCountSimple.java # 简化版 (内置数据)
                └── AverageCalculator.java # 练习题
```

核心依赖（pom.xml）：

```xml
<dependencies>
    <!-- Flink Streaming -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-streaming-java</artifactId>
        <version>1.20.0</version>
        <scope>provided</scope>
    </dependency>
    
    <!-- Flink 客户端（本地运行需要） -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-clients</artifactId>
        <version>1.20.0</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

### 1.6.2 简化版 WordCount（无需外部依赖）

这是最简单的入门程序，使用内置数据，直接运行即可看到结果。

**文件**: `src/main/java/com/example/module01/WordCountSimple.java`

```java
package com.example.module01;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

/**
 * Module 1 入门程序: WordCount 简化版
 * 
 * 使用 fromElements 创建测试数据，无需外部 Socket 或 Kafka
 * 直接运行即可看到结果
 */
public class WordCountSimple {
    
    public static void main(String[] args) throws Exception {
        // ===== 第1步: 创建执行环境 =====
        // StreamExecutionEnvironment 是所有 Flink 程序的入口
        // getExecutionEnvironment() 会自动判断运行环境:
        //   - 如果在集群中提交，连接到集群
        //   - 如果在本地 IDE 运行，启动本地迷你集群
        StreamExecutionEnvironment env = 
            StreamExecutionEnvironment.getExecutionEnvironment();
        
        // 设置并行度为 1，确保输出顺序可预测（仅用于本地测试）
        env.setParallelism(1);
        
        // ===== 第2步: 创建数据源 (Source) =====
        // fromElements 从给定的元素集合创建数据流
        // 适合测试和演示，生产环境通常用 KafkaSource、SocketTextStream 等
        DataStream<String> text = env.fromElements(
            "hello world hello flink",
            "apache spark apache hadoop",
            "flink is awesome flink is fast",
            "hello flink hello world"
        );
        
        // ===== 第3步: 数据转换 (Transformation) =====
        DataStream<Tuple2<String, Integer>> wordCounts = text
            // 3.1 flatMap: 将一行文本拆分为多个 (单词, 1) 对
            //    输入: "hello world" 
            //    输出: ("hello", 1), ("world", 1)
            .flatMap(new Tokenizer())
            
            // 3.2 keyBy: 按单词分组
            //    相同单词的数据会被路由到同一个 subtask
            //    这是后续 sum() 的前提——必须先分组才能聚合
            .keyBy(value -> value.f0)
            
            // 3.3 sum: 对每个单词的计数求和
            //    Tuple2 的第1个字段 (索引 0) 是单词，第2个 (索引 1) 是计数
            //    sum(1) 表示对第2个字段求和
            .sum(1);
        
        // ===== 第4步: 输出结果 (Sink) =====
        // print() 将结果输出到标准输出（控制台）
        // 生产环境会用 addSink() 写入 Kafka、JDBC、文件系统等
        wordCounts.print();
        
        // ===== 第5步: 执行作业 =====
        // Flink 是惰性执行的！前面所有代码只是构建执行计划（数据流图）
        // execute() 才是真正将作业提交到集群运行的触发点
        // 参数是作业名称，会显示在 Web UI 和日志中
        env.execute("Module 1 - WordCount Simple");
    }
    
    /**
     * 自定义 FlatMapFunction: 分词器
     * 
     * FlatMapFunction<T, O>:
     *   T = 输入类型 (String: 一行文本)
     *   O = 输出类型 (Tuple2<String, Integer>: 单词和计数)
     * 
     * 为什么用 Tuple2?
     *   Flink 内置了 Tuple1 ~ Tuple25，用于表示多字段记录
     *   Tuple2<String, Integer> 类似于 Pair<String, Integer>
     *   f0 表示第一个字段，f1 表示第二个字段
     */
    public static class Tokenizer implements FlatMapFunction<String, Tuple2<String, Integer>> {
        
        @Override
        public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
            // 按非单词字符分割（正则 \W+ 匹配任何非字母数字下划线的字符序列）
            // 并转为小写，确保 "Hello" 和 "hello" 被视为同一个词
            for (String word : value.toLowerCase().split("\\W+")) {
                // 过滤空字符串
                if (word.length() > 0) {
                    // collect 发射一个元素到下游
                    out.collect(new Tuple2<>(word, 1));
                }
            }
        }
    }
}
```

**运行方式**：

```bash
cd /root/flink-study/projects
mvn compile exec:java -Dexec.mainClass="com.example.module01.WordCountSimple"
```

**预期输出**：

```
(hello, 1)
(world, 1)
(hello, 2)    <- "hello" 出现了第二次，累计为 2
(flink, 1)
...
(flink, 4)    <- 最终 "flink" 出现 4 次
```

### 1.6.3 完整版 WordCount（Socket 实时流）

这个版本从 Socket 读取实时数据，更接近生产场景。

**文件**: `src/main/java/com/example/module01/WordCount.java`

```java
package com.example.module01;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.TumblingProcessingTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

/**
 * Module 1 实验: Socket WordCount
 * 
 * 从 Socket 读取实时文本流，每 5 秒统计一次单词出现次数
 * 
 * 运行步骤:
 *   1. 终端1: nc -lk 9999
 *   2. 终端2: 运行本程序
 *   3. 在终端1输入文字，观察终端2的输出
 */
public class WordCount {
    
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = 
            StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        
        // 从 Socket 读取数据 (localhost:9999)
        // 如果 Socket 未启动，程序会等待连接
        DataStream<String> text = env.socketTextStream("localhost", 9999);
        
        DataStream<Tuple2<String, Integer>> wordCounts = text
            .flatMap(new Tokenizer())
            .keyBy(value -> value.f0)
            
            // 与简化版的区别：这里加了窗口！
            // TumblingProcessingTimeWindows.of(Time.seconds(5))
            //   - Tumbling: 滚动窗口，窗口之间不重叠
            //   - ProcessingTime: 基于机器的系统时间
            //   - 每 5 秒统计一次这 5 秒内输入的单词
            .window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
            .sum(1);
        
        wordCounts.print();
        env.execute("Module 1 - Socket Window WordCount");
    }
    
    public static class Tokenizer implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
            for (String word : value.toLowerCase().split("\\W+")) {
                if (word.length() > 0) {
                    out.collect(new Tuple2<>(word, 1));
                }
            }
        }
    }
}
```

**为什么需要窗口？**

在简化版中，数据是有限的（4 行文本），所以可以直接全局求和。但在 Socket 版本中，数据是**无限的**——你不知道用户什么时候停止输入。如果不加窗口，Flink 会永远等待，永远不会输出结果。

窗口给无界数据流提供了一个"有界"的视角：*"每 5 秒，把这 5 秒内收到的数据统计一下，输出结果，然后清空，开始下一个 5 秒"*。

---

## 1.7 Flink 程序骨架完全拆解

每个 Flink 程序都遵循固定的五步法。理解这五步，你就理解了 Flink 编程的骨架：

```java
/*
 * ┌─────────────────────────────────────────────────────────────────┐
 * │  Flink 程序五步法                                               │
 * ├─────────────────────────────────────────────────────────────────┤
 * │                                                                 │
 * │  Step 1: 获取执行环境                                          │
 * │  ─────────────────────────────────                             │
 * │  StreamExecutionEnvironment env = ...                          │
 * │                                                                 │
 * │  • 本地运行: getExecutionEnvironment()                         │
 * │  • 远程集群: createRemoteEnvironment(host, port)               │
 * │  • 设置全局参数: 并行度、重启策略、时间语义等                   │
 * │                                                                 │
 * │  Step 2: 读取数据源 (Source)                                    │
 * │  ─────────────────────────────────                             │
 * │  DataStream<T> stream = env.addSource(...)                     │
 * │                                                                 │
 * │  • 内置: fromElements(), fromCollection(), readTextFile()      │
 * │  • Socket: socketTextStream()                                  │
 * │  • 连接器: KafkaSource, PulsarSource, RabbitMQ...              │
 * │  • 自定义: 实现 SourceFunction 或 ParallelSourceFunction       │
 * │                                                                 │
 * │  Step 3: 数据转换 (Transformation)                              │
 * │  ─────────────────────────────────                             │
 * │  DataStream<R> result = stream                                 │
 * │      .map(...)                                                 │
 * │      .filter(...)                                              │
 * │      .keyBy(...)                                               │
 * │      .window(...)                                              │
 * │      .aggregate(...);                                          │
 * │                                                                 │
 * │  • 单流操作: map, filter, flatMap, keyBy, reduce               │
 * │  • 窗口操作: window, windowAll, trigger, evictor               │
 * │  • 多流操作: union, connect, join, coGroup                     │
 * │  • 状态操作: process, keyedProcess                             │
 * │                                                                 │
 * │  Step 4: 输出结果 (Sink)                                        │
 * │  ─────────────────────────────────                             │
 * │  result.print()                                                │
 * │  result.addSink(new KafkaSink<>(...))                          │
 * │                                                                 │
 * │  • 调试: print(), writeAsText()                                │
 * │  • 连接器: KafkaSink, JdbcSink, ElasticsearchSink...           │
 * │  • 自定义: 实现 SinkFunction 或 RichSinkFunction               │
 * │                                                                 │
 * │  Step 5: 触发执行                                              │
 * │  ─────────────────────────────────                             │
 * │  env.execute("Job Name");                                      │
 * │                                                                 │
 * │  • Flink 是惰性执行的！                                        │
 * │  • execute() 之前只是构建执行图（JobGraph）                    │
 * │  • execute() 将 JobGraph 提交到 JobManager                     │
 * │  • 作业名称会显示在 Web UI 和日志中                            │
 * │                                                                 │
 * └─────────────────────────────────────────────────────────────────┘
 */
```

### 1.7.1 惰性执行（Lazy Evaluation）

这是 Flink 最核心的特性之一，也是新手最容易困惑的地方。

```java
// 这段代码做了什么？
DataStream<Integer> result = numbers
    .map(x -> x * 2)        // 翻倍
    .filter(x -> x > 10)    // 过滤
    .keyBy(x -> x % 2)      // 分组
    .sum(0);                 // 求和

// 答案是：什么都没做（至少从执行角度）
// 这些调用只是在内存中构建了一个"执行计划"，类似于 SQL 的查询计划

// 只有在调用 execute() 后，Flink 才会:
// 1. 将执行计划优化为 JobGraph
// 2. 提交到 JobManager
// 3. JobManager 将 JobGraph 转换为 ExecutionGraph
// 4. 调度到 TaskManager 执行

env.execute("My Job");  // ← 真正触发执行的点
```

**为什么需要惰性执行？**

1. **优化机会**：Flink 可以看到整个执行图，进行算子融合（Operator Chain）、优化执行计划
2. **统一提交**：一次性提交完整作业，避免中间状态的混乱
3. **分布式协调**：需要协调多个 TaskManager 的启动和连接

### 1.7.2 执行图的三层转换

Flink 作业在提交后会经历三层图的转换：

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Flink 执行图转换流程                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   StreamGraph                    JobGraph                    ExecutionGraph │
│   (逻辑流图)                      (作业图)                    (执行图)        │
│                                                                             │
│   ┌─────────┐                   ┌─────────┐                  ┌─────────┐   │
│   │ Source  │                   │ Source  │                  │ Source  │   │
│   └────┬────┘                   └────┬────┘                  └────┬────┘   │
│        │                             │                            │        │
│        ▼                             ▼                            ▼        │
│   ┌─────────┐    链化优化      ┌─────────┐    并行化拆分     ┌─────────┐   │
│   │  Map    │  ═══════════▶   │ Map     │  ═══════════▶   │ Map-0   │   │
│   └────┬────┘   (chain)       └────┬────┘   (parallel=2)   │ Map-1   │   │
│        │                             │                      └─────────┘   │
│        ▼                             ▼                            │        │
│   ┌─────────┐                   ┌─────────┐                       ▼        │
│   │ KeyBy   │                   │ KeyBy   │                  ┌─────────┐   │
│   └────┬────┘                   └────┬────┘                  │ KeyBy-0 │   │
│        │                             │                       │ KeyBy-1 │   │
│        ▼                             ▼                       └─────────┘   │
│   ┌─────────┐                   ┌─────────┘                            │   │
│   │  Sum    │                   │                                       ▼   │
│   └────┬────┘                   │                                  ┌────────┐│
│        │                         │                                  │ Sum-0  ││
│        ▼                         ▼                                  │ Sum-1  ││
│   ┌─────────┐                   ┌─────────┐                         └────────┘│
│   │  Sink   │                   │  Sink   │                               │   │
│   └─────────┘                   └─────────┘                               │   │
│                                                                           ▼   │
│                                                                      ┌────────┐│
│                                                                      │ Sink-0 ││
│                                                                      │ Sink-1 ││
│                                                                      └────────┘│
│                                                                             │
│   • StreamGraph: 用户代码直接生成，一对一对应代码中的算子                      │
│   • JobGraph:    将可链化的算子合并（chain），减少网络传输                   │
│   • ExecutionGraph: 将每个算子按并行度展开为多个 subtask                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 1.8 DataStream API 核心操作详解

### 1.8.1 基础转换操作

#### Map：一对一映射

```java
// 将每个数字翻倍
DataStream<Integer> doubled = numbers.map(x -> x * 2);

// 将字符串转为长度
DataStream<Integer> lengths = strings.map(String::length);

// 使用 RichMapFunction（可以访问 RuntimeContext，获取状态、度量等）
DataStream<Result> result = stream.map(new RichMapFunction<Input, Result>() {
    @Override
    public Result map(Input value) {
        // 可以访问 getRuntimeContext() 获取并行度、子任务索引等
        int subtaskIndex = getRuntimeContext().getIndexOfThisSubtask();
        return new Result(value, subtaskIndex);
    }
});
```

#### Filter：过滤

```java
// 只保留正数
DataStream<Integer> positive = numbers.filter(x -> x > 0);

// 过滤空字符串
DataStream<String> nonEmpty = strings.filter(s -> s != null && !s.isEmpty());
```

#### FlatMap：一对多展开

FlatMap 是 Map 的增强版：一个输入元素可以产生零个、一个或多个输出元素。

```java
// 将句子拆分为单词
DataStream<String> words = sentences.flatMap(
    (String sentence, Collector<String> out) -> {
        for (String word : sentence.split(" ")) {
            out.collect(word);  // 发射多个元素
        }
    }
);

// 过滤 + 展开同时进行
DataStream<String> validWords = lines.flatMap(
    (String line, Collector<String> out) -> {
        if (!line.startsWith("#")) {  // 跳过注释行
            for (String word : line.split(" ")) {
                if (word.length() > 2) {  // 只保留长度大于2的词
                    out.collect(word.toLowerCase());
                }
            }
        }
    }
);
```

### 1.8.2 分组与聚合

#### KeyBy：按 Key 分组

`keyBy()` 是 Flink 中最重要的操作之一。它将数据流按指定的 key 分区，相同 key 的数据会被路由到同一个并行实例。

```java
// 按用户 ID 分组
KeyedStream<Event, String> keyed = events.keyBy(event -> event.userId);

// 按多个字段分组 (使用 Tuple)
KeyedStream<Event, Tuple2<String, Integer>> keyed = events
    .keyBy(event -> Tuple2.of(event.city, event.category));

// ⚠️ 重要: keyBy 之后的算子才能做聚合（sum/reduce/window）
// 因为聚合需要在同一个 key 的数据上进行
```

**KeyBy 的底层实现**：

Flink 使用 `hash(key) % parallelism` 决定数据路由到哪个 subtask。这意味着：
- 相同 key 的数据总是到同一个 subtask（保证聚合正确性）
- 不同 key 可能到同一个 subtask（哈希冲突）
- key 的哈希值必须稳定（不要用随机数作为 key）

#### Reduce：归约聚合

```java
// 求每个用户的累计消费金额
DataStream<UserSpending> totalSpending = spendings
    .keyBy(s -> s.userId)
    .reduce((a, b) -> {
        // a 是之前的累计结果，b 是新到来的数据
        return new UserSpending(a.userId, a.amount + b.amount);
    });
```

#### Sum / Min / Max

```java
// 对 Tuple 的第2个字段求和
KeyedStream<Tuple2<String, Integer>, String> keyed = words
    .keyBy(value -> value.f0);
    
DataStream<Tuple2<String, Integer>> summed = keyed.sum(1);  // 对第2个字段求和

// 注意: sum/min/max 只能用于 Tuple 或 POJO 类型
// 对于复杂类型，需要使用 reduce 或 aggregate
```

### 1.8.3 窗口操作入门

窗口是流处理的核心概念，Flink 提供了丰富的窗口类型：

```
窗口类型概览:

┌──────────────────────────────────────────────────────────────────────────┐
│  按时间划分 (Time Windows)                                                │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  滚动窗口 (Tumbling Window)                                              │
│  ┌──────┐┌──────┐┌──────┐┌──────┐                                       │
│  │ 0-5s ││5-10s ││10-15s││15-20s│   窗口不重叠，固定大小                  │
│  └──────┘└──────┘└──────┘└──────┘                                       │
│                                                                          │
│  滑动窗口 (Sliding Window)                                               │
│  ┌──────┐                                                                │
│  │0-10s │┌──────┐                                                        │
│  └──────┘│3-13s │┌──────┐   窗口重叠，滑动前进                            │
│          └──────┘│6-16s │   (size=10s, slide=3s)                        │
│                  └──────┘                                                │
│                                                                          │
│  会话窗口 (Session Window)                                               │
│  ┌────┐    ┌──────────┐  ┌────┐                                         │
│  │活动│gap │   活动   │gap│活动│   根据活动间隙动态划分                    │
│  └────┘    └──────────┘  └────┘                                         │
│                                                                          │
├──────────────────────────────────────────────────────────────────────────┤
│  按数据量划分 (Count Windows) - 使用较少                                  │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  滚动计数窗口: 每 N 条数据一个窗口                                        │
│  滑动计数窗口: 每 M 条数据滑动一次，窗口大小 N 条                         │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

代码示例：

```java
import org.apache.flink.streaming.api.windowing.assigners.*;
import org.apache.flink.streaming.api.windowing.time.Time;

// 滚动窗口: 每 10 秒统计一次
stream.keyBy(...)
      .window(TumblingProcessingTimeWindows.of(Time.seconds(10)))
      .aggregate(...);

// 滑动窗口: 每 5 秒统计过去 15 秒的数据
stream.keyBy(...)
      .window(SlidingProcessingTimeWindows.of(Time.seconds(15), Time.seconds(5)))
      .aggregate(...);

// 会话窗口: 活动间隔超过 30 秒则关闭窗口
stream.keyBy(...)
      .window(ProcessingTimeSessionWindows.withGap(Time.seconds(30)))
      .aggregate(...);
```

### 1.8.4 多流操作

#### Union：合并同类型流

```java
// 将两个数据流合并（要求相同类型）
DataStream<Event> merged = stream1.union(stream2);

// 可以合并多个
DataStream<Event> all = stream1.union(stream2, stream3, stream4);

// ⚠️ Union 只是物理合并，不会去重
// 如果 stream1 和 stream2 有相同数据，结果里会有两份
```

#### Connect：连接不同类型流

```java
// 连接两个不同类型的流
ConnectedStreams<Order, Payment> connected = orders.connect(payments);

// 需要分别处理两种类型的数据
DataStream<Result> result = connected
    .process(new CoProcessFunction<Order, Payment, Result>() {
        // 处理 Order 类型的数据
        @Override
        public void processElement1(Order order, Context ctx, Collector<Result> out) {
            // 暂存 order，等待匹配的 payment
        }
        
        // 处理 Payment 类型的数据
        @Override
        public void processElement2(Payment payment, Context ctx, Collector<Result> out) {
            // 查找匹配的 order，输出结果
        }
    });
```

### 1.8.5 数据源（Source）详解

```java
// 1. 从集合创建（测试用）
DataStream<Integer> numbers = env.fromElements(1, 2, 3, 4, 5);
DataStream<String> lines = env.fromCollection(Arrays.asList("a", "b", "c"));

// 2. 从文件创建
DataStream<String> lines = env.readTextFile("/path/to/file.txt");

// 3. 从 Socket 创建（测试/演示用）
DataStream<String> lines = env.socketTextStream("localhost", 9999);

// 4. 自定义数据源
DataStream<Event> events = env.addSource(new MySourceFunction());

// 5. Kafka 数据源（生产环境最常用）- Module 6 会详细讲解
KafkaSource<String> source = KafkaSource.<String>builder()
    .setBootstrapServers("kafka:9092")
    .setTopics("input-topic")
    .setGroupId("flink-consumer")
    .setStartingOffsets(OffsetsInitializer.earliest())
    .setValueOnlyDeserializer(new SimpleStringSchema())
    .build();
DataStream<String> kafkaStream = env.fromSource(source, WatermarkStrategy.noWatermarks(), "Kafka Source");
```

### 1.8.6 数据输出（Sink）详解

```java
// 1. 打印到控制台（调试）
stream.print();                          // 带前缀输出
stream.printToErr();                     // 输出到 stderr

// 2. 写入文件
stream.writeAsText("/output/path.txt");  // ⚠️ 已废弃，生产用 FileSink

// 3. 自定义 Sink
stream.addSink(new MySinkFunction());

// 4. Kafka Sink（生产常用）
KafkaSink<String> sink = KafkaSink.<String>builder()
    .setBootstrapServers("kafka:9092")
    .setRecordSerializer(KafkaRecordSerializationSchema.builder()
        .setTopic("output-topic")
        .setValueSerializationSchema(new SimpleStringSchema())
        .build())
    .build();
stream.sinkTo(sink);
```

---

## 1.9 并行度与执行图

### 1.9.1 并行度的层次结构

```
并行度设置优先级（高 → 低）:

┌─────────────────────────────────────────────────────────────┐
│  1. 算子级别: stream.map(...).setParallelism(2)            │
│     ↑ 优先级最高，只影响这个算子                             │
├─────────────────────────────────────────────────────────────┤
│  2. 执行环境: env.setParallelism(4)                          │
│     ↑ 影响所有未单独设置并行度的算子                         │
├─────────────────────────────────────────────────────────────┤
│  3. 集群配置: flink-conf.yaml 中 parallelism.default         │
│     ↑ 集群级别的默认值                                       │
├─────────────────────────────────────────────────────────────┤
│  4. 默认值: 1                                                │
│     ↑ 如果没有其他设置                                       │
└─────────────────────────────────────────────────────────────┘
```

### 1.9.2 Task Slot 与并行度的关系

```
场景: TaskManager 有 3 个 Slot，作业并行度 = 5

┌─────────────────────────────────────────────────────────────┐
│                    集群资源                                   │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │  TaskManager-1  │  │  TaskManager-2  │                   │
│  │  ┌───────────┐  │  │  ┌───────────┐  │                   │
│  │  │  Slot 0   │  │  │  │  Slot 0   │  │                   │
│  │  │ [Subtask] │  │  │  │ [Subtask] │  │                   │
│  │  └───────────┘  │  │  └───────────┘  │                   │
│  │  ┌───────────┐  │  │  ┌───────────┐  │                   │
│  │  │  Slot 1   │  │  │  │  Slot 1   │  │                   │
│  │  │ [Subtask] │  │  │  │ [Subtask] │  │                   │
│  │  └───────────┘  │  │  └───────────┘  │                   │
│  │  ┌───────────┐  │  │  ┌───────────┐  │                   │
│  │  │  Slot 2   │  │  │  │  Slot 2   │  │  ← 需要更多 TM!  │
│  │  │ [Subtask] │  │  │  │ [Subtask] │  │                   │
│  │  └───────────┘  │  │  └───────────┘  │                   │
│  └─────────────────┘  └─────────────────┘                   │
│                                                             │
│  总计 Slot 数 = 6，作业需要 5 个 Slot → 可以运行            │
│  如果并行度 = 7，需要 7 个 Slot → 资源不足，作业等待        │
└─────────────────────────────────────────────────────────────┘
```

### 1.9.3 Operator Chain（算子链）

Flink 会自动将某些算子融合到同一个线程中执行，这就是 **Operator Chain**：

```
优化前 (无 chain):
┌─────────┐  网络  ┌─────────┐  网络  ┌─────────┐
│ Source  │───────▶│  Map    │───────▶│  Sink   │
└─────────┘        └─────────┘        └─────────┘
  线程1             线程2              线程3
  
  问题: Source 输出到 Map 需要序列化、网络传输、反序列化

优化后 (chain):
┌─────────────────────────────────────────────┐
│  ┌─────────┐  内存传递  ┌─────────┐  网络   │
│  │ Source  │───────────▶│  Map    │────────▶│──▶ Sink
│  └─────────┘ (无序列化)  └─────────┘         │
│                    同一个线程                │
└─────────────────────────────────────────────┘

好处:
• 减少序列化/反序列化开销
• 减少网络/线程间通信
• 提高吞吐量
```

**什么情况下不会 chain？**

1. **不同并行度**：`map().setParallelism(2)` 后接 `filter().setParallelism(4)`
2. **显式禁用**：`.disableChaining()` 或 `env.disableOperatorChaining()`
3. **重分区操作**：`keyBy()`、`broadcast()`、`rebalance()` 等会打断 chain
4. **不同的 Slot 共享组**：`.slotSharingGroup("group1")`

---

## 1.10 常见问题与调试技巧

### 1.10.1 常见错误

#### 错误1：忘记调用 `execute()`

```java
// ❌ 错误：程序直接退出，没有任何输出
public static void main(String[] args) {
    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
    env.fromElements(1, 2, 3).print();
    // 忘记调用 env.execute()！
}

// ✅ 正确
public static void main(String[] args) throws Exception {
    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
    env.fromElements(1, 2, 3).print();
    env.execute("My Job");  // 必须调用！
}
```

#### 错误2：`keyBy()` 后直接使用 `map()` 而非聚合

```java
// ❌ 错误：keyBy 后接 map 不会做聚合
stream.keyBy(...).map(...);  // 只是对分组后的数据做映射，不会合并

// ✅ 正确：keyBy 后接聚合操作
stream.keyBy(...).reduce(...);
stream.keyBy(...).sum(...);
stream.keyBy(...).window(...).aggregate(...);
```

#### 错误3：在 Socket 程序中使用 Event Time

```java
// ❌ 错误：Socket 数据没有时间戳，使用 Event Time 窗口不会触发
stream.assignTimestampsAndWatermarks(...)
      .window(TumblingEventTimeWindows.of(Time.seconds(5)))

// ✅ 正确：Socket 测试使用 Processing Time
stream.window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
```

### 1.10.2 调试技巧

```java
// 技巧1: 打印执行计划（不运行）
System.out.println(env.getExecutionPlan());

// 技巧2: 给算子命名，在 Web UI 中更容易识别
stream.map(new MyMapper()).name("ParseJSON")
      .filter(new MyFilter()).name("FilterValid")
      .keyBy(...).name("GroupByUser")
      .window(...).name("10sWindow");

// 技巧3: 设置全局 JobListener 观察生命周期
env.registerJobListener(new JobListener() {
    @Override
    public void onJobSubmitted(JobClient jobClient, Throwable throwable) {
        System.out.println("作业已提交: " + jobClient.getJobID());
    }
    
    @Override
    public void onJobExecuted(JobExecutionResult jobExecutionResult, Throwable throwable) {
        System.out.println("作业执行完成");
    }
});

// 技巧4: 使用 uid 为算子设置唯一标识（用于 Checkpoint 恢复）
stream.map(...).uid("my-map-operator")
      .keyBy(...).uid("my-keyby")
```

---

## 1.11 模块练习

### 编程练习 P1-1：实时平均值计算

**需求**：编写一个 Flink 程序，从 Socket 读取数字流，实时计算每 10 秒窗口内的平均值。

**提示**：
- 使用 `AggregateFunction` 自定义平均值计算
- 平均值 = 总和 / 个数，需要维护两个状态值
- 参考项目中的 `AverageCalculator.java`

<details>
<summary>参考答案（先自己尝试！）</summary>

```java
static class AverageAggregate implements 
    org.apache.flink.api.common.functions.AggregateFunction<Integer, AverageAccumulator, Double> {
    
    @Override
    public AverageAccumulator createAccumulator() {
        return new AverageAccumulator();
    }
    
    @Override
    public AverageAccumulator add(Integer value, AverageAccumulator acc) {
        acc.sum += value;
        acc.count++;
        return acc;
    }
    
    @Override
    public Double getResult(AverageAccumulator acc) {
        return acc.count == 0 ? 0.0 : (double) acc.sum / acc.count;
    }
    
    @Override
    public AverageAccumulator merge(AverageAccumulator a, AverageAccumulator b) {
        a.sum += b.sum;
        a.count += b.count;
        return a;
    }
}
```
</details>

### 编程练习 P1-2：日志过滤与输出

**需求**：编写程序从 Socket 读取日志行，过滤掉包含 "ERROR" 的行，将结果写入文本文件。

**提示**：
- 使用 `filter()` 过滤 ERROR 日志
- 使用 `writeAsText()` 或 `FileSink` 输出到文件
- 注意文件路径需要是 TaskManager 可访问的

### 思考题 Q1-1：惰性执行的原理

为什么 Flink 要采用惰性执行？`env.execute()` 调用前后，Flink 内部发生了哪些事情？

### 思考题 Q1-2：Slot 与并行度

一个 TaskManager 有 3 个 slot，作业设置了全局并行度 5：
1. 这个作业能正常运行吗？需要几个 TaskManager？
2. 如果另一个作业也设置了并行度 5，两个作业能同时运行吗？
3. 什么情况下算子不会被 chain 在一起？

---

## 1.12 延伸阅读与参考

### 官方资源

- [Apache Flink 官方文档](https://nightlies.apache.org/flink/flink-docs-stable/)
- [Flink 中文文档](https://nightlies.apache.org/flink/flink-docs-stable/zh/)
- [Flink 实战案例](https://flink.apache.org/usecases.html)

### 推荐文章

- 《Streaming 101》by Tyler Akidau（Google）—— 理解流处理的基础概念
- 《The Dataflow Model》—— Google 的流批统一论文，Flink 的重要参考

### 相关论文

- **Chandy-Lamport 算法**: "Distributed Snapshots: Determining Global States of Distributed Systems" (1985)
- **Flink 的架构论文**: "Apache Flink™: Stream and Batch Processing in a Single Engine" (IEEE 2015)

---

## 本章小结

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Module 1 核心要点                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. 流处理处理的是无界数据流，不是"更快的批处理"                            │
│                                                                             │
│  2. Flink 是真正的流处理引擎（原生逐条处理），不是微批                        │
│                                                                             │
│  3. Flink 的核心优势: 精确一次、事件时间、分布式快照、有状态计算             │
│                                                                             │
│  4. 架构: JobManager (调度/协调) + TaskManager (执行)                      │
│                                                                             │
│  5. 编程五步法: Environment → Source → Transformation → Sink → Execute   │
│                                                                             │
│  6. Flink 是惰性执行的，execute() 才是真正触发运行                           │
│                                                                             │
│  7. 并行度决定算子同时运行的实例数，Slot 是资源隔离单元                      │
│                                                                             │
│  8. Operator Chain 自动优化，减少序列化和网络开销                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

> **下一步**: 完成本章练习后，可以说"开始测试"进行 Module 1 的测验。测验通过（≥80%）后，进入 Module 2: DataStream API 核心编程。

---

*本文档是 Flink 学习路径 Module 1 的完整参考资料。如果在学习过程中有任何疑问，随时提问。*
