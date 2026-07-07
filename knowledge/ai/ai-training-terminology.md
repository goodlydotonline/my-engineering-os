# AI 训练相关术语

## 一、按阶段分

| 名词 | 英文 | 含义 | 例子 |
|---|---|---|---|
| 预训练 | Pre-training | 在大规模通用数据上先学一遍，得到基础能力 | YOLO11n 在 COCO 数据集上预训练 |
| 微调 | Fine-tuning | 在预训练基础上，用目标数据继续训练 | 用消防数据在 `yolo11n.pt` 上微调 |
| 中间训练 | Intermediate training | 预训练和微调之间的额外训练，通常用领域大数据 | 先用消防领域大数据训，再用具体项目微调 |
| 增量训练 / 持续训练 | Continual training | 模型上线后持续加入新数据继续训练 | 每月把新采集的隐患图片加进来训练 |
| 后训练 | Post-training | 训练完成后的进一步处理，严格说不算训练阶段 | 量化、蒸馏、RLHF 对齐 |

> 没有“中训练”这个标准说法。如果有人说“中训练”，通常指的是 **Intermediate training**，或者口语化的“训练到一半”。

---

## 二、按训练方式 / 技术分

| 名词 | 英文 | 含义 |
|---|---|---|
| 监督学习 | Supervised learning | 有标注数据，告诉模型“这张图里有什么” |
| 自监督学习 | Self-supervised learning | 没有人工标注，模型自己从数据结构设计任务去学 |
| 无监督学习 | Unsupervised learning | 完全没有标签，让模型自己发现结构 |
| 迁移学习 | Transfer learning | 把 A 任务学到的能力迁移到 B 任务 |
| 域适应 | Domain adaptation | 模型从通用场景适配到特定场景 |
| 知识蒸馏 | Distillation | 用大模型教小模型，让小模型逼近大模型效果 |
| 量化感知训练 | Quantization-aware training (QAT) | 训练时就考虑量化，让模型适合在边缘设备跑 |
| 课程学习 | Curriculum learning | 先学简单样本，再学难样本 |
| 热启动 | Warm start | 用已有权重开始训练 |
| 冷启动 | Cold start | 从零随机权重开始训练 |

---

## 三、YOLO / 目标检测里常见的

| 名词 | 含义 |
|---|---|
| Backbone（骨干网络） | 负责提取图片特征的部分 |
| Head（检测头） | 负责输出 bbox 和类别的部分 |
| Neck | Backbone 和 Head 之间的特征融合层 |
| Epoch | 整个训练集过一遍 |
| Batch / Batch size | 一次处理多少张图 |
| Iteration / Step | 一次参数更新 |
| Checkpoint | 训练过程中保存的模型快照 |
| Best.pt / Last.pt | 验证效果最好 / 最后一轮的权重 |
| mAP | 目标检测常用评估指标 |
| Loss（损失） | 模型预测和真实标注的差距 |
| Learning rate（学习率） | 每次更新参数的步长 |
| 过拟合 Overfitting | 模型只记住了训练集，泛化能力差 |
| 欠拟合 Underfitting | 模型还没学够 |

---

## 四、YOLO 模型中的类别 ID（class_id）

在 YOLO 训练里，`class_id` 看起来只是一个整数，但它同时承担两层含义：对人/业务来说，它是类别的名字；对神经网络来说，它是检测头输出向量的**下标（索引）**。理解这一层，就能明白为什么训练时不能随便改 `nc`，也不能让 `class_id` 越界。

### 4.1 对人：class_id 是业务标识

比如 FireCopilot 的 `data.yaml` 里可能这样写：

```yaml
names:
  0: FIRE_HYDRANT_BLOCKED
  2: E_BIKE_VIOLATION
  10: STAIRCASE_OBSTRUCTION
```

对人来说，`0`、`2`、`10` 只是 `FIRE_HYDRANT_BLOCKED`、`E_BIKE_VIOLATION`、`STAIRCASE_OBSTRUCTION` 的缩写。

### 4.2 对模型：class_id 是输出向量的座位号

YOLO 的检测头（Head）会为每个候选框输出一条向量，大致长这样：

```text
[x, y, w, h, objectness, score_0, score_1, score_2, ..., score_nc-1]
            ↑ 这 4 个是 bbox   ↑ 这个是“有没有物体”   ↑ 这 nc 个是“每类概率”
```

- `nc` 就是 `data.yaml` 里的 `nc`。
- `score_3` 表示模型认为这个框属于第 3 类的概率。
- 标签里的 `class_id=3`，意思就是：**第 3 个位置上的分数应该高，其他位置应该低**。

所以 `class_id` 对模型来说，就像**电影院座位号**：座位号是几，就往第几个位置坐。

### 4.3 `nc` 和 `names` 的分工

- **`nc`**：决定电影院有多少个座位（输出向量有多长）。
- **`names`**：决定每个座位上坐的是谁（把索引 `0~nc-1` 翻译成人类可读的类别名）。

```yaml
nc: 22
names:
  0: FIRE_HYDRANT_BLOCKED   # 座位 0 坐的是 FIRE_HYDRANT_BLOCKED
  1: BLOCKED_FIRE_EXIT      # 座位 1 坐的是 BLOCKED_FIRE_EXIT
  # ... 一直到 21
```

### 4.4 为什么 class_id 必须满足 `0 <= class_id < nc`

还是因为它是数组下标。

如果 `nc=22`，那就只有 0~21 这 22 个合法座位。标签里一旦出现 `class_id=30`，就相当于给了一张“30 号座位”的票——电影院根本没有这个座位，程序会直接报索引越界错误。

在训练时，这个错误通常发生在计算分类损失（比如 CrossEntropyLoss）的地方：

```python
loss = F.cross_entropy(pred_scores, class_id)
# 如果 class_id >= nc，这里会抛 RuntimeError
```

### 4.5 ID 不连续也可以，但会浪费座位

不是说 ID 必须从 0 开始连续递增，而是说：**最大的 class_id 不能超过 `nc-1`**。

比如只有 6 个类，ID 分别是 `0, 2, 3, 4, 6, 10`，那 `nc` 至少要是 `11`（最大 ID 10 + 1）。向量里 0、2、3、4、6、10 这 6 个位置有训练信号，1、5、7、8、9 这 5 个位置是“空座位”，模型也会学，但永远学不到正确样本。

### 4.6 只训练部分类别时的两种方案

实际项目中经常只想训练一部分类别，比如 FireCopilot 的 22 类里只选 `0, 2, 3, 4, 6, 10`。这时有两种做法：

| 方案 | 做法 | `nc` | 优点 | 缺点 |
|---|---|---|---|---|
| **A：保留原始 ID** | 标签里仍然写 `0, 2, 3, 4, 6, 10` | 22 | 模型输出的 class_id 和文档/数据库完全一致，下游不用转换 | 模型头有 22 个输出，其中 16 个没有样本，浪费参数和计算 |
| **B：remap 成 0~K-1** | 把 `0,2,3,4,6,10` 映射为 `0,1,2,3,4,5` | 6 | 模型头最小，只学这 6 类 | 推理返回的是 0~5，必须维护“模型 ID → 业务 ID”映射表 |

### 4.7 部署时的 ID 映射问题

这个选择会直接影响 AI service 的代码。

- **方案 A**：模型输出 `class_id=10`，AI service 直接拿 `10` 去查数据库或 hazard-class-taxonomy.md，**不需要额外映射**。
- **方案 B**：模型输出 `class_id=5`，AI service 必须查表才知道它对应原始业务 ID 是 `10`。如果忘了做这层映射，就会把一个楼梯间 obstruction 错误地当成另一个类别。

另外，Ultralytics 的 `result.names` 只能告诉你“模型内部 class_id=5 叫什么名字”，它不会自动帮你做业务 ID 对齐。

### 4.8 怎么选？

- 类别数固定、追求模型更小、能接受维护映射表 → **方案 B**。
- 要和现有 taxonomy/数据库保持一致、类别可能继续扩展、不想在推理层加映射 → **方案 A**。

在当前 FireCopilot 项目里，因为下游 APP、数据库、法规引用都基于 `hazard-class-taxonomy.md` 里定义的原始 class_id，所以**方案 A 更安全**，代价就是模型头仍然是 22 类。

---

## 五、模型评估指标

| 缩写 | 全称 | 含义 | 公式 |
|---|---|---|---|
| TP | True Positive | 预测正确，且与真实框 IoU ≥ 阈值 | — |
| FP | False Positive | 预测错误，或 IoU 不足，或重复检测 | — |
| FN | False Negative | 真实存在但模型没检测到的目标 | — |
| IoU | Intersection over Union | 预测框与真实框的重叠比例 | `交集面积 / 并集面积` |
| P | Precision（精确率） | 预测为某类的框中，真正属于该类的比例 | `TP / (TP + FP)` |
| R | Recall（召回率） | 所有真实目标中，被模型找出的比例 | `TP / (TP + FN)` |
| PR | Precision-Recall 曲线 | 横轴 Recall、纵轴 Precision 的曲线 | 曲线下面积为 AP |
| AP | Average Precision | 单个类别的 PR 曲线下面积 | — |
| mAP | mean Average Precision | 所有类别 AP 的平均值 | 常用 `mAP@0.5` 和 `mAP@0.5:0.95` |
| F1 | F1 Score | Precision 和 Recall 的调和平均 | `2PR / (P + R)` |

### 训练结果里的曲线图

| 文件名 | 说明 |
|---|---|
| `P_curve.png` | 不同置信度阈值下的 Precision 变化 |
| `R_curve.png` | 不同置信度阈值下的 Recall 变化 |
| `PR_curve.png` | PR 曲线，围成的面积即 AP |
| `F1_curve.png` | 不同置信度阈值下的 F1 Score，峰值点代表较均衡的阈值 |
| `confusion_matrix.png` | 各类别之间的误分情况 |

### 怎么看

- **想要误报少** → 提高置信度阈值，关注 **Precision**。
- **想要漏检少** → 降低置信度阈值，关注 **Recall**。
- **想要两者兼顾** → 看 **F1 Score** 最大的点。
- **看模型整体好坏** → 看 **mAP**，尤其是 `mAP@0.5`。

---

## 六、为什么叫“预训练”？

最早机器学习都是**冷启动**，直接在自己的小数据上从零训练。后来研究者发现：

> 先在 ImageNet、COCO 这种大数据上训练好的模型，再用自己的数据微调，效果比从零开始好得多。

于是就有了“**预训练（Pre-training）**”这个说法，意思是“**提前训练好，供下游任务使用**”。

相对的，**微调（Fine-tuning）**就是“把预训练模型调整到具体任务上”。

---

## 七、当前项目的训练流程

消防 YOLO 训练准确说就是：

```text
Pre-trained yolo11n.pt
        ↓
Fine-tuning on firecopilot-v0.1
        ↓
    best.pt
```

---

*创建于 2026-07-07*
