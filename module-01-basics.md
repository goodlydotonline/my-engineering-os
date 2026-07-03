# Apache Flink 学习笔记 · Module 01：基础入门

> **目标读者**：初次接触 Flink 的开发者  
> **前置知识**：Java/Scala 基础、基本的数据库与分布式概念  
> **学习时长**：建议 4–6 小时（含动手实验）  
> **版本参考**：Flink 1.18+（概念兼容 1.12+）

---

## 目录

1. [1.1 什么是 Apache Flink](#11-什么是-apache-flink)
2. [1.2 核心概念与架构](#12-核心概念与架构)
3. [1.3 第一个 Flink 程序：骨架解析](#13-第一个-flink-程序骨架解析)
4. [1.4 DataStream API 速查](#14-datastream-api-速查)
5. [附录：环境搭建速通](#附录环境搭建速通)

---

## 1.1 什么是 Apache Flink

### 1.1.1 一句话定义

**Apache Flink 是一个开源的分布式流处理框架，它以"流优先"（Stream-First）的架构设计，同时支持流处理和批处理，并在两者之上提供统一的编程模型。**

这句话里的每个词都值得拆开理解：

- **开源**：Apache 顶级项目，社区活跃，阿里云、Ververica 等公司有商业化产品
- **分布式**：天然支持多机并行，水平扩展至数千节点
- **流处理**：核心能力是对**无界数据流**（Unbounded Stream）进行持续计算
- **批处理**：通过将有限数据集视为"有界流"来实现批处理
- **统一**：同一套 API、同一套运行时、同一套语义，不用为批和流写两套代码

### 1.1.2 为什么 Flink 重要？

#### 流处理的演进

| 时代 | 代表技术 | 特点 | 局限 |
|:---|:---|:---|:---|
| 第一代 | Storm | 纯实时，低延迟 | 语义弱（At-least-once），无状态管理 |
| 第二代 | Spark Streaming | 微批（Micro-batch）模拟流 | 延迟高（秒级），语义靠外部系统保证 |
| **第三代** | **Flink** | **原生流、精确一次、有状态** | 学习曲线较陡 |

Flink 的核心竞争力在于：**它在流处理引擎的底层设计上就是"原生流"（Native Streaming），而非把批切成小块来模拟流。** 这个区别决定了它在延迟、语义和状态管理上的根本优势。

#### 典型应用场景

```
┌─────────────────────────────────────────────────────────────┐
│                    Flink 典型应用场景                        │
├─────────────────────────────────────────────────────────────┤
│  🔄 实时 ETL          →  数据清洗、格式转换、路由分发        │
│  📊 实时报表/看板      →  分钟级/秒级业务指标监控            │
│  🚨 异常检测          →  欺诈识别、系统告警、风控决策        │
│  🎯 实时推荐          →  电商/视频个性化推荐                 │
│  🌊 事件驱动应用      →  复杂事件处理（CEP）、IoT 规则引擎   │
│  📈 实时机器学习      →  在线特征工程、模型在线更新          │
└─────────────────────────────────────────────────────────────┘
```

### 1.1.3 Flink vs Spark：核心差异

| 维度 | Apache Flink | Apache Spark |
|:---|:---|:---|
| **计算模型** | 原生流（Native Streaming） | 微批（Micro-batch） |
| **延迟** | 毫秒级 | 秒级（受批大小影响） |
| **语义保证** | Exactly-once（内置） | Exactly-once（依赖外部） |
| **状态管理** | 内置、分布式、可快照 | 依赖外部存储（如 Redis） |
| **事件时间处理** | 原生支持 Watermark 机制 | Structured Streaming 后支持 |
| **迭代计算** | 有限支持 | GraphX 等迭代场景更成熟 |
| **生态成熟度** | 流处理生态强 | 批处理/机器学习生态更广 |

> 💡 **选型建议**：如果业务以实时流处理为核心，选 Flink；如果批处理为主、偶尔有流需求，Spark 可能更省维护成本。

### 1.1.4 Flink 生态概览

```
                         ┌─────────────────┐
                         │   用户应用层     │
                         │  Table API / SQL │
                         │  DataStream API  │
                         │  DataSet API     │
                         └────────┬────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
    ┌─────────▼────────┐ ┌────────▼────────┐ ┌───────▼───────┐
    │  Table API/SQL   │ │  DataStream API │ │   CEP 库      │
    │  （声明式/关系型）  │ │  （流处理核心）  │ │ （复杂事件）   │
    └──────────────────┘ └─────────────────┘ └───────────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │   Flink 运行时层     │
              │  - 分布式执行引擎     │
              │  - 状态后端（RocksDB/ │
              │    Heap/Memory）     │
              │  - Checkpoint/Savepoint│
              │  - 调度与容错        │
              └─────────────────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │   资源管理层         │
              │  Standalone / YARN  │
              │  Kubernetes / Mesos │
              └─────────────────────┘
```

---

## 1.2 核心概念与架构

### 1.2.1 DataStream：Flink 的核心抽象

在 Flink 中，**一切皆是流**。DataStream API 是最基础、最灵活的编程接口。

```java
// 一个 DataStream 代表一个元素序列，这些元素是...
// - 无界的（Unbounded）：数据持续产生，没有明确的结束
// - 有序的或乱序的：元素可能不按时间顺序到达
// - 可分区并行处理的：可以拆分到多个子任务并行执行

DataStream<Event> stream = env.addSource(new KafkaSource<>(...));
```

### 1.2.2 时间与 Watermark

时间是流处理中最复杂也最重要的概念。Flink 区分三种时间：

```
┌────────────────────────────────────────────────────────────────┐
│                        三种时间语义                            │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Event Time（事件时间）    Processing Time（处理时间）    Ingestion Time（摄入时间）│
│  ─────────────────────     ──────────────────────      ────────────────────    │
│  数据产生的时间戳          算子处理数据时的系统时间      数据进入 Flink 的时间    │
│  ←────── 最常用 ──────→                                        │
│                                                                │
│  优点：结果可复现、不受处理速度影响                              │
│  挑战：需要处理乱序和延迟数据                                    │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

#### Watermark（水位线）：解决乱序的钥匙

```
时间轴 →
│
│    事件: ①──②────⑤──③──④──⑥──⑦──⑧──⑨
│    时间: 1   2    5   3   4   6   7   8   9
│                    ↑
│              乱序到达！⑤ 比 ③④ 早到
│
│    Watermark(2) ──────────→  表示"2 之前的数据应该都到齐了"
│         │
│    Watermark(5) ─────────────────────→  延迟容忍 3 个单位
│         │
│         └─ 触发窗口计算，输出 [1,5) 区间的结果
│
│    ⚠️ 延迟超过 Watermark 的数据 → 归入侧输出流（Side Output）
```

```java
// 代码示例：设置 Watermark 策略
stream.assignTimestampsAndWatermarks(
    WatermarkStrategy
        .<Event>forBoundedOutOfOrderness(Duration.ofSeconds(5))  // 允许 5 秒乱序
        .withTimestampAssigner((event, timestamp) -> event.getEventTime())
);
```

### 1.2.3 状态（State）

**状态是区分"简单转换"和"复杂计算"的分水岭。**

```
无状态转换                    有状态转换
─────────────────           ─────────────────
map(x -> x * 2)             keyBy(id).sum("amount")
filter(x -> x > 0)          需要记住每个 key 的累计值
                            故障恢复时必须恢复这个值
```

Flink 提供多种状态类型：

| 状态类型 | 适用场景 | 特点 |
|:---|:---|:---|
| `ValueState<T>` | 单值状态（如计数器、最新值） | 每个 key 一个值 |
| `ListState<T>` | 列表状态（如窗口内所有元素） | 可追加、可遍历 |
| `MapState<K, V>` | Map 结构状态 | key-value 存储 |
| `ReducingState<T>` | 自动归约的状态 | 只保留归约结果，省内存 |
| `AggregatingState<IN, OUT>` | 自动聚合的状态 | 增量聚合 |

```java
// 状态声明示例
private ValueState<Long> countState;

@Override
public void open(Configuration parameters) {
    StateTtlConfig ttlConfig = StateTtlConfig
        .newBuilder(Time.hours(24))
        .setUpdateType(OnCreateAndWrite)
        .build();
    
    countState = getRuntimeContext().getState(
        new ValueStateDescriptor<>("count", Types.LONG)
    );
}
```

### 1.2.4 Checkpoint 与容错

```
┌─────────────────────────────────────────────────────────────┐
│                    Checkpoint 机制                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ① Checkpoint Coordinator 触发快照                         │
│            │                                                │
│   ② 所有 Source 插入 Checkpoint Barrier（栅栏）             │
│            │                                                │
│   ③ Barrier 流经整个 DAG，算子遇到时快照本地状态              │
│            │                                                │
│  ④ 状态写入持久化存储（HDFS/S3/本地磁盘）                     │
│            │                                                │
│   ⑤ 所有算子确认后，Checkpoint 完成                         │
│                                                             │
│   🔄 故障时：从最新 Checkpoint 恢复，状态精确回到故障前一刻   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

关键参数：

```java
env.enableCheckpointing(60000);           // 每 60 秒触发一次 Checkpoint
env.getCheckpointConfig().setCheckpointingMode(
    CheckpointingMode.EXACTLY_ONCE        // 精确一次语义
);
env.getCheckpointConfig().setMinPauseBetweenCheckpoints(30000); // 最小间隔
env.setStateBackend(new RocksDBStateBackend("hdfs://..."));     // 状态后端
```

### 1.2.5 部署模式

| 模式 | 适用场景 | 特点 |
|:---|:---|:---|
| **Local** | 本地开发调试 | 单 JVM，不用集群 |
| **Standalone** | 小型集群、测试 | Flink 自带集群管理 |
| **YARN** | 已有 Hadoop 生态 | 资源复用，与 MapReduce/Spark 共存 |
| **Kubernetes** | 云原生部署（推荐） | 容器化、弹性伸缩、云厂商集成好 |

---

## 1.3 第一个 Flink 程序：骨架解析

### 1.3.1 完整代码：WordCount

```java
package com.example.flink;

import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

public class WordCount {

    public static void main(String[] args) throws Exception {
        // ═══════════════════════════════════════════════════
        // 步骤 1：创建执行环境（程序的"发动机"）
        // ═══════════════════════════════════════════════════
        final StreamExecutionEnvironment env = 
            StreamExecutionEnvironment.getExecutionEnvironment();
        
        // 设置并行度（决定任务拆分多少份同时执行）
        env.setParallelism(2);
        
        // ═══════════════════════════════════════════════════
        // 步骤 2：读取数据源（Data Source）
        // ═══════════════════════════════════════════════════
        DataStream<String> text = env
            .readTextFile("path/to/input.txt");  // 从文件读（有界，批模式）
            // .socketTextStream("localhost", 9999);  // 从 Socket 读（无界，流模式）
            // .addSource(new FlinkKafkaConsumer<>(...));  // 从 Kafka 读（生产环境）
        
        // ═══════════════════════════════════════════════════
        // 步骤 3：数据转换（Transformations）
        // ═══════════════════════════════════════════════════
        DataStream<Tuple2<String, Integer>> wordCounts = text
            // 3.1 FlatMap：一行拆成多个单词
            .flatMap(new Tokenizer())
            
            // 3.2 按单词分组（相同单词分到同一分区）
            .keyBy(value -> value.f0)
            
            // 3.3 窗口：每 5 秒统计一次
            .window(TumblingEventTimeWindows.of(Time.seconds(5)))
            
            // 3.4 聚合：同一窗口内相同单词计数累加
            .sum(1);
        
        // ═══════════════════════════════════════════════════
        // 步骤 4：输出结果（Data Sink）
        // ═══════════════════════════════════════════════════
        wordCounts.print();  // 打印到控制台（开发调试）
        // wordCounts.addSink(new FlinkKafkaProducer<>(...));  // 写入 Kafka
        // wordCounts.writeAsText("path/to/output");  // 写入文件
        
        // ═══════════════════════════════════════════════════
        // 步骤 5：触发执行（Flink 是懒执行的！）
        // ═══════════════════════════════════════════════════
        env.execute("WordCount Job");  // ⚠️ 没有这行，前面都是"定义"，不会真正运行
    }
    
    // ═══════════════════════════════════════════════════
    // 自定义 FlatMap 函数：分词
    // ═══════════════════════════════════════════════════
    public static class Tokenizer implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
            // 规范化：转小写、按非单词字符分割
            for (String word : value.toLowerCase().split("\\W+")) {
                if (word.length() > 0) {
                    out.collect(new Tuple2<>(word, 1));  // 每个单词输出 (word, 1)
                }
            }
        }
    }
}
```

### 1.3.2 骨架逐层拆解

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Flink 程序骨架                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │  ① 环境（Environment）                                       │   │
│   │     StreamExecutionEnvironment.getExecutionEnvironment()    │   │
│   │     └── 程序的"上下文"：配置、并行度、状态后端、Checkpoint    │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │  ② 数据源（Source）                                          │   │
│   │     env.addSource(...) / env.readTextFile(...)              │   │
│   │     └── 数据的入口：Kafka、Socket、文件、自定义 Source        │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │  ③ 转换（Transformation）── DAG 的核心                       │   │
│   │     .map() .filter() .flatMap() .keyBy() .window() .sum()   │   │
│   │     └── 构建有向无环图（DAG），定义数据如何流动和变换        │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │  ④ 输出（Sink）                                              │   │
│   │     .print() .addSink(...) .writeAsText(...)                │   │
│   │     └── 数据的出口：控制台、Kafka、数据库、文件系统           │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │  ⑤ 执行触发（Trigger）                                       │   │
│   │     env.execute("Job Name")                                  │   │
│   │     └── ⚠️ Flink 是懒执行框架！没有 execute() 什么都不发生    │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.3.3 懒执行（Lazy Evaluation）

这是 Flink（以及 Spark）与常规程序最大的不同：

```java
// 传统程序：顺序执行，这行跑完才跑下一行
list = readFile();
result = process(list);   // ← 立即执行
save(result);             // ← 立即执行

// Flink 程序：构建执行计划，不立即执行
DataStream<String> stream = env.readTextFile(...);  // ← 只是"记录"要读文件
DataStream<Result> result = stream.map(...);         // ← 只是"记录"要 map
result.print();                                       // ← 只是"记录"要打印

// 直到这里，一切才真正开始执行：
env.execute("My Job");  // ← 提交 DAG 到集群，开始调度运行
```

**好处**：
- Flink 能看到完整的执行计划，做全局优化（算子链合并、分区策略优化等）
- 可以在提交前检查计划是否合理

```java
// 查看执行计划（调试时用）
System.out.println(env.getExecutionPlan());
```

### 1.3.4 算子链（Operator Chaining）

```
优化前（4 个独立算子）：
┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐
│source│──→│map  │──→│keyBy│──→│sink │
└─────┘   └─────┘   └─────┘   └─────┘
   ↑         ↑         ↑         ↑
  4 个线程，4 次序列化/反序列化，4 次网络/内存传输

优化后（Flink 自动链合并）：
┌─────────────────────────────┐
│  ┌─────┐┌─────┐  ┌─────┐   │
│  │source→│map  │──→│sink │   │
│  └─────┘└─────┘  └─────┘   │
│        ↑  keyBy  ↑          │
│        └─ 同一 ──┘          │
│        线程内执行            │
└─────────────────────────────┘
   ↑
  1 个线程，减少序列化和线程切换开销
```

可以通过 `env.disableOperatorChaining()` 禁用，或用 `startNewChain()`/`disableChaining()` 细粒度控制。

---

## 1.4 DataStream API 速查

### 1.4.1 基础转换算子

| 算子 | 签名 | 作用 | 类比 |
|:---|:---|:---|:---|
| `map` | `T → R` | 一对一转换 | Stream.map |
| `flatMap` | `T → Collection<R>` | 一对多展开 | Stream.flatMap |
| `filter` | `T → boolean` | 条件过滤 | Stream.filter |
| `keyBy` | `T → K` | 按键分组（产生 KeyedStream） | SQL GROUP BY |
| `reduce` | `(T, T) → T` | 归约聚合 | Stream.reduce |

```java
// map：转换元素
DataStream<Integer> lengths = stream.map(String::length);

// flatMap：展开嵌套结构
DataStream<Word> words = sentences.flatMap(
    (Sentence s, Collector<Word> out) -> s.getWords().forEach(out::collect)
);

// filter：过滤数据
DataStream<Event> errors = stream.filter(e -> e.getLevel() == ERROR);

// keyBy：后续操作（窗口、状态）的前提
KeyedStream<Order, String> keyed = orders.keyBy(Order::getUserId);
```

### 1.4.2 窗口算子

```
┌────────────────────────────────────────────────────────────────┐
│                       窗口类型速查                              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  时间窗口（Time Window）                                        │
│  ┌────────────────────────────────────────────────────────┐   │
│  │  Tumbling（滚动）  │  固定大小、不重叠  │  每小时统计   │   │
│  │  Sliding（滑动）   │  固定大小、可重叠  │  每 5 分钟统计最近 1 小时 │   │
│  │  Session（会话）   │  活动间隔触发      │  用户行为分析 │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                                │
│  计数窗口（Count Window）                                       │
│  ┌────────────────────────────────────────────────────────┐   │
│  │  Tumbling Count    │  每 N 条数据触发   │  每 100 条统计 │   │
│  │  Sliding Count     │  每 N 条统计最近 M 条              │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

```java
// 滚动窗口：每 10 秒统计一次，窗口间不重叠
stream.keyBy(Event::getType)
      .window(TumblingEventTimeWindows.of(Time.seconds(10)))
      .aggregate(new CountAggregate());

// 滑动窗口：每 5 秒输出一次，统计最近 20 秒的数据
stream.keyBy(Event::getType)
      .window(SlidingEventTimeWindows.of(Time.seconds(20), Time.seconds(5)))
      .aggregate(new AverageAggregate());

// 会话窗口：用户 30 秒无活动则关闭窗口
stream.keyBy(Event::getUserId)
      .window(EventTimeSessionWindows.withDynamicGap(
          (element) -> Time.minutes(30)
      ))
      .aggregate(new UserSessionAggregate());
```

### 1.4.3 窗口函数类型

| 函数类型 | 输入 | 输出 | 适用场景 |
|:---|:---|:---|:---|
| `AggregateFunction` | 增量聚合，每条数据触发 | 单个结果 | 计数、求和、最值（省内存） |
| `ReduceFunction` | 两个值合并成一个 | 同类型结果 | 简单归约 |
| `ProcessWindowFunction` | 整个窗口的所有数据 | 任意结果 | 需要访问窗口元数据或全量数据 |
| `WindowFunction`（旧版）| 整个窗口的所有数据 | 任意结果 | 兼容旧代码 |

```java
// AggregateFunction：增量聚合，性能好
public class AverageAggregate implements AggregateFunction<Event, AverageAccumulator, Double> {
    @Override
    public AverageAccumulator createAccumulator() { return new AverageAccumulator(); }
    
    @Override
    public AverageAccumulator add(Event value, AverageAccumulator acc) {
        acc.sum += value.getValue();
        acc.count++;
        return acc;
    }
    
    @Override
    public Double getResult(AverageAccumulator acc) {
        return acc.sum / acc.count;
    }
    
    @Override
    public AverageAccumulator merge(AverageAccumulator a, AverageAccumulator b) {
        a.sum += b.sum;
        a.count += b.count;
        return a;
    }
}

// ProcessWindowFunction：全量处理，功能强
public class TopNProcess extends ProcessWindowFunction<Event, List<Event>, String, TimeWindow> {
    @Override
    public void process(String key, Context ctx, Iterable<Event> events, Collector<List<Event>> out) {
        List<Event> sorted = StreamSupport.stream(events.spliterator(), false)
            .sorted(Comparator.comparing(Event::getValue).reversed())
            .limit(10)
            .collect(Collectors.toList());
        out.collect(sorted);
    }
}
```

### 1.4.4 连接与合并

```java
// Union：同类型流的合并（多条流合成一条）
DataStream<Event> merged = stream1.union(stream2, stream3);

// Connect：不同类型流的连接（保留各自类型，后续可以 CoMap）
ConnectedStreams<Alert, Rule> connected = alerts.connect(rules);
DataStream<Action> actions = connected.map(
    new CoMapFunction<Alert, Rule, Action>() {
        public Action map1(Alert alert) { /* 处理 Alert */ }
        public Action map2(Rule rule) { /* 处理 Rule */ }
    }
);

// Join：基于窗口的关联（类似 SQL JOIN）
stream1.join(stream2)
    .where(Event1::getKey)
    .equalTo(Event2::getKey)
    .window(TumblingEventTimeWindows.of(Time.seconds(10)))
    .apply((e1, e2) -> new JoinedEvent(e1, e2));

// Interval Join：基于时间区间的关联（流 A 的元素关联流 B 在特定时间范围内的元素）
stream1.keyBy(Event1::getKey)
    .intervalJoin(stream2.keyBy(Event2::getKey))
    .between(Time.minutes(-5), Time.minutes(5))  // ±5 分钟内的关联
    .process(new ProcessJoinFunction<>() {...});
```

### 1.4.5 侧输出流（Side Output）

处理延迟数据、异常数据的优雅方式：

```java
// 定义侧输出标签
OutputTag<Event> lateDataTag = new OutputTag<Event>("late-data"){};

SingleOutputStreamOperator<Result> mainResult = stream
    .keyBy(Event::getType)
    .window(TumblingEventTimeWindows.of(Time.seconds(10)))
    .allowedLateness(Time.seconds(5))      // 允许 5 秒延迟
    .sideOutputLateData(lateDataTag)       // 超时的延迟数据发到侧输出
    .aggregate(new MyAggregate());

// 获取主输出
DataStream<Result> results = mainResult;

// 获取侧输出（延迟数据，可单独处理）
DataStream<Event> lateData = mainResult.getSideOutput(lateDataTag);
lateData.addSink(new LateDataHandler());
```

### 1.4.6 ProcessFunction：终极武器

当内置算子不够用时，ProcessFunction 提供对 Flink 运行时最底层的访问：

```java
public class MyProcess extends KeyedProcessFunction<String, Event, Result> {
    
    private ValueState<Long> state;
    
    @Override
    public void open(Configuration parameters) {
        state = getRuntimeContext().getState(
            new ValueStateDescriptor<>("myState", Types.LONG)
        );
    }
    
    @Override
    public void processElement(Event event, Context ctx, Collector<Result> out) 
            throws Exception {
        // 访问当前 key 的状态
        Long current = state.value();
        state.update(current == null ? 1 : current + 1);
        
        // 注册定时器（事件时间）
        ctx.timerService().registerEventTimeTimer(event.getTimestamp() + 60000);
        
        // 输出结果
        out.collect(new Result(event.getId(), state.value()));
    }
    
    @Override
    public void onTimer(long timestamp, OnTimerContext ctx, Collector<Result> out) 
            throws Exception {
        // 定时器触发时的回调
        // 可以做超时处理、状态清理等
    }
}
```

ProcessFunction 的能力：
- ✅ 访问和修改 KeyedState
- ✅ 注册事件时间/处理时间定时器
- ✅ 输出到侧输出流
- ✅ 获取当前 Processing Time / Watermark

---

## 附录：环境搭建速通

### A.1 Maven 依赖

```xml
<properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    <flink.version>1.18.0</flink.version>
</properties>

<dependencies>
    <!-- Flink 核心 -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-streaming-java</artifactId>
        <version>${flink.version}</version>
        <scope>provided</scope>  <!-- 集群运行时使用集群提供的版本 -->
    </dependency>
    
    <!-- Flink 客户端（本地运行时需要用） -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-clients</artifactId>
        <version>${flink.version}</version>
        <scope>provided</scope>
    </dependency>
    
    <!-- Kafka Connector（可选） -->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-connector-kafka</artifactId>
        <version>3.0.1-1.18</version>
    </dependency>
    
    <!-- 日志 -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>2.0.9</version>
    </dependency>
</dependencies>
```

### A.2 本地运行

```bash
# 1. 克隆/创建项目，添加依赖

# 2. 编译
mvn clean package

# 3. 本地运行（直接在 IDE 里运行 main 方法即可）
# 或者命令行：
mvn exec:java -Dexec.mainClass="com.example.flink.WordCount"
```

### A.3 提交到 Standalone 集群

```bash
# 1. 下载 Flink
wget https://archive.apache.org/dist/flink/flink-1.18.0/flink-1.18.0-bin-scala_2.12.tgz
tar -xzf flink-1.18.0-bin-scala_2.12.tgz
cd flink-1.18.0

# 2. 启动集群
./bin/start-cluster.sh

# 3. 提交作业
./bin/flink run -c com.example.flink.WordCount \
    /path/to/your-project-1.0-SNAPSHOT.jar

# 4. 查看 Web UI
open http://localhost:8081

# 5. 停止集群
./bin/stop-cluster.sh
```

### A.4 快速验证

```bash
# 用 netcat 模拟流数据源
nc -lk 9999

# 输入一些文字，Flink 程序会实时统计词频
hello world
hello flink
flink is awesome
```

---

## 本节回顾

| 知识点 | 掌握标准 |
|:---|:---|
| Flink 定位与优势 | 能向同事解释清楚"为什么用 Flink 而不是 Spark" |
| 三种时间语义 | 能说明 Event Time / Processing Time / Ingestion Time 的区别和适用场景 |
| Watermark | 能解释 Watermark 如何解决乱序问题，知道 allowedLateness 和 sideOutputLateData |
| 状态 | 知道 ValueState/ListState/MapState 的适用场景，能在 ProcessFunction 中使用 |
| Checkpoint | 能说明 Checkpoint 的触发流程和恢复机制 |
| 程序骨架 | 能独立写出完整的 `main` 方法结构（环境 → Source → Transform → Sink → execute） |
| 懒执行 | 能解释为什么 `env.execute()` 是必须的 |
| 基础算子 | 能熟练使用 map/filter/flatMap/keyBy/window/sum/aggregate |
| 窗口类型 | 能区分 Tumbling/Sliding/Session/Count 窗口，知道怎么选 |
| ProcessFunction | 知道它是"终极武器"，能在什么场景下使用 |

---

> **下一步**：Module 02 将深入 Flink 的部署与运维（Standalone/YARN/K8s）、监控指标调优、以及实际生产环境的最佳实践。
