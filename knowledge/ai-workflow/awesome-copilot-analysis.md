---
Title: "GitHub Awesome Copilot 仓库分析与工程借鉴"
Date: 2026-07-11
Tags: ["#ai-workflow", "#github-copilot", "#awesome-list", "#skill-design", "#engineering-practices", "#analysis"]
Status: "Verified"
Source: "https://github.com/github/awesome-copilot"
Related: ["knowledge/ai-workflow/claude-code-customization-guide.md", "knowledge/ai-workflow/installed-skills-reference.md", "../awesome-ai-coding/README.md"]
---

# GitHub Awesome Copilot 仓库分析与工程借鉴

> 本文是对 [`github/awesome-copilot`](https://github.com/github/awesome-copilot) 的系统性拆解，覆盖其内容组织、工程规范、自动化工具链和生态设计，并提炼对 [`awesome-ai-coding`](https://github.com/goodlydotonline/awesome-ai-coding) 以及个人 AI 工作流建设的可复用经验。

---

## 1. 项目概述

### 1.1 定位与目标受众

`awesome-copilot` 是 GitHub 官方维护的**社区驱动型 GitHub Copilot 定制资源集市**。它不是单一工具，而是一个把零散 prompt、agent、instruction、skill、hook、workflow、plugin 等 AI 可消费资源进行**标准化编目**的仓库。

目标受众包括：

| 角色 | 使用方式 |
| :--- | :--- |
| 普通开发者 | 安装 plugin / skill / agent，提升 Copilot 在特定技术栈下的表现 |
| 团队 Lead | 把 `.instructions.md` 或 `.agent.md` 作为团队规范下发 |
| AI 工具 / 平台开发者 | 参考其目录结构、元数据规范、marketplace 生成机制 |
| 社区贡献者 | 提交新的 agent、skill、workflow 等，遵循既定流程 |

### 1.2 核心使命

仓库首页用一句话概括：

> *A community-created collection of custom agents, instructions, skills, hooks, workflows, and plugins to supercharge your GitHub Copilot experience.*

其背后隐含了三个工程目标：

1. **可发现（Discoverable）**：通过分类目录、生成表格、网站、marketplace.json 让用户快速找到资源。
2. **可安装（Installable）**：资源不只是文档，而是能被 Copilot CLI、VS Code、GitHub Actions 直接消费的资产。
3. **可治理（Governable）**：通过 frontmatter、校验脚本、CI 工作流、外部插件审核流程保证质量。

### 1.3 规模感知

从仓库结构看，其规模已经超出普通 awesome-list：

- `agents/`：220+ 个 `.agent.md` 文件
- `skills/`：400+ 个 skill 文件夹
- `instructions/`：200+ 个 `.instructions.md` 文件
- `plugins/`：60+ 个 plugin 目录
- `hooks/`：8 个 hook 目录（起步阶段）
- `workflows/`：10 个 Agentic Workflow
- `extensions/`：10+ Canvas Extension
- `cookbook/`：示例与 SDK 使用指南

> 参考来源：[README.md](https://github.com/github/awesome-copilot/blob/main/README.md)、[docs/README.agents.md](https://github.com/github/awesome-copilot/blob/main/docs/README.agents.md)、[docs/README.skills.md](https://github.com/github/awesome-copilot/blob/main/docs/README.skills.md) 等生成索引。

---

## 2. 内容组织：七类 AI 可消费资产

仓库把 AI 定制资源拆成七类，每一类都有独立目录、命名约定、frontmatter 和贡献指南。这种分类是工程上最值得学习的第一点。

### 2.1 资产类型总览

| 类型 | 目录 | 核心文件 | 作用 | 典型使用方式 |
| :--- | :--- | :--- | :--- | :--- |
| **Agents** | `agents/` | `<name>.agent.md` | 把 Copilot Chat 变成特定领域的专家助手 | 安装到 VS Code / CCA |
| **Instructions** | `instructions/` | `<name>.instructions.md` | 按文件模式自动生效的编码规范 | 放入 `.github/instructions/` |
| **Skills** | `skills/` | `<name>/SKILL.md` | 自带 bundled assets 的复杂任务模板 | `gh skills install` 或复制 |
| **Plugins** | `plugins/` | `<name>/.github/plugin/plugin.json` | 把多个 agent/skill/command 打包成主题工具包 | `copilot plugin install` |
| **Hooks** | `hooks/` | `<name>/README.md` + `hooks.json` | 在 Copilot coding agent 会话中触发自动化脚本 | 放入 `.github/hooks/` |
| **Agentic Workflows** | `workflows/` | `<name>.md` | 用自然语言编写的 GitHub Actions 自动化 | `gh aw compile` 后提交 |
| **Cookbook** | `cookbook/` | `README.md` + 示例 | 面向 Copilot API / SDK 的 copy-paste 示例 | 学习、参考 |

> 来源：[README.md#whats-in-this-repo](https://github.com/github/awesome-copilot/blob/main/README.md#whats-in-this-repo)

### 2.2 分类背后的设计思想

1. **粒度分层**：Instruction 是最轻量的规范；Skill 增加了 bundled assets；Plugin 再把多个资源打包；Workflow 提升到仓库级自动化。
2. **触发方式不同**：Instruction 按文件模式自动触发；Agent 由用户显式召唤；Hook 由会话事件触发；Workflow 由 schedule / issue / PR 触发。
3. **可组合性**：Plugin 只声明 `agents`、`commands`、`skills` 数组，源文件仍在顶层目录，CI 负责“物化”到 marketplace。这避免了重复存储，也便于交叉引用。

### 2.3 “一个 Skill 一个文件夹”模式

Skill 采用 **one-folder-per-skill** 组织：

```text
skills/<skill-name>/
├── SKILL.md              # 主定义，必须带 frontmatter
├── README.md（可选）     # 更详细的使用说明
├── assets/（可选）       # 模板、图片、示例
├── references/（可选）   # 补充文档、快速参考
├── scripts/（可选）      # 可执行脚本
└── LICENSE.txt（可选）   # 许可证
```

示例：`skills/acquire-codebase-knowledge/` 包含 `scripts/scan.py`、`references/stack-detection.md`、`assets/templates` 等。

> 来源：[docs/README.skills.md](https://github.com/github/awesome-copilot/blob/main/docs/README.skills.md)、[AGENTS.md#agent-skills](https://github.com/github/awesome-copilot/blob/main/AGENTS.md#agent-skills)

**工程价值**：

- 自包含：迁移、安装、删除只需复制/删除一个文件夹。
- 可发现：文件夹名即 skill ID，与 `SKILL.md` 中 `name` 字段一致。
- 可扩展：assets / references / scripts 让复杂任务有地方放配套资源，而不是全塞进 prompt。

### 2.4 Skill 的交付物契约与典型案例

高质量的 `SKILL.md` 不仅是 prompt，更是一份**可执行契约**。其正文通常包含以下章节：

| 章节 | 作用 |
| :--- | :--- |
| `## When to Use` / `## When NOT to Use` | 明确触发条件与反例，防止误触发 |
| `## Prerequisites` | 环境、CLI、权限、IDE 扩展要求 |
| `## Step-by-Step Workflow` | 编号步骤，强调顺序执行 |
| `## Core Capabilities` | 能力清单 |
| `## Guidelines` / `## Agent Behavior Rules` | `DO` / `DO NOT` 行为约束 |
| `## Anti-Patterns` | 禁止做法与正确做法对照 |
| `## Output Contract` | 交付物验收标准 |
| `## Bundled Assets` / `## References` | 关联资源索引 |

几个值得学习的代表性 skill：

| Skill | 亮点 |
| :--- | :--- |
| [`acquire-codebase-knowledge`](https://github.com/github/awesome-copilot/tree/main/skills/acquire-codebase-knowledge) | 把“理解代码库”转化为 7 份可验证文档；自带 `scripts/scan.py`；要求每条主张都有源文件证据 |
| [`brag-sheet`](https://github.com/github/awesome-copilot/tree/main/skills/brag-sheet) | 挖掘 Copilot CLI session 日志、git 提交、`gh pr list` 补全遗忘工作；强制 `action → result → evidence` 三段式 |
| [`copilot-pr-autopilot`](https://github.com/github/awesome-copilot/tree/main/skills/copilot-pr-autopilot) | 10 步循环驱动 Copilot Code Review；每 10 轮 circuit breaker 防止机器人失控 |
| [`code-tour`](https://github.com/github/awesome-copilot/tree/main/skills/code-tour) | 为 20 种 persona 生成 CodeTour `.tour` 文件；自带 `validate_tour.py` 校验路径与行号 |
| [`draw-io-diagram-generator`](https://github.com/github/awesome-copilot/tree/main/skills/draw-io-diagram-generator) | 生成可直接在 VS Code draw.io 扩展中打开的 `.drawio` 文件；包含 XML 规范与校验脚本 |

> 来源：`skills/` 目录扫描与 `SKILL.md` 抽样分析。

---

## 3. 元数据与命名规范

仓库对每种资产都定义了严格的 frontmatter 和命名规则，这是保证 700+ 贡献者能协同、自动化能跑起来的基础。

### 3.1 通用规则

| 规则 | 说明 |
| :--- | :--- |
| 文件名 | 全小写，单词用连字符 `-` 连接（kebab-case） |
| 描述字段 | 用单引号包裹，非空，长度受限 |
| frontmatter | 必须存在，且字段完整 |
| 路径引用 | plugin.json 中 `agents`、`skills` 等必须用相对路径指向真实存在的文件 |

> 来源：[AGENTS.md#code-style-guidelines](https://github.com/github/awesome-copilot/blob/main/AGENTS.md#code-style-guidelines)、[.github/copilot-instructions.md](https://github.com/github/awesome-copilot/blob/main/.github/copilot-instructions.md)

### 3.2 各类资产的 frontmatter 要求

#### Agent 文件（`.agent.md`）

```markdown
---
description: 'Brief description of the agent and its purpose'
model: 'gpt-5'
tools: ['codebase', 'terminalCommand']
name: 'My Agent Name'
---
```

- `description`：必填。
- `model`：强烈建议指定，避免默认模型不匹配。
- `tools`：建议列出该 agent 依赖的工具。
- `name`：人类可读名称。

示例：[`agents/address-comments.agent.md`](https://github.com/github/awesome-copilot/blob/main/agents/address-comments.agent.md) 指定了 28 个 tools，包括 `changes`、`codebase`、`editFiles`、`github` 等。

#### Instruction 文件（`.instructions.md`）

```markdown
---
description: 'Guidelines for building C# applications'
applyTo: '**.cs, **.csproj'
---
```

- `applyTo`：指定生效的文件模式，多个模式用逗号分隔，如 `'**.js, **.ts'`。

示例：[`instructions/csharp.instructions.md`](https://github.com/github/awesome-copilot/blob/main/instructions/csharp.instructions.md)。

#### Skill（`SKILL.md`）

```markdown
---
name: 'my-skill-name'
description: 'Clear and non-empty description, 10-1024 chars'
---
```

- `name`：必须与文件夹名一致，kebab-case，最多 64 字符。
- `description`：单引号，10-1024 字符。
- bundled assets 必须在正文里引用，且单个文件不超过 5MB。

#### Plugin（`plugin.json`）

```json
{
  "name": "my-plugin-id",
  "description": "Plugin description",
  "version": "1.0.0",
  "keywords": [],
  "author": { "name": "Awesome Copilot Community" },
  "repository": "https://github.com/github/awesome-copilot",
  "license": "MIT",
  "agents": ["./agents/my-agent.md"],
  "commands": ["./commands/my-command.md"],
  "skills": ["./skills/my-skill/"]
}
```

关键工程点：

- Plugin 目录在 `main` 分支**只保留** `.github/plugin/plugin.json` 和 `README.md`。
- `agents`、`commands`、`skills` 通过相对路径指向顶层源文件；真正的 marketplace 发布时由 CI “物化”。
- 不允许在 plugin 目录内出现 symlink 或实物化文件。

> 来源：[CONTRIBUTING.md#adding-plugins](https://github.com/github/awesome-copilot/blob/main/CONTRIBUTING.md#adding-plugins)、[AGENTS.md#plugin-folders](https://github.com/github/awesome-copilot/blob/main/AGENTS.md#plugin-folders)

#### Hook（`README.md` + `hooks.json`）

```markdown
---
name: "My Hook Name"
description: "Brief description of what this hook does"
tags: ["logging", "automation"]
---
```

`hooks.json` 配置触发事件：`sessionStart`、`sessionEnd`、`userPromptSubmitted`、`preToolUse`、`postToolUse`、`errorOccurred`。

示例：[`hooks/session-logger/`](https://github.com/github/awesome-copilot/tree/main/hooks/session-logger)。

#### Agentic Workflow（`.md`）

```markdown
---
name: "Daily Issues Report"
description: "Generates a daily summary of open issues..."
on:
  schedule: daily on weekdays
permissions:
  contents: read
  issues: read
safe-outputs:
  create-issue:
    title-prefix: "[daily-report] "
    labels: [report]
---
```

- 只能提交 `.md` 源文件，`.yml` / `.lock.yml` 会被 CI 拦截。
- 遵循最小权限原则，用 `safe-outputs` 代替直接写权限。

> 来源：[CONTRIBUTING.md#adding-agentic-workflows](https://github.com/github/awesome-copilot/blob/main/CONTRIBUTING.md#adding-agentic-workflows)

---

## 4. 工程实践：贡献、校验与发布

### 4.1 本地开发流程

官方推荐的贡献流程非常清晰：

```bash
# 1. 安装依赖
npm ci

# 2. 创建新资源
npm run skill:create -- --name <skill-name> --description "..."
npm run plugin:create -- --name <plugin-name>

# 3. 编辑文件后校验
npm run skill:validate
npm run plugin:validate

# 4. 重新生成 README / marketplace
npm run build      # 或 npm start

# 5. 修复行尾符
bash eng/fix-line-endings.sh
```

> 来源：[AGENTS.md#setup-commands](https://github.com/github/awesome-copilot/blob/main/AGENTS.md#setup-commands)、[CONTRIBUTING.md#submitting-your-contribution](https://github.com/github/awesome-copilot/blob/main/CONTRIBUTING.md#submitting-your-contribution)

### 4.2 关键脚本职责

| 脚本 | 路径 | 职责 |
| :--- | :--- | :--- |
| `update-readme.mjs` | `eng/` | 扫描 agents / instructions / skills / hooks / workflows / plugins，生成根 README.md 和 docs/README.*.md 中的索引表格 |
| `generate-marketplace.mjs` | `eng/` | 读取所有 `plugins/<name>/.github/plugin/plugin.json`，生成 `.github/plugin/marketplace.json` |
| `generate-website-data.mjs` | `eng/` | 为官方网站生成 JSON 数据 |
| `validate-plugins.mjs` | `eng/` | 校验 plugin 元数据、路径一致性、外部插件规则 |
| `validate-skills.mjs` | `eng/` | 校验 skill 结构、frontmatter、资产引用 |
| `create-skill.mjs` / `create-plugin.mjs` | `eng/` | 脚手架生成新资源 |
| `materialize-plugins.mjs` / `clean-materialized-plugins.mjs` | `eng/` | 在发布到 `marketplace` 分支时物化/清理 plugin 内容 |
| `fix-line-endings.sh` | `eng/` | 把 CRLF 转 LF |

> 来源：[eng/README.md](https://github.com/github/awesome-copilot/blob/main/eng/README.md)

### 4.3 分支策略

仓库使用 **`main` 作为源分支**，`staged` / `marketplace` 是发布/物化分支。

- 所有 PR 必须指向 `main`。
- 不要从 `staged` 切分支，否则可能带上已物化的 plugin 文件，导致冲突。
- 外部插件审批后，机器人自动向 `main` 提交更新 `plugins/external.json` 的 PR。

> 来源：[CONTRIBUTING.md#submitting-your-contribution](https://github.com/github/awesome-copilot/blob/main/CONTRIBUTING.md#submitting-your-contribution)

### 4.4 CI/CD 质量门槛

`.github/workflows/` 里约有 30 个工作流，形成多层防护：

| 工作流 | 触发条件 | 作用 |
| :--- | :--- | :--- |
| `validate-readme.yml` | PR 改动资源目录 | 运行 `npm run plugin:validate` + `npm start`，若 README 生成结果与提交不一致则失败 |
| `skill-check.yml` | PR 改动 skills/agents | 用 `@microsoft/vally-cli lint` 校验 changed skill / agent |
| `check-plugin-structure.yml` | PR 改动 plugins | 检查 plugin 目录内是否存在实物化文件或 symlink |
| `validate-canvas-extensions.yml` | PR 改动 extensions | 校验 canvas extension 结构 |
| `validate-agentic-workflows-pr.yml` | PR 改动 workflows | 校验 `.md` workflow，阻止 `.yml`/`.lock.yml` 入仓 |
| `contributor-check.yml` | PR / issue 打开/编辑 | 用 AGT（Agent Governance Toolkit）做贡献者声誉与凭据审计 |
| `external-plugin-intake.yml` | issue 提交外部插件 | 自动化 intake 校验、打标签、质量门 |
| `external-plugin-pr-quality-gates.yml` | 外部插件 PR | 对 changed external plugin 跑安装冒烟测试和 vally lint |
| `publish.yml` | 发布 | 把资源物化并发布到 marketplace 分支/网站 |
| `build-website.yml` / `deploy-website.yml` | 网站相关改动 | 构建并部署官方站点 |

> 来源：[`.github/workflows/`](https://github.com/github/awesome-copilot/tree/main/.github/workflows)

### 4.5 外部插件审核流程

`awesome-copilot` 本身是一个 plugin marketplace，对外部插件的审核堪称小型应用商店：

1. 提交者通过 issue 表单提交，包含 `repo`、`ref`、`sha`、`version`、`license` 等字段。
2. 自动化 intake 校验必填字段和 GitHub 仓库格式。
3. 质量门：对提交路径跑 `vally lint` + Copilot CLI 安装冒烟测试。
4. 通过后排到 `ready-for-review`，维护者用 `/approve` 或 `/reject <reason>` 决定。
5. 批准后机器人自动更新 `plugins/external.json` 并生成 marketplace 输出。
6. 每六个月自动重新审核（`re-review-due`），可 `/re-review-keep`、`/re-review-needs-changes`、`/re-review-remove`。

> 来源：[CONTRIBUTING.md#adding-external-plugins](https://github.com/github/awesome-copilot/blob/main/CONTRIBUTING.md#adding-external-plugins)

---

## 5. 跨平台与生态设计

### 5.1 多入口消费

同一套资源支持多种入口：

| 入口 | 使用方式 | 对应资源 |
| :--- | :--- | :--- |
| VS Code Copilot Chat | 点击 Install in VS Code 按钮 | agents / instructions |
| GitHub Copilot CLI | `copilot plugin install <name>@awesome-copilot` | plugins |
| GitHub CLI | `gh skills install github/awesome-copilot <skill-name>` | skills |
| GitHub Actions | `gh aw compile` + 提交 `.md` | workflows |
| Canvas / Extensions | 安装 extension 目录 | extensions |
| 官方网站 | 搜索、过滤、浏览 | 全部 |
| `llms.txt` | 机器可读索引 | agents / instructions / skills |

> 来源：[README.md](https://github.com/github/awesome-copilot/blob/main/README.md)

### 5.2 规范优先于实现

仓库大量依赖**外部规范**，而不是自己发明格式：

- Agent Skills 遵循 [agentskills.io/specification](https://agentskills.io/specification)。
- Plugin 使用 **Claude Code spec**（`agents`、`commands`、`skills` 字段）。
- Agentic Workflows 遵循 [GitHub Agentic Workflows 规范](https://github.github.com/gh-aw)。
- Hooks 遵循 [GitHub Copilot hooks 规范](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/use-hooks)。
- Instructions 遵循 VS Code Copilot Customization 的文件模式约定。

**工程启示**：不要为每种平台单独设计格式，而是在通用层定义最小的 schema，再在不同平台上做适配层。

### 5.3 Marketplace 物化机制

Plugin 是一个有趣的工程案例：

- `main` 分支只保存“声明式”元数据（`plugin.json` + `README.md`）。
- 发布时 CI 运行 `materialize-plugins.mjs`，把顶层 `agents/`、`commands/`、`skills/` 中声明的文件**复制/ symlink** 到 plugin 目录，再推到 `marketplace` 分支。
- 消费端（Copilot CLI）从 `marketplace` 分支读取完整 plugin。

这样做的好处：

- 避免源文件在多个 plugin 中重复。
- 一个 agent/skill 可以被多个 plugin 引用。
- `main` 保持轻量，便于 PR 审查。

> 来源：[`.github/workflows/check-plugin-structure.yml`](https://github.com/github/awesome-copilot/blob/main/.github/workflows/check-plugin-structure.yml)、[CONTRIBUTING.md#plugin-guidelines](https://github.com/github/awesome-copilot/blob/main/CONTRIBUTING.md#plugin-guidelines)

### 5.4 MCP 生态集成

很多 agent 不是“空 prompt”，而是声明依赖 MCP server。例如 [`agents/apify-integration-expert.agent.md`](https://github.com/github/awesome-copilot/blob/main/agents/apify-integration-expert.agent.md) 明确依赖 `apify` MCP，并在 README 表格里展示 “Install MCP” 按钮。

这代表一种趋势：**agent 的能力边界由 prompt + tools + MCP 共同定义**。仓库通过表格把 MCP 依赖显式化，用户安装前就知道需要配置什么。

### 5.5 Skill 的分类机制与平台适配

`awesome-copilot` 的 Skill **没有强主题标签体系**。构建脚本为 Skill 生成的 filters 只有 `hasAssets: ["Yes", "No"]`，网站和 README 主要按标题字母顺序排列。主题识别更多依赖：

- **命名前缀**：`azure-*`、`aws-*`、`csharp-*`、`mcp-*`、`github-*`
- **描述关键词**：用户通过全文搜索发现

但这并不意味着 Skill 不写平台要求，相反，**平台适配被显式化**：

| 维度 | 说明 |
| :--- | :--- |
| 操作系统 | `batch-files` 声明 Windows + cmd.exe；`brag-sheet` 声明 Cross-platform |
| CLI / 运行时 | `acquire-codebase-knowledge` 要求 Python 3.8+ + git；`copilot-pr-autopilot` 要求 PowerShell + `gh` CLI |
| IDE 扩展 | `draw-io-diagram-generator` 依赖 `hediet.vscode-drawio`；`code-tour` 依赖 CodeTour 扩展 |
| 第三方服务 | 大量 Azure / AWS / Microsoft 365 / GitHub 原生 Skill |

**工程启示**：不要假设用户环境一致，而是在 `Prerequisites` 或 frontmatter 中把依赖说清楚，让安装失败时容易定位。

---

## 6. 对 `awesome-ai-coding` 的借鉴与建议

### 6.1 推荐采纳的做法

#### 6.1.1 严格分层：Skill / Agent / Instruction / Hook / Plugin / Workflow

`awesome-ai-coding` 目前以 skills 为主，未来可逐步引入：

- `agents/`：针对 Claude Code / Codex / Cursor 的 chat mode 定义。
- `instructions/`：项目级或语言级编码规范。
- `hooks/`：在 AI 工具执行前后插入检查（如 secrets scan、lint、test）。
- `plugins/`：把相关 skills 打包成场景化工具包。
- `workflows/`：Agentic Workflow 或自定义自动化。

#### 6.1.2 “一个 Skill 一个文件夹”并附带平台适配文件

`awesome-ai-coding` 已经采用 `skills/<name>/SKILL.md`，可以继续深化：

```text
skills/<name>/
├── SKILL.md          # 通用/跨平台定义
├── README.md         # 使用说明
├── claude-code.md    # Claude Code 适配
├── codex.md          # GitHub Copilot / Codex 适配
└── assets/           # 模板、脚本、参考数据
```

这与 `awesome-copilot` 的 `SKILL.md + bundled assets` 模式一致，只是增加了平台适配层。

#### 6.1.3 frontmatter 作为“机器可读契约”

`awesome-coding` 已经使用：

```yaml
---
Title: "..."
Date: YYYY-MM-DD
Tags: ["#tag"]
Status: "..."
Source: "..."
Related: ["..."]
---
```

可以继续借鉴 `awesome-copilot` 在 SKILL.md / agent.md 内部增加 `name`、`description`、`model`、`tools`、`applyTo` 等技术 frontmatter，用于自动化索引和校验。

#### 6.1.4 用脚本生成索引，而非手工维护

`awesome-copilot` 的 `npm run build` 会重新生成所有 README 表格。`awesome-ai-coding` 当前还靠手工更新根 README 和各目录 README，建议引入：

- `scripts/update-index.mjs`：扫描 skills / collections，更新 README 表格。
- `scripts/validate-skills.mjs`：检查文件夹名与 `SKILL.md` 中 `name` 是否一致、frontmatter 是否完整。
- GitHub Actions：PR 时自动校验索引是否过期。

#### 6.1.5 拒绝低价值贡献

`CONTRIBUTING.md` 明确拒绝：

> *Duplicate Existing Model Strengths Without Meaningful Uplift*

即不要提交让 Copilot 做它已经擅长的事的 instruction（如泛泛的 TypeScript、HTML）。

对个人仓库同样适用：skill 应该解决**特定领域、特定工具链、特定团队规范**的问题，而不是泛泛的“帮我写代码”。

#### 6.1.6 安全与责任 AI 底线

`awesome-copilot` 拒绝任何绕过安全策略、生成有害内容、利用漏洞的提交，并运行 contributor-check 与凭据审计。

`awesome-ai-coding` 应在 `CONTRIBUTING.md` 中明确：

- 不收录用于规避安全、生成恶意代码、自动化攻击的 skill。
- hook 涉及 secrets scan、权限检查等安全检查优先。

### 6.2 需要避免的反模式

| 反模式 | 说明 | 规避方法 |
| :--- | :--- | :--- |
| **把 awesome-list 当成笔记堆** | 只收集链接或 prompt 片段，缺乏结构化 frontmatter | 每个条目必须有 frontmatter + 明确目录 |
| **手工维护大量表格** | 条目多了之后更新索引成为负担 | 用脚本生成 README 表格 |
| **Skill 里塞超大二进制** | 仓库膨胀、下载慢 | 单个 asset 限制在 5MB 以内 |
| **把不同平台格式混在同一文件** | 导致无法跨工具消费 | 通用放 `SKILL.md`，平台适配放 `claude-code.md` / `codex.md` |
| **缺少校验门槛** | PR 质量不可控 | 引入本地 `npm run validate` + GitHub Actions |
| **让 AI 直接提交 prompt 而不审查** | 易产生低质量、冲突或有害内容 | 要求人类 review，AI 用 `🤖🤖🤖` 标记 PR 标题以快速通道 |

### 6.3 可立即落地的行动建议

针对 `awesome-ai-coding` 的现状，建议按优先级实施：

#### 立即（本周）

1. **统一 SKILL.md frontmatter**：要求每个 skill 文件夹的 `SKILL.md` 包含 `name`（与文件夹同名）和 `description`（10-1024 字符）。
2. **更新 README 索引脚本**：写一个 `scripts/update-readme.mjs`，自动扫描 `skills/`、`collections/` 并生成表格，减少手工维护。
3. **明确平台适配文件约定**：`claude-code.md`、`codex.md` 等文件名的语义写入 `CLAUDE.md` 或 `docs/style-guide.md`。

#### 短期（本月）

4. **引入 `agents/` 和 `instructions/` 目录**：即使初期只有一个示例，也能让结构完整。
5. **引入 GitHub Actions**：
   - PR 时校验 frontmatter 完整性。
   - PR 时运行 `npm run build`，检查 README 是否过期。
6. **写一份 `CONTRIBUTING.md`**：明确接受/拒绝的 skill 类型、命名规则、PR 流程。

#### 中期（未来 1-3 个月）

7. **设计 Plugin 机制**：允许把多个 skill 声明式组合成 `plugins/<theme>/plugin.json`，CI 生成 marketplace.json。
8. **建立外部 skill 审核流程**：如果有人想提交 skill，通过 issue 模板 + `/approve` 命令管理。
9. **发布机器可读索引**：如 `llms.txt` 或 JSON catalog，让 AI 工具能自动发现仓库资源。

---

## 7. 个人学习要点

1. **AI 定制资源的本质不是 prompt 工程，而是“产品化”**：把 prompt、tool、MCP、workflow 当作可安装、可版本化、可治理的软件包。
2. **目录结构即 API**：`agents/`、`skills/`、`plugins/` 这些目录名不只是分类，它们决定了消费端如何扫描和加载。
3. **自动化是规模的必要条件**：当条目超过 50 个，手工维护索引和规范检查不现实，必须用脚本和 CI 承担重复劳动。
4. **规范先于内容**：先定义 frontmatter、命名、目录、PR 流程，再开放贡献，否则后期治理成本极高。
5. **生态借力**：不要自己发明格式，优先复用 Claude Code spec、Agent Skills spec、GitHub Agentic Workflows 等已有标准。

---

## 8. 相关链接

- 分析对象仓库：[github/awesome-copilot](https://github.com/github/awesome-copilot)
- 官方站点：[awesome-copilot.github.com](https://awesome-copilot.github.com)
- 机器可读索引：[awesome-copilot.github.com/llms.txt](https://awesome-copilot.github.com/llms.txt)
- 核心文档：
  - [README.md](https://github.com/github/awesome-copilot/blob/main/README.md)
  - [CONTRIBUTING.md](https://github.com/github/awesome-copilot/blob/main/CONTRIBUTING.md)
  - [AGENTS.md](https://github.com/github/awesome-copilot/blob/main/AGENTS.md)
  - [.github/copilot-instructions.md](https://github.com/github/awesome-copilot/blob/main/.github/copilot-instructions.md)
- 本分析所属项目：[goodlydotonline/my-engineering-os](https://github.com/goodlydotonline/my-engineering-os)
- 新建的 skill 仓库：[goodlydotonline/awesome-ai-coding](https://github.com/goodlydotonline/awesome-ai-coding)

---

*文档生成时间：2026-07-11*
