---
Title: "Claude Code 定制三件套：CLAUDE.md · Skills · Hooks"
Date: 2026-06-24
Tags: ["#ai-workflow", "#agent", "#claude-code", "#verified"]
Status: Verified
Source: "https://code.claude.com/docs"
Related: ["rules/ai-workflow/ai-agent-skills-combo-design-enhance-frontend-stitch.md"]
---

# Claude Code 定制三件套：CLAUDE.md · Skills · Hooks

> 一份「怎么把 Claude Code 调成你自己工作流」的实操参考。
> 三种机制各回答四个问题：**是什么 / 怎么用 / 怎么创建 / 什么场景用**，最后用 `tidy-docs` 这个真实例子走一遍「从想法到 Skill」的完整流程。
>
> 准确性说明：本文技术细节以 Claude Code 官方文档（code.claude.com/docs）为准。Claude Code 迭代很快，**字段名、Hook 事件名以你本机 `claude --version` 对应的官方文档为准**；文中标注「需查证」的，是会随版本漂移、用前请复核的点。

---

## 0. 一分钟速览

| 维度 | **CLAUDE.md** | **Skills** | **Hooks** |
| :-- | :-- | :-- | :-- |
| 本质 | 每次会话**自动加载**的常驻说明书 | **按需加载**的流程/知识包 | 在生命周期事件上触发的**确定性 shell 命令** |
| 谁写 | 你（人） | 你（人） | 你（人） |
| 谁触发 | 自动，永远在场 | 你打 `/名字`，或模型判断相关时自动调 | 系统在事件发生时自动执行 |
| 加载时机 | 会话启动即全文进上下文 | 调用时才把正文塞进上下文 | 不进上下文，作为外部程序运行 |
| 强制力 | **软**：是上下文/建议，模型可能不遵守 | **软**：同上，靠 description 命中 | **硬**：代码层执行，与模型意图无关 |
| 成本 | 常驻 token，越长越稀释 | 不调用≈零成本 | 零 token，但有进程/延迟成本 |
| 典型产物 | `./CLAUDE.md`、`~/.claude/CLAUDE.md` | `.claude/skills/<名>/SKILL.md` | `settings.json` 里的 `hooks` 配置 |
| 一句话 | 「**永远**记住这些事实/规矩」 | 「**需要时**照这套流程做」 | 「**每次** X 发生，**必须**跑 Y」 |

**一句话选型**：常驻的简短事实 → CLAUDE.md；偶尔触发的多步流程或大段参考 → Skill；不容商量、必须发生的动作 → Hook。

---

## 1. 心智模型：三层 + 「治已病 vs 治未病」

可以把三者想成**三层防线**，从「软建议」到「硬执行」：

```
       软 ←─────────────────────────────────→ 硬
   CLAUDE.md            Skills              Hooks
  常驻的约定        按需的流程/知识        事件触发的强制动作
  「应该这样」       「这样做这件事」        「必须发生这件事」
```

另一个有用的切分是**「治未病 vs 治已病」**：

- **CLAUDE.md = 治未病（预防）**：把约定常驻在场，从源头减少「写出烂代码 / 乱放文件」的概率。它便宜、长期、广谱，但不保证执行。
- **Skill = 治已病（按需诊疗）**：当问题已经发生（文档乱了、要发版了、要审 PR 了），你显式叫它来，按既定流程处理一遍。
- **Hook = 强制隔离（硬约束）**：不靠模型「自觉」，在代码层卡死或自动补齐——比如「提交前必须跑 lint」「禁止 `rm -rf`」。

> 官方有一句很关键的话：CLAUDE.md 和记忆都是**上下文，不是被强制执行的配置**。**「要无论模型怎么想都必须阻止的动作，用 PreToolUse Hook，而不是写在 CLAUDE.md 里。」** 这就是软/硬分界。

---

## 2. CLAUDE.md

### 2.1 是什么

CLAUDE.md 是一个**纯 Markdown 文件**，给 Claude 提供跨会话的持久指令。每次会话启动时，Claude Code 会把它**全文**读入上下文（作为系统提示之后的一条用户消息）。所以它适合放「**每次会话都该记得的事实和规矩**」：构建命令、目录约定、命名规范、"永远做 X / 永远别做 Y"。

注意它和**自动记忆（Auto memory）**是两套互补系统：CLAUDE.md 是**你写**的指令；自动记忆是 **Claude 自己**根据你的纠正/偏好记的笔记（存在 `~/.claude/projects/<project>/memory/`，本仓库当前正在用的就是它）。本文只讲你手写的 CLAUDE.md。

### 2.2 怎么用（放在哪 + 怎么写）

**放在哪**——按加载顺序，从「广」到「窄」（窄的后加载、优先级更高）：

| 作用域 | 位置 | 用途 |
| :-- | :-- | :-- |
| 组织级（Managed） | macOS：`/Library/Application Support/ClaudeCode/CLAUDE.md`<br>Linux/WSL：`/etc/claude-code/CLAUDE.md`<br>Windows：`C:\Program Files\ClaudeCode\CLAUDE.md` | 全公司统一标准（IT 下发） |
| **用户级** | `~/.claude/CLAUDE.md` | 你个人所有项目通用的偏好 |
| **项目级** | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | 团队共享、随仓库进版本控制 |
| 本地级 | `./CLAUDE.local.md` | 个人的、项目内但不入库（记得加 `.gitignore`） |

加载规则：Claude Code 从当前工作目录**向上逐级**查找每一层的 `CLAUDE.md` 全部拼接进上下文（根目录在前、离你启动位置越近越靠后）；**子目录**里的 CLAUDE.md 不在启动时加载，而是当 Claude 读到那个子目录的文件时才按需加载。

**怎么写**（官方强调，写法直接影响遵守率）：

- **短**：目标每个文件 **200 行以内**。越长越稀释、遵守率越低。
- **结构化**：用标题和 bullet 分组，别堆大段落。
- **具体到可验证**：写「用 2 空格缩进」而不是「格式化好代码」；写「提交前跑 `npm test`」而不是「测试你的改动」。
- **不自相矛盾**：多个 CLAUDE.md 规则打架时，模型会随机挑一个。

**进阶**：
- `@路径` 导入：`CLAUDE.md` 里写 `@docs/git-instructions.md` 会把该文件展开进上下文（最多 4 层嵌套）。想提到路径又不导入，用反引号包起来。
- `.claude/rules/`：大项目可把规则拆成多文件，每个一个主题（`testing.md`、`security.md`）。带 `paths:` frontmatter 的规则**只在 Claude 处理匹配文件时才加载**，省上下文。

### 2.3 怎么创建

最简单：在项目根新建 `CLAUDE.md`，写几条规矩即可。或者：

```bash
# 在项目里跑（官方推荐）：Claude 扫描代码库，自动生成一份起始 CLAUDE.md
/init
```

`/init` 会分析代码库，填入它发现的构建命令、测试命令、项目约定；已存在则给改进建议而非覆盖。生成后你再补充它「猜不到」的规矩。

一个精简的项目级 `CLAUDE.md` 例子：

```markdown
# 项目约定

## 构建与测试
- 装依赖：`pnpm install`
- 跑测试：`pnpm test`（提交前必跑）
- 代理环境下测试 loopback 会断：`NO_PROXY=localhost,127.0.0.1 pnpm test`

## 代码风格
- 2 空格缩进；组件文件用 PascalCase
- API handler 放在 `src/api/handlers/`

## 别做
- 别在未问的情况下提交或推送
```

### 2.4 什么场景用

✅ 适合：
- 构建/测试/lint 命令这类**每次都要用**的事实
- 目录结构、命名约定、架构决策
- 「永远做 X / 永远别做 Y」的短规矩
- 你**上次会话已经纠正过、这次不想再说一遍**的东西

❌ 不适合：
- 多步**流程**（发版、审查 PR）→ 用 Skill
- 只在代码库某一小块才相关的规则 → 用 `.claude/rules/`（带 `paths`）
- **必须**强制发生的动作（提交前必跑 lint）→ 用 Hook
- 大段参考资料（API 规格、长清单）→ 用 Skill 的正文/附属文件，按需加载

---

## 3. Skills

### 3.1 是什么

Skill 是一个**带 `SKILL.md` 的目录**：YAML frontmatter 告诉 Claude「这技能干嘛、什么时候用」，下面的 Markdown 正文是「触发后照着做的指令」。

和 CLAUDE.md 的关键区别——**按需加载**：平时只有 `description` 在上下文里供模型判断是否相关；**只有真正被调用时**，正文才进上下文。所以哪怕正文很长、附带大段参考资料，不用时**几乎零成本**。

> 官方判断标准：当你**反复把同一套指令/清单/多步流程**粘进对话，或者 **CLAUDE.md 里某一段从"事实"长成了"流程"**，就该把它做成 Skill。

（注：旧的「自定义命令」`.claude/commands/*.md` 已并入 Skills，二者都生成 `/名字`，老文件继续可用。）

### 3.2 怎么用（放在哪 + 怎么触发 + frontmatter）

**放在哪**——决定谁能用：

| 作用域 | 路径 | 适用范围 |
| :-- | :-- | :-- |
| 个人 | `~/.claude/skills/<名>/SKILL.md` | 你的所有项目 |
| 项目 | `.claude/skills/<名>/SKILL.md` | 仅本项目（随仓库共享） |
| 插件 | `<plugin>/skills/<名>/SKILL.md` | 启用该插件处 |

同名时：企业 > 个人 > 项目；任意层级都能覆盖同名内置 Skill。

**怎么触发**（两种）：
1. **你手动**：打 `/<名>`。命令名来自**目录名**（不是 frontmatter 的 `name`）。
2. **模型自动**：当你的话和 `description` 匹配时，Claude 自行加载。

**两个控制谁能触发的字段**：
- `disable-model-invocation: true`：**只有你**能调（模型不会自动跑）。用于有副作用、要你掌控时机的动作——`/deploy`、`/commit`、`/send-slack-message`。
- `user-invocable: false`：**只有 Claude**能调，`/` 菜单里藏起来。用于「背景知识」型技能（如"遗留系统怎么回事"），它该知道但不是你要手动点的命令。

**frontmatter 字段速查**（全部可选，**只有 `description` 是推荐项**）：

| 字段 | 作用 |
| :-- | :-- |
| `name` | 列表里显示的名字。**默认取目录名**；一般不影响你 `/` 后打的命令名 |
| `description` | **最重要**：这技能干嘛 + 何时用。模型靠它判断是否自动调。把关键用例写在最前 |
| `when_to_use` | 补充触发场景/触发短语，追加在 description 后 |
| `argument-hint` | 自动补全时的参数提示，如 `[issue-number]` |
| `disable-model-invocation` | `true` = 仅手动 `/名` 触发 |
| `user-invocable` | `false` = 仅模型可调，菜单隐藏 |
| `allowed-tools` | 此技能激活时，免确认放行的工具（如 `Bash(git add *)`） |
| `disallowed-tools` | 此技能激活时，从可用池里移除的工具 |
| `model` / `effort` | 此技能激活时临时切换模型 / 推理强度 |
| `context: fork` | 在**隔离子代理**里跑（拿不到主对话历史），正文即子代理的 prompt |
| `agent` | 配合 `context: fork`，指定用哪种子代理（`Explore`/`Plan`/自定义） |
| `paths` | glob，限定只有处理匹配文件时才自动激活 |

> 字段集合会随版本增减，**以官方 frontmatter reference 为准（需查证）**。

**两个好用的进阶能力**：
- **动态上下文注入**：正文里写 `` !`命令` ``，Claude Code 会**先执行**该命令、把输出**替换**进正文，再让 Claude 看到。于是技能拿到的是「当前真实数据」而不是命令本身。例：`` !`git diff HEAD` ``。
- **参数**：`$ARGUMENTS` 拿全部参数；`$0`/`$1` 或 `$ARGUMENTS[0]` 按位取。`/fix-issue 123` → 把 `$ARGUMENTS` 换成 `123`。

### 3.3 怎么创建

三步（以个人级为例）：

```bash
# 1. 建目录（目录名 = 命令名）
mkdir -p ~/.claude/skills/summarize-changes
```

```markdown
# 2. 写 ~/.claude/skills/summarize-changes/SKILL.md
---
description: 总结未提交的改动并标出风险。当用户问"改了什么/要个 commit message/帮我看看 diff"时使用。
---

## 当前改动

!`git diff HEAD`

## 指令

把上面的改动用 2–3 个 bullet 总结，然后列出你注意到的风险（缺少错误处理、写死的值、需要更新的测试）。若 diff 为空，就说没有未提交的改动。
```

```text
# 3. 测试：改动一个文件后，问 "What did I change?"（自动触发）
#    或直接打 /summarize-changes（手动触发）
```

**可带附属文件**——让 `SKILL.md` 保持精简，大段参考按需读：

```text
my-skill/
├── SKILL.md        # 入口（必需）
├── reference.md    # 详细参考（需要时才读）
└── scripts/
    └── helper.py   # 可执行脚本（执行，不进上下文）
```

在 `SKILL.md` 里用链接引用它们，Claude 才知道何时去读。建议 `SKILL.md` 控制在 **500 行内**。

### 3.4 什么场景用

✅ 适合：
- 你**反复粘贴**的同一套多步流程（发版、生成 commit、审查清单）
- 偶尔才需要、但**正文很长**的参考资料（不用时零成本）
- CLAUDE.md 里**长成了流程**的那一段
- 想在**隔离上下文**里跑、不污染主对话的任务（`context: fork`）

❌ 不适合：
- 每次会话都要在场的简短事实 → CLAUDE.md（Skill 默认不在场）
- 必须**确定性**发生、不能靠模型自觉的动作 → Hook

---

## 4. Hooks

### 4.1 是什么

Hook 是在 Claude Code **生命周期事件**上触发的**外部命令**（最常见是 shell 命令）。和前两者最大的不同：它是**确定性**的——**不进上下文、不靠模型判断**，事件一到就执行，**无论模型当时想干嘛**。

所以 Hook 是你唯一的「硬约束」手段：要「提交前必跑测试」「禁止某类命令」「每次编辑后自动格式化」这种**不容商量**的规则，只能靠它。

### 4.2 怎么用（配在哪 + 结构 + 怎么拦截）

**配在哪**（`settings.json` 的 `hooks` 键）：

| 位置 | 作用域 | 可共享 |
| :-- | :-- | :-- |
| `~/.claude/settings.json` | 所有项目 | 否 |
| `.claude/settings.json` | 单个项目 | 是（入库） |
| `.claude/settings.local.json` | 单个项目 | 否（gitignore） |
| 插件 / 托管策略 | 视情况 | 是 |

**常用事件**（核心几个；完整事件集更大，**以官方文档为准，需查证**）：

| 事件 | 触发时机 | 能否拦截 |
| :-- | :-- | :-- |
| `PreToolUse` | 工具调用**前** | ✅ 可阻止（最常用的「卡死」点） |
| `PostToolUse` | 工具调用**后** | 用于自动格式化/校验 |
| `UserPromptSubmit` | 你提交 prompt 时 | ✅ 可阻止/改写 |
| `Stop` | Claude 准备结束回合 | ✅ 可要求继续 |
| `SessionStart` / `SessionEnd` | 会话开始/结束 | 用于环境准备/收尾 |
| `PreCompact` / `PostCompact` | 上下文压缩前/后 | — |
| `Notification` | 需要通知时 | 用于自定义提醒 |

**配置结构**（三层嵌套：事件 → matcher 分组 → 处理器）：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/block-rm.sh",
            "timeout": 600
          }
        ]
      }
    ]
  }
}
```

- **matcher**：`"*"`/`""`/省略 = 匹配全部；`"Edit|Write"` = 列表；含特殊字符则按 JS 正则（如 `"mcp__memory__.*"`）。
- **hook 类型**：`command`（最常用）、`http`、`mcp_tool`、`prompt`、`agent`。

**怎么拦截 / 反馈**（退出码语义，**有坑**）：

| 退出码 | 含义 |
| :-- | :-- |
| `0` | 成功（stdout 视事件可作结构化输出） |
| **`2`** | **阻断**：stderr 作为错误回喂给 Claude，动作被拦截 |
| 其它非零 | **非阻断**错误：继续执行，仅在 transcript 显示首行 stderr |

> ⚠️ 官方明确警告：**多数事件只有退出码 2 会阻断**。Claude Code 把退出码 **1 当成"非阻断错误"并照样继续**——尽管 1 在 Unix 习惯里是失败码。别用 1 去拦截。

更精细的控制可在退出 0 时打印 JSON（如 `PreToolUse` 的 `permissionDecision: "deny"`）。注意：**退出 2 时 JSON 会被忽略**，二选一。

### 4.3 怎么创建

最小例子——**编辑后自动跑格式化**（项目级）。在 `.claude/settings.json`：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "pnpm prettier --write \"$CLAUDE_FILE_PATHS\" 2>/dev/null || true" }
        ]
      }
    ]
  }
}
```

> 上例中 `$CLAUDE_FILE_PATHS` 等注入变量的**确切名字以官方文档为准（需查证）**——不同事件暴露的环境变量/JSON 输入不同。建议先写个把入参打到日志的脚本，确认字段后再正式用。

**拦截危险命令**——在 `.claude/hooks/block-rm.sh`（`chmod +x`），配在 `PreToolUse` + `matcher: "Bash"`：

```bash
#!/usr/bin/env bash
input=$(cat)                       # stdin 收到本次工具调用的 JSON
if echo "$input" | grep -qE 'rm +-rf'; then
  echo "拒绝：检测到 rm -rf，已被 Hook 阻断" >&2
  exit 2                           # 退出码 2 = 阻断
fi
exit 0
```

创建后用 `/hooks` 菜单或重启确认已加载（**需查证**具体刷新方式）。

### 4.4 什么场景用

✅ 适合：
- **必须确定性发生**的事：提交/编辑后自动 lint/format/test
- **安全护栏**：拦截 `rm -rf`、阻止改某些路径、禁用某类工具
- 与外部系统集成：发通知、写审计日志、触发 CI
- 「在某个固定时点」必须跑的动作（CLAUDE.md 的软提醒做不到）

❌ 不适合：
- 需要**判断/灵活性**的工作（Hook 是死命令，不会"看情况") → 交给模型 + Skill
- 只是"希望"而非"必须"的偏好 → CLAUDE.md 更省事

---

## 5. 三者怎么选

### 决策路径

```
你要加的东西是……

1）一条"必须发生、不容模型商量"的动作？
   （提交前必跑测试、禁止某命令、编辑后必格式化）
        └─ 是 → Hook（唯一的硬约束手段）

2）一套"偶尔触发的多步流程"或"大段按需参考"？
   （发版、审 PR、整理文档、长 API 规格）
        └─ 是 → Skill（按需加载，不用时零成本）

3）一条"每次会话都该记得的简短事实/规矩"？
   （构建命令、目录约定、命名规范、永远别 push）
        └─ 是 → CLAUDE.md（常驻在场）

4）只在代码库某一小块才相关的规则？
        └─ 是 → .claude/rules/（带 paths，按文件加载）
```

### 三个判别问句

1. **「必须」还是「最好」？** 必须 → Hook（硬）；最好 → CLAUDE.md / Skill（软）。
2. **「每次在场」还是「按需出现」？** 每次 → CLAUDE.md（但要短）；按需 → Skill（可以长）。
3. **「事实」还是「流程」？** 事实/约定 → CLAUDE.md；多步流程 → Skill。

### 常见组合

它们不是互斥的，最佳实践是**搭配**：
- **CLAUDE.md（预防）+ Skill（诊疗）**：CLAUDE.md 写一行"新文档放对位置"防止再乱；`tidy-docs` Skill 负责把已经乱的清理一遍。
- **Skill（流程）+ Hook（兜底）**：`/commit` Skill 走提交流程；`PreToolUse` Hook 兜底拦截危险命令，防止流程被绕过。
- **CLAUDE.md（软提醒）+ Hook（硬执行）**：CLAUDE.md 写"提交前跑 lint"让模型主动做；Hook 在提交前强制跑，模型忘了也跑。

---

## 6. 实战案例：从 0 创建 `tidy-docs` Skill

把前面的理论走一遍真实流程：你的项目里 Markdown 文档散落各处（有的在 `ai-service/`，有的在 Docker 目录，有的在别的项目里），你想要一套「**审计 + 分类（保留/删除/移动）+ 给出目录建议**」的可复用流程。

### 6.1 为什么是 Skill（而不是 CLAUDE.md / Hook）

对照第 5 节的判别：

- 它是**多步流程**（扫描→分类→建议→确认→执行），不是一条简短事实 → 不该塞进 CLAUDE.md（会撑大常驻上下文，且流程写在 CLAUDE.md 里很别扭）。
- 它需要**判断**（哪些文档冗余、该放哪），不是确定性死命令 → 不该做成 Hook。
- 它**偶尔才用**、用时希望有完整流程指引、不用时不占成本 → **正中 Skill 的定位**。

> 顺带一个「治未病」的搭配：可在 `~/.claude/CLAUDE.md` 追加 2–3 行文档约定（如「新文档默认放 `docs/`，README 只留入口」）。**为什么**？因为 Skill 是「治已病」——你叫它时它清理一次；而 CLAUDE.md 约定常驻在场，能在**每次新建文档时**就减少乱放，从源头降低复发。两者配合：约定防新乱，Skill 治旧乱。

### 6.2 选层级：个人 vs 项目

- 想**所有项目通用** → 个人级 `~/.claude/skills/tidy-docs/`。
- 想**随某仓库共享给团队**、且规则带本仓库特性 → 项目级 `.claude/skills/tidy-docs/`（入库）。

整理文档是通用诉求，这里用**个人级**。

### 6.3 写 frontmatter（让模型/你都能触发）

```bash
mkdir -p ~/.claude/skills/tidy-docs
```

`~/.claude/skills/tidy-docs/SKILL.md` 的骨架：

```markdown
---
name: tidy-docs
description: 审计并重整仓库内所有 Markdown 文档——分类「保留/删除/移动」，提出 docs/ 结构，先出方案、确认后再执行。当用户说"整理文档 / markdown 太乱 / 文档该归档了"时触发。
---
```

要点：
- `description` 把**关键用例 + 触发短语**写在前面，模型才好命中自动触发。
- 这里**不加** `disable-model-invocation`——既允许你 `/tidy-docs` 手动叫，也允许模型在你抱怨"文档好乱"时主动提议。若你只想手动掌控，可加 `disable-model-invocation: true`。
- `name` 其实可省（默认取目录名 `tidy-docs`），写上只是更直观。

### 6.4 写正文（流程提示词）

frontmatter 之下就是「触发后照着做」的指令。把整理流程结构化成几块：

```markdown
你是文档整理助手。目标：让仓库里所有 Markdown 直观清晰、各归其位。

## 范围
- 扫描全仓库 `.md` / `.markdown`。
- 排除：`node_modules`、`build`、`dist`、`.dart_tool`、`vendor`、`.git`。

## 产出（先给方案，不要直接动手）
1. **清单表**：路径 | 作用一句话 | 建议（保留/删除/移动）| 理由。
2. **分类**：核心文档 / 过时冗余 / 放错位置 / 内容可瘦身。
3. **目录建议**：提议统一的 `docs/` 结构，并列出每个文件的目标位置。
4. **内容瘦身**：指出哪些文档内容重复或过时、可删减的段落。
5. **风险标注**：被别处引用的文档、可能的死链。

## 执行约束（硬性）
- **先出方案，经我确认后再改**。
- 删除前**再列一遍**待删清单让我确认。
- 移动文件时**同步修复**指向它的链接/引用。

## 输出格式
先一张总览表，再按「保留/移动/删除」分组详述，最后给「确认后我将执行的操作」清单。
```

> 可选增强：用**动态注入**让技能拿到真实文件树而不是凭空猜。例如在正文加一行
> `` 当前 Markdown 清单：!`git ls-files "*.md"` ``
> Claude Code 会先跑 `git ls-files`、把结果替换进正文，技能于是基于真实文件列表工作。

### 6.5 测试与迭代

```text
# 自动触发（匹配 description）
我项目里的 markdown 文档太乱了，帮我整理一下

# 手动触发
/tidy-docs
```

迭代要点（官方建议）：
- 若**该触发却没触发** → description 里补用户会自然说的关键词。
- 若**触发太频繁** → description 写具体些，或加 `disable-model-invocation: true` 只留手动。
- 改完 `SKILL.md` **当前会话即时生效**（个人/项目 skills 目录被监听），无需重启；但**新建顶层 skills 目录**需重启才会被监听。

---

## 7. 附录

### 7.1 常见坑

- **CLAUDE.md 太长**：超 200 行会稀释、降低遵守率。拆 `.claude/rules/`（带 `paths`）或精简。
- **以为 CLAUDE.md 能强制**：它是上下文不是强制层。要硬拦截，用 Hook。
- **Skill 命令名搞错**：命令名来自**目录名**，不是 frontmatter `name`（插件根 SKILL.md 例外）。
- **Hook 用退出码 1 想拦截**：拦不住，多数事件**只有退出码 2 阻断**，1 被当非阻断错误照常继续。
- **Skill 正文当一次性步骤写**：技能正文进上下文后**整个会话留存**，要写成「全程适用的常驻指令」而非「只做一次的步骤」。

### 7.2 一句话回顾

| | 关键词 | 记住这句 |
| :-- | :-- | :-- |
| CLAUDE.md | 常驻 · 软 · 事实 | 「永远记住的短规矩，写短点」 |
| Skills | 按需 · 软 · 流程 | 「需要时叫来的流程，长也没事」 |
| Hooks | 触发 · 硬 · 动作 | 「必须发生的动作，靠代码不靠自觉」 |

### 7.3 官方文档（以这些为准）

- 记忆 / CLAUDE.md：`https://code.claude.com/docs/en/memory`
- Skills：`https://code.claude.com/docs/en/skills`
- Hooks：`https://code.claude.com/docs/en/hooks`（及 `/en/hooks-guide`）
- 设置 / 权限：`https://code.claude.com/docs/en/settings`、`/en/permissions`

> 本文写于 2026-06，基于上述官方文档。Claude Code 更新频繁，**字段名、Hook 事件名、注入变量名以你本机版本对应的官方文档为准**。
