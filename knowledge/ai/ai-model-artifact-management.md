# AI 模型制品管理：Fine-tuning 权重是否应该进 Git？

> **一句话结论**：Fine-tuning 得到的 `.pt`、`.pth`、`.ckpt` 等模型权重文件属于**制品（artifact）**，不是代码，**不应该直接提交到 Git**。工业界的标准做法是：用 Git 管理代码和配置，用 DVC / Git LFS / 对象存储 / 模型注册中心来管理模型文件。

---

## 1. 为什么模型权重不能进 Git？

Git 是为文本代码设计的版本控制工具，二进制大文件会让仓库迅速劣化：

| 问题 | 说明 |
|---|---|
| 仓库体积膨胀 | 一个 YOLO `.pt` 通常 5~50MB；多版本、多实验、多模型叠加后，仓库可能从 MB 级涨到 GB 级。 |
| 无法有效 diff | Git 对二进制文件只能做全量快照，无法像代码一样查看“改了什么”。 |
| 历史记录爆炸 | 每次更新模型都会产生一份完整副本，即使只改了一点点参数。 |
| clone/pull 变慢 | 新成员、CI、部署环境拉代码时都要下载所有历史模型。 |
| 职责错位 | Git 管“代码状态”，模型文件应该由专门的制品管理工具负责。 |

> 即使当前模型只有 5MB，也建议从一开始就建立规范，否则后续 22 类模型、多个 baseline、多个实验版本会让仓库不可维护。

---

## 2. 工业界的常见做法

### 2.1 DVC（Data Version Control）—— ML 工程首选

DVC 可以理解为“给数据和模型做的 Git”。

```text
代码、配置、DVC 元文件    →  提交到 Git
模型文件 (.pt/.pth)       →  放到 S3 / OSS / GCS / MinIO
DVC 只记录文件的 hash 和远程地址
```

基本用法：

```bash
# 安装 DVC 和对应远程存储插件
pip install dvc dvc-oss   # 或 dvc-s3, dvc-gcs

# 初始化
dvc init

# 把模型注册为 DVC 跟踪文件
dvc add ai-service/app/weights/v0.1/best.pt

# 配置远程存储
dvc remote add -d myremote oss://firecopilot-models/weights/
dvc push
```

提交到 Git 的只有小文件：

```text
ai-service/app/weights/v0.1/best.pt.dvc   ← 记录模型 hash 和远程路径
ai-service/app/weights/v0.1/.gitignore    ← 忽略真实的 .pt
```

协作者拉代码后：

```bash
dvc pull
# best.pt 自动从远程存储下载到本地
```

**优点**：

- 模型版本和代码版本一一对应（`git checkout` 后 `dvc checkout`）
- 支持多实验、多版本管理
- 远程存储便宜（对象存储按量计费）
- 和 Git 工作流完全融合

---

### 2.2 Git LFS（Large File Storage）—— 简单但不够 ML 友好

Git LFS 让 Git 把大文件存到 LFS 服务器，工作区里看起来还是普通文件。

```bash
# 跟踪所有 .pt 文件
git lfs track "*.pt"

# 正常 add/commit
git add ai-service/app/weights/v0.1/best.pt
git commit -m "add fine-tuned model"
```

**优点**：

- 对使用者透明，像普通文件一样操作
- 配置简单，小团队上手快

**缺点**：

- LFS 存储通常按容量收费
- 历史模型版本也会占用空间
- 不适合频繁产生大量实验模型的场景

**适合场景**：模型数量少、团队规模小、不想引入 DVC。

---

### 2.3 模型注册中心 / MLOps 平台

更成熟的团队会用专门的模型管理和实验追踪平台：

| 工具 | 典型用法 |
|---|---|
| **MLflow Tracking + Model Registry** | 记录实验参数 → 注册模型版本 → CI/CD 按版本拉取 |
| **Weights & Biases Artifacts** | 实验管理 + 模型版本 + 可视化 |
| **Hugging Face Hub（私有 repo）** | 适合 YOLO / Transformer，下载方便，`from_pretrained` 即可 |
| **内部 MinIO / S3 + 自研 model-gateway** | 大厂常见，按项目自己搭建制品服务 |

这些工具通常提供类似接口：

```text
model: firecopilot/yolo11n-v0.1-subset-6cls
version: v1.0.0
```

部署时由服务或 CI/CD 从注册中心下载指定版本。

---

### 2.4 CI/CD 构建时拉取（部署侧常用）

部署 Docker 镜像时，不把模型打进镜像层，而是：

1. 镜像里只放代码和依赖
2. 构建时或启动时从 S3/OSS 下载 `best.pt` 到指定目录
3. 或者把模型目录挂载为外部 volume

```dockerfile
# 示例：启动脚本中拉取模型
RUN python scripts/download_model.py \
    --model firecopilot/yolo11n-v0.1-subset-6cls \
    --output app/weights/v0.1/best.pt
```

**优点**：

- 镜像体积小
- 模型可以独立更新，不用重新打镜像
- 同一镜像可切换不同模型版本

---

## 3. 应该提交到 Git 的有哪些？

| 类型 | 是否进 Git | 示例 |
|---|---|---|
| 训练脚本和推理代码 | ✅ 必须 | `train_yolo_baseline.py`, `yolo_detector.py` |
| 模型配置文件 | ✅ 必须 | `configs/yolo11n-v0.1.yaml` |
| 类别映射表 | ✅ 必须 | `class_mapping.json` |
| 训练/评估报告 | ✅ 推荐 | `reports/yolo11n_test.json` |
| 数据集划分配置 | ✅ 必须 | `split_manifest.json` |
| 模型权重 `.pt` / `.pth` | ❌ 不要 | `best.pt`, `last.pt` |
| 训练中间检查点 | ❌ 不要 | `epoch10.pt`, `epoch20.pt` |
| 原始图片/视频 | ❌ 不要 | `datasets/firecopilot-v0.1/images/` |

---

## 4. FireCopilot 当前实践的判定

当前 FireCopilot 项目已经做对了关键一步：

```gitignore
# Model weights (YOLO / PyTorch / ONNX)
*.pt
*.pth
*.onnx
*.engine
*.mlmodel
```

这意味着：

- `best.pt` 不会被提交到 Git
- `class_mapping.json` 等小文件可以被提交
- 模型文件应通过本地拷贝、DVC、或远程存储来分发

### 4.1 当前最小可行流程

```text
1. 训练完成 → best.pt 在 ai-service/training/runs/.../weights/best.pt
2. 拷贝到部署目录 → ai-service/weights/deployed/v0.1/best.pt
3. 提交 class_mapping.json + 代码/配置变更到 Git
4. best.pt 保留在本地或推送到远程存储
```

### 4.2 下一步演进建议

当模型版本开始增多（YOLO11n / YOLO26n / 全量 22 类 / 不同 epochs），建议引入 DVC：

```text
ai-service/weights/
├── pretrained/
│   ├── yolo11n.pt              # 本地文件，.gitignore 忽略
│   └── yolov8s-world.pt
└── deployed/
    └── v0.1/
        ├── best.pt              # 本地文件，.gitignore 忽略
        ├── best.pt.dvc          # 提交到 Git，DVC 元数据
        └── class_mapping.json   # 提交到 Git
```

切换版本就像切换代码分支：

```bash
git checkout v0.1.0
dvc checkout
# best.pt 自动变成 v0.1.0 对应的模型
```

---

## 5. 决策速查表

| 场景 | 推荐方案 |
|---|---|
| 个人实验，模型 < 10MB，团队 1~2 人 | 可以本地保留 `.pt`，Git 忽略即可 |
| 小团队，模型版本不多 | Git LFS |
| 正式 ML 项目，需要实验复现和版本对应 | **DVC + S3/OSS** |
| 企业级，多模型、多版本、多环境 | 模型注册中心（MLflow / W&B / 自研） |
| 容器化部署，镜像需要瘦身 | CI/CD 构建时从远程拉取模型 |

---

## 6. 关联文档

- [[ai-training-terminology]] — YOLO 训练术语、class_id 原理、评估指标
- [[engineering-git-workflow]] — Git 分支与提交规范（如果存在）

---

*创建于 2026-07-09*
