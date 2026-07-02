---
Title: "已安装 Agent Skills 参考手册"
Date: 2026-07-02
Tags: ["#ai-workflow", "#agent", "#skills", "#reference", "#verified"]
Status: Verified
Source: "~/.agents/skills/ SKILL.md 与 ~/.agents/.skill-lock.json"
Related: ["knowledge/ai-workflow/claude-code-customization-guide.md", "rules/ai-workflow/ai-agent-skills-combo-design-enhance-frontend-stitch.md"]
---

# 已安装 Agent Skills 参考手册

> 本手册审计 `~/.agents/skills/` 目录下所有通过 `npx skills` 安装的 Skill，按功能领域分组，说明每个 Skill 的用途、使用场景、调用方式和注意事项。
>
> 更新时间：2026-07-02

---

## 安装来源总览

| Skill | 来源仓库 | 安装路径 |
|-------|----------|----------|
| `ai-research-explore` | 未在 `.skill-lock.json` 中记录 | `~/.agents/skills/ai-research-explore/` |
| `boost-prompt` | `github/awesome-copilot` | `~/.agents/skills/boost-prompt/` |
| `design-md` | `google-labs-code/stitch-skills` | `~/.agents/skills/design-md/` |
| `enhance-prompt` | `google-labs-code/stitch-skills` | `~/.agents/skills/enhance-prompt/` |
| `find-skills` | `vercel-labs/skills` | `~/.agents/skills/find-skills/` |
| `firecrawl-deep-research` | `firecrawl/firecrawl-workflows` | `~/.agents/skills/firecrawl-deep-research/` |
| `firecrawl-market-research` | `firecrawl/firecrawl-workflows` | `~/.agents/skills/firecrawl-market-research/` |
| `frontend-design` | `anthropics/skills` | `~/.agents/skills/frontend-design/` |
| `humanizer-zh` | `op7418/Humanizer-zh` | `~/.agents/skills/humanizer-zh/` |
| `read-arxiv-paper` | `karpathy/nanochat` | `~/.agents/skills/read-arxiv-paper/` |
| `stitch-loop` | `google-labs-code/stitch-skills` | `~/.agents/skills/stitch-loop/` |

> **注意**：`.skill-lock.json` 中记录了 `tech-research`，但文件系统中不存在；文件系统中存在 `ai-research-explore`，但 `.skill-lock.json` 未记录。原因待查。

---

## 快速索引

| 领域 | Skills |
|------|--------|
| 设计 / UI | `design-md`, `enhance-prompt`, `frontend-design`, `stitch-loop` |
| 研究 / 分析 | `ai-research-explore`, `firecrawl-deep-research`, `firecrawl-market-research`, `read-arxiv-paper` |
| 效率 / 工作流 | `boost-prompt`, `find-skills`, `humanizer-zh` |

---

## 一、设计 / UI

### 1.1 `design-md`

| 属性 | 说明 |
|------|------|
| **来源** | `google-labs-code/stitch-skills` |
| **一句话定义** | 分析 Stitch 项目，提取语义化设计系统并生成 `DESIGN.md`。 |
| **适用场景** | - 已有 Stitch 页面，需要沉淀为可复用的设计系统<br>- 为多页面 Stitch 项目建立统一的视觉语言基准<br>- 准备让 `enhance-prompt` / `stitch-loop` 引用设计上下文 |
| **调用方式** | `/design-md` 后提供 Stitch 项目或页面信息 |
| **关键要求** | - 需要 Stitch MCP Server<br>- 目标项目至少有一个已设计页面<br>- 输出文件名为 `DESIGN.md` |
| **注意事项** | 需要能访问 `htmlCode.downloadUrl` 和 `screenshot.downloadUrl`；输出以自然语言描述为主，避免技术术语堆砌。 |

### 1.2 `enhance-prompt`

| 属性 | 说明 |
|------|------|
| **来源** | `google-labs-code/stitch-skills` |
| **一句话定义** | 把粗糙的 UI 想法转化为 polished、Stitch 优化的生成 prompt。 |
| **适用场景** | - 只有一句话需求，如"做一个登录页"<br>- Stitch 生成结果不满意，需要优化 prompt<br>- 想保持多页面设计一致性 |
| **调用方式** | `/enhance-prompt "需求描述"` |
| **关键能力** | - 自动检查并注入 `DESIGN.md` 设计系统<br>- 添加 UI/UX 关键词、平台、页面结构、颜色格式<br>- 输出可直接用于 Stitch 生成 |
| **注意事项** | 如果项目没有 `DESIGN.md`，会在输出末尾提示创建；适合一次一个页面或一次一个组件修改。 |

### 1.3 `frontend-design`

| 属性 | 说明 |
|------|------|
| **来源** | `anthropics/skills` |
| **一句话定义** | 提供有辨识度、非模板化的视觉设计方向指导。 |
| **适用场景** | - 新项目启动，确定视觉调性<br>- 现有 UI 看起来太"AI 模板化"，需要差异化<br>- 需要为特定主题/受众做审美决策 |
| **调用方式** | `/frontend-design` 后描述产品、受众、情绪关键词 |
| **关键流程** | 1. 确定主题、受众、页面目标<br>2. 头脑风暴设计计划（颜色、字体、布局、signature）<br>3. 自检是否落入三种常见默认风格<br>4. 输出设计方向和实施建议 |
| **注意事项** | 提供的是方向而非最终代码；强调"只在一个地方大胆"，避免过度装饰；建议配合截图自我批评。 |

### 1.4 `stitch-loop`

| 属性 | 说明 |
|------|------|
| **来源** | `google-labs-code/stitch-skills` |
| **一句话定义** | 用"接力棒"模式自动迭代构建 Stitch 网站。 |
| **适用场景** | - 需要连续生成多个 Stitch 页面<br>- 希望建立可自动化/可人工审核的网站构建流水线<br>- 已有一个 Stitch 项目 + `DESIGN.md` + `SITE.md` |
| **调用方式** | `/stitch-loop`（读取 `.stitch/next-prompt.md` 作为当前任务） |
| **关键机制** | - Baton 文件：`.stitch/next-prompt.md`<br>- 上下文文件：`.stitch/DESIGN.md`、`.stitch/SITE.md`、`.stitch/metadata.json`<br>- 输出 staging：`.stitch/designs/{page}.html` 和 `.png`<br>- 生产目录：`site/public/{page}.html` |
| **注意事项** | - 必须在完成前更新 `.stitch/next-prompt.md`，否则循环中断<br>- 不要重复创建已存在的页面<br>- 可选 Chrome DevTools MCP 做视觉验证<br>- 与 `design-md` 和 `enhance-prompt` 搭配使用效果最佳 |

#### 设计领域组合用法

```text
Step 1: /frontend-design
   └─ 确定视觉方向和调性

Step 2: 用 Stitch 做出第一个页面

Step 3: /design-md
   └─ 从第一个页面生成 .stitch/DESIGN.md

Step 4: /enhance-prompt
   └─ 基于 DESIGN.md 优化新页面 prompt

Step 5: /stitch-loop
   └─ 持续迭代生成剩余页面
```

---

## 二、研究 / 分析

### 2.1 `ai-research-explore`

| 属性 | 说明 |
|------|------|
| **来源** | 未在 `.skill-lock.json` 中记录（文件系统存在） |
| **一句话定义** | 在有明确研究锚点的基础上，进行有科学依据的深度学习候选方案探索。 |
| **适用场景** | - 已确定 task family、dataset、benchmark、SOTA<br>- 想要在受控条件下生成并排序候选 idea<br>- 需要可审计、可比较的实验候选 |
| **调用方式** | `/ai-research-explore`，并提供 `current_research`、`task_family`、`dataset`、`benchmark`、`evaluation_source`、`sota_reference`、`compute_budget` |
| **关键流程** | 1. 确认 current_research 锚点<br>2. 冻结任务/数据/评估/预算<br>3. 理解代码库<br>4. 生成、排序、门控候选 idea<br>5. 输出到 `analysis_outputs/`、`sources/`、`explore_outputs/` |
| **注意事项** | - 不适用于开放方向探索、纯代码探索、纯运行探索、README-first 复现<br>- 不承诺 novelty 或 global benchmark 完整性<br>- 输出应标注为 candidate-only，不能当作可信复现 |

### 2.2 `firecrawl-deep-research`

| 属性 | 说明 |
|------|------|
| **来源** | `firecrawl/firecrawl-workflows` |
| **一句话定义** | 生成有引用、多角度的深度研究报告。 |
| **适用场景** | - 需要科学、技术、政策或市场分析类的正式书面报告<br>- 用户明确要求"给我一份报告"<br>- 话题复杂，无法通过短搜索回答 |
| **调用方式** | `/firecrawl-deep-research`，提供研究主题 |
| **关键输入** | `FIRECRAWL_API_KEY`（必需） |
| **深度分级** | Quick（3-5 查询，5-10 来源）/ Thorough（5-10 查询，15-25 来源）/ Exhaustive（10+ 查询，25+ 来源） |
| **输出结构** | Executive Summary、Key Findings、Detailed Analysis、Contrarian Views And Risks、Open Questions、Sources、Rerun Inputs |
| **注意事项** | - 不用于产品推荐、Top-N 列表、快速查询<br>- 默认会问"希望研究运行多久？" |

### 2.3 `firecrawl-market-research`

| 属性 | 说明 |
|------|------|
| **来源** | `firecrawl/firecrawl-workflows` |
| **一句话定义** | 提取市场、财务、行业和公司的结构化数据。 |
| **适用场景** | - 市场研究、行业趋势分析<br>- 上市公司数据、财务对比<br>- 财报研究、结构化市场报告 |
| **调用方式** | `/firecrawl-market-research`，提供市场/公司/指标焦点 |
| **关键输入** | `FIRECRAWL_API_KEY`（必需） |
| **输出结构** | Market Overview、Company Profiles、Comparison Tables、Trends And Outlook、Sources、Rerun Inputs |
| **注意事项** | - 交叉验证关键数字<br>- 标注数据冲突<br>- 每个指标注明周期和单位<br>- 不提供投资建议 |

### 2.4 `read-arxiv-paper`

| 属性 | 说明 |
|------|------|
| **来源** | `karpathy/nanochat` |
| **一句话定义** | 读取 arXiv 论文源码并生成摘要。 |
| **适用场景** | - 用户给出 arXiv URL，要求总结论文<br>- 想从论文中提取与当前项目（如 nanochat）相关的启示 |
| **调用方式** | `/read-arxiv-paper https://arxiv.org/abs/XXXX.XXXXX` |
| **关键流程** | 1. 将 URL 规范化为 `https://arxiv.org/src/XXXX.XXXXX`<br>2. 下载 `.tar.gz` 到 `~/.cache/nanochat/knowledge/`<br>3. 解压并定位 `main.tex` 入口<br>4. 递归读取论文源码<br>5. 生成 `./knowledge/summary_{tag}.md` |
| **注意事项** | - 下载的是 TeX 源码，不是 PDF<br>- 摘要会结合 nanochat 项目上下文分析应用价值<br>- 需要本地缓存目录可写 |

#### 研究领域组合用法

```text
轻量调研 → /find-skills 或常规搜索
深度报告 → /firecrawl-deep-research
市场/财务 → /firecrawl-market-research
论文精读 → /read-arxiv-paper
AI/ML 候选方案探索 → /ai-research-explore（需先准备好 task/dataset/benchmark/SOTA）
```

---

## 三、效率 / 工作流

### 3.1 `boost-prompt`

| 属性 | 说明 |
|------|------|
| **来源** | `github/awesome-copilot` |
| **一句话定义** | 交互式优化用户 prompt，把模糊需求变成结构化、可执行的任务描述。 |
| **适用场景** | - 有一个模糊想法，需要细化成清晰 prompt<br>- 想让 AI 执行任务前先把需求说透<br>- 准备把 prompt 交给另一个 agent 执行 |
| **调用方式** | `/boost-prompt "你的原始需求"` |
| **关键流程** | 1. 询问范围、交付物、约束<br>2. 必要时做项目探索<br>3. 输出优化后的 Markdown prompt<br>4. 复制到剪贴板（需 Joyride 扩展） |
| **注意事项** | - 不写代码，只优化 prompt<br>- 需要 Joyride 扩展才能自动复制到剪贴板<br>- 在纯 CLI 环境可手动复制聊天中的输出 |

### 3.2 `find-skills`

| 属性 | 说明 |
|------|------|
| **来源** | `vercel-labs/skills` |
| **一句话定义** | 帮用户发现、评估和安装社区 Skill。 |
| **适用场景** | - 用户问"怎么做 X"，可能已有 Skill<br>- 想找某个领域的现成 Skill<br>- 想扩展 agent 能力 |
| **调用方式** | `/find-skills` 或直接描述需求 |
| **关键命令** | `npx skills find [query]`、`npx skills add <package>`、`npx skills check`、`npx skills update` |
| **注意事项** | - 优先看 skills.sh leaderboard 和热门源（vercel-labs、anthropics、microsoft）<br>- 推荐前检查安装量和 GitHub stars<br>- 安装用 `-g -y` 全局跳过确认 |

### 3.3 `humanizer-zh`

| 属性 | 说明 |
|------|------|
| **来源** | `op7418/Humanizer-zh` |
| **一句话定义** | 去除中文文本中的 AI 生成痕迹，使其更自然。 |
| **适用场景** | - 编辑或审阅 AI 生成的中文文本<br>- 需要发布、对外分享的文档<br>- 任何读起来"太 AI"的中文内容 |
| **调用方式** | `/humanizer-zh` 后粘贴需要处理的文本 |
| **修复模式** | 夸大象征、宣传语言、-ing 肤浅分析、模糊归因、破折号过度使用、三段式、AI 词汇、否定式排比、连接词过多等 |
| **输出** | 重写后的文本 + 修改说明 + 1-10 分质量评分（5 个维度） |
| **注意事项** | - 不仅是"去 AI 味"，还要注入真实个性和观点<br>- 会改变原文风格，发布前需人工确认<br>- 允许 Read/Write/Edit/AskUserQuestion |

---

## 四、如何选择合适的 Skill

### 决策树

```text
你要做什么？
│
├─ 设计/生成 UI ─────────────────────→ design-md / enhance-prompt / frontend-design / stitch-loop
│   ├─ 已有页面，要提取设计系统 → design-md
│   ├─ 一句话需求，要优化 prompt → enhance-prompt
│   ├─ 要确定视觉方向 → frontend-design
│   └─ 要连续生成多页面 Stitch 站点 → stitch-loop
│
├─ 研究/分析 ────────────────────────→ ai-research-explore / firecrawl-deep-research / firecrawl-market-research / read-arxiv-paper
│   ├─ 深度学习候选方案探索 → ai-research-explore
│   ├─ 深度研究报告 → firecrawl-deep-research
│   ├─ 市场/财务数据 → firecrawl-market-research
│   └─ 读 arXiv 论文 → read-arxiv-paper
│
└─ 效率/工作流 ──────────────────────→ boost-prompt / find-skills / humanizer-zh
    ├─ 优化 prompt → boost-prompt
    ├─ 找/装 Skill → find-skills
    └─ 去除中文 AI 味 → humanizer-zh
```

### 关键原则

1. **Skill 不是万能的**：每个 Skill 都有明确的触发条件和限制，强行调用会浪费 token 和时间。
2. **先小后大**：简单查询用普通对话；复杂、结构化、可重复的任务再考虑 Skill。
3. **组合优于单用**：设计类 Skill 经常需要 `frontend-design` → `design-md` → `enhance-prompt` → `stitch-loop` 的流水线。
4. **注意依赖**：`design-md`、`stitch-loop` 需要 Stitch MCP Server；`firecrawl-*` 需要 `FIRECRAWL_API_KEY`；`boost-prompt` 理想情况下需要 Joyride。

---

## 五、待验证项

| 项 | 说明 | 建议操作 |
|----|------|----------|
| `ai-research-explore` 未记录 | 文件系统存在，但 `.skill-lock.json` 中没有 | 检查是否为手动复制或安装失败 |
| `tech-research` 丢失 | `.skill-lock.json` 记录，但文件系统不存在 | 重新安装或从 lock 文件中移除 |
| Skill 版本 | 本手册基于当前 `~/.agents/skills/` 内容 | 运行 `npx skills check` 和 `npx skills update` 后需重新审计 |

---

## 六、常用管理命令

```bash
# 搜索 Skill
npx skills find [query] [--owner <owner>]

# 安装 Skill（全局）
npx skills add <owner/repo@skill> -g -y

# 检查更新
npx skills check

# 更新所有 Skill
npx skills update

# 初始化新 Skill
npx skills init my-skill
```

---

## 七、相关文档

- [Claude Code 定制三件套](claude-code-customization-guide.md) — CLAUDE.md、Skills、Hooks 的完整说明。
- [AI Agent Skills 组合实践](../rules/ai-workflow/ai-agent-skills-combo-design-enhance-frontend-stitch.md) — `design-md`、`enhance-prompt`、`frontend-design`、`stitch-loop` 的组合用法。
