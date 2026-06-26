---
Title: "AI Agent Skills 组合实践：design-md / enhance-prompt / frontend-design / stitch-loop"
Date: 2026-06-24
Tags: ["#ai-workflow", "#rule", "#agent", "#frontend", "#design", "#stitch", "#verified"]
Status: Verified
Source: "skill-registry-analysis"
Related: []
---

# AI Agent Skills 组合实践

## 涉及的 Skills

| Skill | 核心能力 | 适用阶段 |
|-------|----------|----------|
| `design-md` | 分析 Stitch 项目，提炼语义化设计系统并生成 `DESIGN.md` | 设计系统沉淀 / 项目收尾 |
| `enhance-prompt` | 把粗糙的 UI 想法转化为 polished、Stitch 优化的 prompt | 生成前 / 需求输入阶段 |
| `frontend-design` | 提供有辨识度、有意图的视觉设计指导 | 设计方向探索 / 美学决策 |
| `stitch-loop` | 当前 Skill 注册表中未列出，可能指 Stitch 生成-迭代-反馈的循环工作流 | 迭代优化阶段 |

> 注：截至当前会话环境，`stitch-loop` 未出现在可用 Skill 列表中。下文将其作为“Stitch 迭代工作流”这一概念进行讨论；如后续该 Skill 可用，可直接替换为 Skill 调用。

## 它们之间有关联吗？

**有，且是强关联。** 四个能力围绕同一个目标展开：

> **让 AI 生成符合设计意图、可落地、可复用的前端界面。**

它们分别负责工作流的不同环节：

```text
设计方向 ──→ prompt 增强 ──→ 生成/迭代 ──→ 设计系统沉淀
   │              │               │                │
frontend-design enhance-prompt  stitch-loop     design-md
```

- `frontend-design` 解决“要长成什么样”的问题。
- `enhance-prompt` 解决“怎么告诉 Stitch 才能生成这样”的问题。
- `stitch-loop`（或手工迭代）解决“生成结果不满意，如何多次优化”的问题。
- `design-md` 解决“把结果沉淀为可复用设计规范”的问题。

## 单独使用的最佳实践

### design-md

**什么时候用：**
- 项目已有 UI，需要提炼设计系统。
- 团队需要一份 `DESIGN.md` 作为后续生成的基准。
- 收尾阶段做知识沉淀。

**怎么用：**
- 指向 Stitch 项目根目录或关键组件文件。
- 让 Skill 扫描颜色、字体、间距、组件模式、语义命名。
- 输出后人工审查，补充业务-specific 的约束。

**注意事项：**
- 它分析的是“已有实现”，不是“未来构想”。
- 输出质量取决于项目本身是否一致、规范。

### enhance-prompt

**什么时候用：**
- 只有一句话需求，如“做一个设置页面”。
- prompt 生成结果不稳定或太模板化。
- 需要把设计系统上下文注入到 prompt 中。

**怎么用：**
- 输入原始想法 + 目标平台 + 用户场景。
- 如果有 `DESIGN.md`，先让 Skill 读取并融入 prompt。
- 明确输出格式要求（如组件列表、变体、响应式规则）。

**注意事项：**
- 不要输入过度设计的需求，避免 prompt 膨胀。
- 增强后的 prompt 建议保存到 `rules/prompts/` 供复用。

### frontend-design

**什么时候用：**
- 新项目启动，需要确定视觉方向。
- 现有 UI 看起来太“模板化”，需要差异化。
- 做设计评审前，先让 AI 给出方向性建议。

**怎么用：**
- 描述产品定位、目标用户、情绪关键词。
- 让 Skill 输出多套方向（如极简、工业、温暖、未来感）。
- 选定方向后，把结论写进 `DESIGN.md` 或 `knowledge/frontend/`。

**注意事项：**
- 它提供的是方向，不是最终设计稿。
- 输出需要与品牌、现有组件库对齐。

### stitch-loop（概念）

**什么时候用：**
- Stitch 首次生成结果不满意。
- 需要多轮微调（颜色、布局、间距、文案）。
- 需要在生成过程中持续引入约束。

**怎么用：**
- 第 1 轮：用 `enhance-prompt` 生成初始 prompt。
- 第 2 轮：检查输出，把问题结构化（如“按钮太大会员化”“间距不统一”）。
- 第 3 轮：把问题反馈给 AI，要求基于 `DESIGN.md` 修正。
- 重复直到满意或触发收益递减。

## 可以组合使用吗？

**可以，而且推荐组合使用。** 它们天然构成一个“设计 → 生成 → 迭代 → 沉淀”的闭环。

## 推荐组合工作流

### 场景 A：从 0 到 1 做一个新页面

```text
Step 1: frontend-design
   └─ 输出视觉方向、调性、关键设计决策

Step 2: enhance-prompt
   └─ 把方向 + 页面需求 转化为 Stitch-ready prompt

Step 3: Stitch 生成初稿

Step 4: stitch-loop（或手工迭代）
   └─ 多轮修正，直到满足要求

Step 5: design-md
   └─ 把最终页面提炼为 DESIGN.md，沉淀设计系统
```

### 场景 B：给已有项目新增组件

```text
Step 1: design-md
   └─ 读取现有项目，输出 DESIGN.md

Step 2: enhance-prompt
   └─ 基于 DESIGN.md 写新组件 prompt

Step 3: Stitch 生成

Step 4: stitch-loop
   └─ 让新组件与现有设计系统对齐
```

### 场景 C：设计系统升级 / 风格重塑

```text
Step 1: design-md
   └─ 先梳理现有设计系统

Step 2: frontend-design
   └─ 基于现状提出新方向

Step 3: enhance-prompt
   └─ 把新方向转化为示例页面/组件 prompt

Step 4: stitch-loop
   └─ 小范围验证新风格

Step 5: design-md
   └─ 重新输出升级后的 DESIGN.md
```

## 组合使用时的原则

1. **不要跳过前端设计方向直接生成。**
   没有方向时，`enhance-prompt` 容易产出平庸、模板化的结果。

2. **始终让后续步骤能看到前面的结论。**
   把 `frontend-design` 的输出、`DESIGN.md`、上一轮生成结果都作为上下文传给下一步。

3. **小步快跑，不要一次改太多。**
   在 `stitch-loop` 中，每轮只聚焦 1-2 个明确问题，避免 prompt 失控。

4. **把 prompt 资产化。**
   优质的增强后 prompt 应该保存到 `rules/prompts/` 或 `knowledge/ai/`，而不是每次重新写。

5. **设计系统不是一次性的。**
   每次重大项目后都用 `design-md` 更新 `DESIGN.md`，保持其与代码一致。

## 与本工程操作系统的结合

- 把 `frontend-design` 的输出保存到 `knowledge/frontend/` 或 `projects/<name>/design-direction.md`。
- 把 `enhance-prompt` 的成品 prompt 保存到 `rules/prompts/stitch-ui-generation.md`。
- 把 `design-md` 输出的 `DESIGN.md` 放在项目根目录或 `knowledge/frontend/design-system.md`。
- 把 `stitch-loop` 的迭代记录保存到 `inbox/` 或 `meta/`，便于复盘哪些修正最有效。

## 快速启动模板

当你要启动一个 Stitch 相关页面时，可以按下面顺序调用：

```text
/frontend-design
  "产品：xxx，目标用户：yyy，希望调性：zzz，避免模板感"

/enhance-prompt
  "基于上面方向，生成 Stitch prompt，目标是：一个 xxx 页面"

/stitch-loop
  "用上面 prompt 生成，检查是否符合 DESIGN.md，修正：..."

/design-md
  "分析最终项目，输出 DESIGN.md"
```

如果 `stitch-loop` 当前不可用，把第三步替换为多次手动 `/capture` 迭代记录 + 重新调用 `enhance-prompt` 即可。
