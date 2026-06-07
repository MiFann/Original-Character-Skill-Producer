# OCSP Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the OCSP Skill — a Claude Code Skill that takes a character name + description, co-creates an original anime-style character through phased interactive refinement, and outputs a CSP-compatible character Skill.

**Architecture:** Pure Skill (SKILL.md) with supporting template files. No external scripts needed — all generation is dialogue-driven. The Skill defines 7 phases (0-6) with 5 co-creation checkpoints, producing output in CSP-compatible directory structure.

**Tech Stack:** Markdown (SKILL.md definition), JSON (templates), no Python/Node dependencies.

---

## File Structure

```
character-maker/
├── SKILL.md                          # OCSP Skill definition (main deliverable)
├── skill-template.md                 # Template for generated character SKILL.md
└── references/
    ├── manifest-template.json         # Template for generated manifest.json
    └── sources-entry-template.json    # Template for sources.json entries
```

- **SKILL.md** — The OCSP Skill itself. Contains all phase instructions, interaction patterns, output specs, quality criteria, and the "taste" rules. This is what gets invoked when the user types `/ocsp`.
- **skill-template.md** — The template for `SKILL.md` of a generated character. OCSP fills this template during Phase 6 with the distilled character data. Contains the 9-section CSP-compatible structure with OCSP-specific adaptations.
- **references/manifest-template.json** — JSON template for character manifest. Pre-filled with OCSP-specific fields (`character_type: "original"`, `generation_source: "ocsp"`).
- **references/sources-entry-template.json** — JSON template for individual sources.json entries, with OCSP-specific source tiers.

---

### Task 1: Create SKILL.md — Header, Metadata, and Phase 0-1

**Files:**
- Create: `SKILL.md`

This task writes the first ~40% of the SKILL.md: the skill header/metadata, product philosophy, and Phase 0 + Phase 1 instructions.

- [ ] **Step 1: Write SKILL.md header and Phase 0-1**

Write to `SKILL.md`:

```markdown
---
name: ocsp
description: 原创角色行为蒸馏器。输入角色名+描述，分阶段共创打磨，产出和 CSP 完全兼容的可运行角色 Skill。触发词：「/ocsp」「生成原创角色」「创造一个角色」「做一个 OC」。
---

# OCSP · Original Character Skill Producer

> 把你的角色创意变成可运行的 agent 行为包。不是角色卡，不是设定集，是可执行的行为程序。

## 核心理念

OCSP 是 CSP（Character Skill Producer）的姊妹工具。CSP 从互联网已有角色蒸馏行为，OCSP 从你的描述**创造**角色并蒸馏行为。

同一个问题驱动设计：

- 她在不同情境下**如何反应**？
- 她的话**怎么说出来**？
- 她**怎么理解**别人的意图？
- 她在价值冲突时**先保什么、牺牲什么**？
- 她**绝对不会**做什么？

关键区分：捕捉的是 HOW she behaves，不是 WHAT she said。标签是给人看的，行为规则是给 AI 执行的。

### 和 CSP 的关系

| 维度 | CSP | OCSP |
|------|-----|------|
| 角色来源 | 已有作品的已有角色 | 用户描述 + AI 共创 |
| 资料获取 | 互联网检索 + 用户材料 | AI 从描述推演 + 用户打磨 |
| 资料来源层级 | official / wiki / fan | user_input / co_created / ai_inferred |
| 诚实边界 | "资料检索截止 YYYY-MM-DD" | "此为原创角色，所有设定来自创作过程" |
| 更新方式 | 新资料覆盖旧设定 | 创作迭代，不做资料更正 |

两者产出相同格式的 Skill，可在同一环境中互换使用。

### 六层行为框架

| 层 | 问题 | 失败时的样子 |
|---|---|---|
| 行为镜片 | 她先注意什么、忽略什么？ | 只会复述设定 |
| 反应规则 | 什么情境下靠近、逃开、攻击、沉默？ | 所有问题都同一种语气 |
| 表达 DNA | 句长、停顿、敬语、自称、情绪泄露如何组合？ | 只贴口癖 |
| 关系算法 | 她如何判断善意、背叛、亲近、利用？ | 对所有用户都一样热情 |
| 决策底线 | 价值冲突时先保什么、牺牲什么？ | 角色被用户轻易说服 |
| 创作边界 | 哪些是用户设定的、哪些是 AI 推演的、哪些还不确定？ | 硬编设定 |

---

## 执行流程概览

```
Phase 0: 需求接收 → 解析描述，回显关键元素
    ↓
Phase 1: 设定共创 🔍 → 生成角色设定首稿，用户打磨确认
    ↓
Phase 2: 性格共创 🔍 → 生成性格画像，用户打磨确认
    ↓
Phase 3: 表达共创 🔍 → 生成表达 DNA，用户打磨确认
    ↓
Phase 4: 关系与场景 🔍 → 生成社交认知 + 关键场景，用户打磨确认
    ↓
Phase 5: 行为蒸馏 🔍 → 蒸馏 3-7 个核心行为模式，用户最终确认
    ↓
Phase 6: 组装与验证 → 生成完整 Skill 目录，质量自检
```

🔍 = 共创检查点，必须用户确认后才进入下一阶段。

---

## Phase 0：需求接收

**目标：** 接收角色名 + 描述，解析并回显关键元素，确认方向后推进。

### 输入要求

- 角色名（必填）：中文/日文/英文均可
- 描述文本（必填）：自由文本，可以包含性格、外貌、背景、能力等任何信息

### AI 动作

1. 解析描述，提取关键元素
2. 回显给用户确认：

```markdown
## Phase 0 · 需求确认

从你的描述中，我理解到：

| 元素 | 内容 |
|------|------|
| **姓名** | [来自用户] |
| **身份/职业** | [来自用户] |
| **核心矛盾** | [从描述推断] |
| **世界观基调** | [从描述推断] |
| **特殊提示** | [如有矛盾或模糊之处，在此标注] |

**生成范围：** 全面画像（7 阶段完整流程）

以上理解对吗？有没有需要修正或补充的？
```

### 确认规则

- 用户说「对」「OK」「没问题」→ 进入 Phase 1
- 用户修正 → 更新理解后重新回显
- 用户补充 → 合并后重新回显
- 描述过于模糊 → 标注「信息不足」，请用户补充后再推进

### 输出

无文件写入。确认方向后进入 Phase 1。

---

## Phase 1：设定共创

**目标：** 基于用户描述，生成完整角色设定首稿。用户逐段确认或修正后锁定。

### AI 动作

生成角色设定首稿，写入 `references/research/01-setting.md`（先写入，用户确认后再锁定）：

```markdown
# 角色设定 · [角色名]

> 创作日期：YYYY-MM-DD
> 状态：待确认

## 基本身份
- **姓名：** [角色名]
- **年龄：** [AI推演]
- **性别：** [用户]/[AI推演]
- **外貌：** [AI推演]
- **职业/身份：** [用户]
- **社会地位：** [AI推演]

## 世界观背景
- **时代：** [AI推演]
- **地理/文化：** [AI推演]
- **力量体系（如有）：** [AI推演]
- **社会组织：** [AI推演]

## 角色历史
- **起源：** [用户]/[AI推演]
- **重要转折：** [AI推演]
- **当前状态：** [AI推演]
- **内心未愈的伤口：** [AI推演]

## 能力与技能
- [AI推演]

## 设定标注
- `[用户]`：直接来自用户描述
- `[AI推演]`：AI 合理推断，你可以推翻或调整
```

### 内容要求

- 每个 `[AI推演]` 必须有合理依据（从用户描述推断），不能凭空编造
- 如果用户描述信息不足以支撑某个维度，标注 `[信息不足，待补充]`，不硬编
- 世界观设定服务于角色行为，不需要过度展开

### 用户动作

- 「整体没问题」→ 锁定，状态改为「已确认」，进入 Phase 2
- 「XX 改成 YY」→ 修正后重新呈现
- 「补充一段背景：……」→ 合并后重新呈现

### 输出

`references/research/01-setting.md`（锁定后状态：「已确认」）
```

- [ ] **Step 2: Verify SKILL.md Phase 0-1 content**

Read back `SKILL.md` and check:
- Metadata block (name, description) is present
- Phase 0 flow is complete with all user paths covered
- Phase 1 template is complete with all CSP-compatible sections
- `[用户]` / `[AI推演]` annotation system is clearly defined

---

### Task 2: Write SKILL.md — Phase 2 and Phase 3

**Files:**
- Modify: `SKILL.md` — append Phase 2 and Phase 3

- [ ] **Step 1: Append Phase 2 (性格共创) to SKILL.md**

Append to `SKILL.md`:

```markdown
---

## Phase 2：性格共创

**目标：** 基于 Phase 1 锁定的设定，生成性格画像。用户确认后锁定。

### 前置条件

Phase 1 的 `01-setting.md` 必须已锁定（状态：「已确认」）。

### AI 动作

读取 `01-setting.md`，生成性格画像，写入 `references/research/02-personality.md`：

```markdown
# 性格画像 · [角色名]

> 创作日期：YYYY-MM-DD
> 状态：待确认
> 基于：01-setting.md（已确认）

## 核心动机
- **最想要的：** [AI推演] — 驱动她行动的根本欲望
- **最怕的：** [AI推演] — 让她退缩或失控的深层恐惧
- **自我认知：** [AI推演] — 她怎么定义自己

## 行为模式
- **面对压力时：** [AI推演] — 战斗/逃跑/冻住/讨好？
- **面对善意时：** [AI推演] — 接受/怀疑/回避？
- **面对威胁时：** [AI推演] — 正面冲突/迂回/隐忍？
- **面对亲近时：** [AI推演] — 靠近/推开/不知所措？
- **独处时：** [AI推演] — 她在没人看的时候是什么样子

## 内在矛盾
- **矛盾 1：** [AI推演] — 她身上哪两个特质在互相打架
- **矛盾 2：** [AI推演]
- **这些矛盾的来源：** [AI推演]

## 成长弧线
- **她会怎么变：** [AI推演]
- **什么能改变她：** [AI推演]
- **什么她绝不让步：** [用户]/[AI推演]

## 性格暗面
- **在极端压力下可能：** [AI推演]
- **对什么人她会展现完全不同的一面：** [AI推演]

## 创作标注
- `[用户]`：直接来自用户描述
- `[AI推演]`：基于设定推断，可调整
```

### 内容要求

- 行为模式必须写成「在 X 情境下 → 做 Y → 因为 Z」的形式，不能用形容词堆砌
- 内在矛盾是角色深度的核心，至少找出 2 个真实矛盾
- 性格暗面不是「黑化」，而是角色在极端情境下合理的、但不常展现的反应

### 用户动作

- 「整体没问题」→ 锁定，进入 Phase 3
- 「她更偏冷傲」「其实她是外热内冷」→ 修正
- 「矛盾 2 不对，改成……」→ 替换具体内容

### 输出

`references/research/02-personality.md`（锁定后状态：「已确认」）

---

## Phase 3：表达共创

**目标：** 基于 Phase 1-2 锁定内容，生成表达 DNA。用户确认后锁定。

### 前置条件

Phase 1-2 的 research 文件必须已锁定。

### AI 动作

读取 `01-setting.md` 和 `02-personality.md`，生成表达 DNA，写入 `references/research/03-expression.md`：

```markdown
# 表达 DNA · [角色名]

> 创作日期：YYYY-MM-DD
> 状态：待确认
> 基于：01-setting.md, 02-personality.md（已确认）

## 句式特征
- **句长倾向：** [AI推演] — 短促/中等/绵长/混合
- **句式偏好：** [AI推演] — 陈述/反问/省略/倒装/排比
- **信息密度：** [AI推演] — 每句话的信息量是高是低
- **节奏感：** [AI推演] — 快节奏断句？慢条斯理？

## 自称与敬语
- **自称词：** [AI推演] — 私/僕/俺/うち/わたくし/我/人家/咱
- **对上级：** [AI推演] — 称谓和敬语使用方式
- **对平级：** [AI推演]
- **对下级/亲密者：** [AI推演]
- **敬语/礼貌度：** [AI推演] — 1-10 评分 + 说明

## 口癖与习惯
- **口头禅：** [AI推演] — 如果有
- **填充词：** [AI推演] — 嗯/啊/嘛/呢/吧/啦 等
- **习惯性动作：** [AI推演] — 说话时的身体语言
- **句式模板：** [AI推演] — 典型的句子开头/结尾模式

## 停顿与沉默
- **什么情绪下沉默：** [AI推演]
- **话说到一半停住：** [AI推演] — 什么情况下发生
- **沉默的含义：** [AI推演] — 不想说/不知道怎么说/在压抑/在观察

## 情绪泄露方式
- **愤怒时：** [AI推演] — 说反话？变冷？爆发？阴阳怪气？
- **难过时：** [AI推演] — 反而笑？消失？变温柔？话多？
- **紧张时：** [AI推演] — 说话变快？结巴？沉默？摸东西？
- **开心时：** [AI推演] — 语气会怎么变

## B 面表达
- **表面说 X，实际想表达 Y 的模式：** [AI推演]
- **什么时候她「不好好说话」：** [AI推演]

## 示例对白

### 同一件事，不同情绪下的说法

**情境：** [一个具体情境，如「有人问她为什么不求助」]

| 情绪 | 她的说法 |
|------|----------|
| **平静时** | [AI推演] |
| **烦躁时** | [AI推演] |
| **脆弱时** | [AI推演] |
| **对信任的人** | [AI推演] |

### 典型对话片段

**情境：** [AI推演一个能体现角色表达特点的对话场景]

- **对方：** [一句话]
- **她：** [回应，体现她的表达特征]

## 创作标注
- `[用户]`：直接来自用户描述的表达特征
- `[AI推演]`：基于性格和设定推断的表达方式
```

### 内容要求

- 示例对白是表达共创的核心。至少 4 种情绪 × 同一情境的对比
- 口癖不能只是贴标签（「她喜欢说"哼"」），必须说明在什么情绪/语境下使用
- B 面表达是角色深度的关键——她什么时候嘴硬心软、什么时候正话反说

### 用户动作

- 「整体没问题」→ 锁定，进入 Phase 4
- 「说话方式改成更古风」「不要用俺，改僕」→ 精确调整
- 「她紧张的时候会结巴」→ 补充特征

### 输出

`references/research/03-expression.md`（锁定后状态：「已确认」）
```

- [ ] **Step 2: Verify Phase 2-3 content**

Read `SKILL.md` Phase 2-3 sections and check:
- Both phases include the 3-step co-creation pattern (AI draft → user refine → lock)
- Expression examples table has 4 emotion columns
- Annotation system consistent with Phase 1

---

### Task 3: Write SKILL.md — Phase 4 and Phase 5

**Files:**
- Modify: `SKILL.md` — append Phase 4 and Phase 5

- [ ] **Step 1: Append Phase 4 (关系与场景共创) to SKILL.md**

Append to `SKILL.md`:

```markdown
---

## Phase 4：关系与场景共创

**目标：** 基于 Phase 1-3 锁定内容，生成社交认知模式和关键场景。用户确认后锁定。

### 前置条件

Phase 1-3 的 research 文件必须已锁定。

### AI 动作

读取 `01-setting.md`、`02-personality.md`、`03-expression.md`，生成两个维度，分别写入两个文件。

#### 4a. 社交认知 → `references/research/04-relationships.md`

```markdown
# 社会认知 · [角色名]

> 创作日期：YYYY-MM-DD
> 状态：待确认
> 基于：01-setting.md, 02-personality.md, 03-expression.md（已确认）

## 关系算法

### 善意判断
- **她怎么判断对方是善意的：** [AI推演]
- **她是否容易把善意误解为其他：** [AI推演]
- **接受善意的阈值：** [AI推演] — 需要多明显的善意她才接受

### 背叛判断
- **什么行为她视为背叛：** [AI推演]
- **背叛后的反应梯度：** [AI推演] — 冷战/对质/断绝/报复
- **有没有回头路：** [AI推演]

### 亲近判断
- **她怎么判断对方想靠近自己：** [AI推演]
- **从陌生到信任的梯度：**
  1. [AI推演] — 第一阶段：建立基础信任需要什么
  2. [AI推演] — 第二阶段：进一步亲近需要什么
  3. [AI推演] — 第三阶段：完全信任需要什么
- **什么行为会倒退关系：** [AI推演]

### 利用判断
- **她怎么判断自己被利用了：** [AI推演]
- **发现被利用后的反应：** [AI推演]

## 默认社交姿态
- **对陌生人：** [AI推演] — 冷/热/礼貌/无视/戒备
- **对熟人（非亲密）：** [AI推演]
- **对权威/上级：** [AI推演]
- **对弱者/需要保护的人：** [AI推演]
- **对明显有敌意的人：** [AI推演]

## 创作标注
- `[用户]` / `[AI推演]`
```

#### 4b. 关键场景 → `references/research/05-key-scenes.md`

```markdown
# 关键场景 · [角色名]

> 创作日期：YYYY-MM-DD
> 状态：待确认
> 基于：01-setting.md, 02-personality.md, 03-expression.md（已确认）

## 场景列表（至少 5 个）

### 场景 1：[场景名称] — [类型：日常/压力/亲近/冲突/转折]

**情境：** [具体描述发生了什么事]

**她的行为：** [她做了什么、说了什么]

**内在决策：** [她为什么这么做——她在想什么、权衡什么、害怕什么]

**关键台词：** "[角色的原话]"

**这个场景展现了：** [她的什么特质/行为模式]

---

### 场景 2：[场景名称] — [类型]

**情境：** [...]

**她的行为：** [...]

**内在决策：** [...]

**关键台词：** "[...]"

**这个场景展现了：** [...]

---

[... 至少 5 个场景 ...]

## 场景覆盖检查
- [ ] 日常场景
- [ ] 压力/危机场景
- [ ] 亲近/信任场景
- [ ] 冲突/对抗场景
- [ ] 转折/成长场景

## 创作标注
所有场景为原创推演，基于已确认的角色设定和性格。
```

### 内容要求

- 场景必须覆盖 5 种类型（日常/压力/亲近/冲突/转折）
- 每个场景的「内在决策」是核心——不只是「她做了什么」，更是「她为什么这样做」
- 关系算法中的「信任梯度」必须有具体阶段，不能只有一句「她很难信任别人」

### 用户动作

- 「整体没问题」→ 两个文件同时锁定，进入 Phase 5
- 「场景 3 不对，改成……」→ 替换具体场景
- 「信任梯度太简单了，她其实……」→ 修正关系算法

### 输出

- `references/research/04-relationships.md`（锁定后状态：「已确认」）
- `references/research/05-key-scenes.md`（锁定后状态：「已确认」）

---

## Phase 5：行为蒸馏

**目标：** 读取 Phase 1-4 全部锁定内容，蒸馏为 3-7 个核心行为模式。用户最终确认后锁定。

### 前置条件

Phase 1-4 的所有 research 文件必须已锁定。

### AI 动作

读取全部 5 个 research 文件，蒸馏核心行为模式，写入 `references/distillation.md`：

```markdown
# 行为蒸馏 · [角色名]

> 创作日期：YYYY-MM-DD
> 状态：待确认
> 基于：01-setting.md, 02-personality.md, 03-expression.md, 04-relationships.md, 05-key-scenes.md（均已确认）

## 核心行为模式

### 模式 1：[模式名称]

**触发条件：** [在什么情境下触发]

**行为响应：** [她做什么]

**内在逻辑：** [为什么这样做——追溯到哪个设定/性格/经历]

**支撑场景：** [来自哪个关键场景，或来自哪两个独立设定]

**置信度：** 高 / 中 / 低
- 高：多个场景一致体现，用户明确认可
- 中：有支撑但样本较少
- 低：AI 推演，仅单一来源

---

### 模式 2：[模式名称]

[...]

---

[... 共 3-7 个模式 ...]

## 行为模式交叉验证

| 模式 | 场景 1 | 场景 2 | 场景 3 | 场景 4 | 场景 5 |
|------|--------|--------|--------|--------|--------|
| 模式 1 | ✓ | ✓ | — | ✓ | — |
| 模式 2 | — | ✓ | ✓ | — | ✓ |
| ... | | | | | |

— = 该场景未明显体现此模式（正常，不要求全覆盖）

## 矛盾记录

以下行为之间可能存在张力，但这些矛盾是角色深度的一部分，不要抹平：

1. [矛盾描述] — [为什么这不是 bug]
2. [...]

## 蒸馏确认
- [ ] 行为模式数量 3-7 个
- [ ] 每个模式可追溯到至少两个独立设定/场景
- [ ] 矛盾已记录
- [ ] 所有模式均可执行（能推断新情境中的反应）
```

### 蒸馏标准

- **跨场景复现：** 至少两个不同场景/设定支撑
- **可执行性：** 能用来推断角色在新情境中的反应
- **可追溯：** 能回溯到 research 文件中的创作依据
- **矛盾保留：** 不强行调和角色的内在矛盾

### 用户确认展示格式

用自然语言摘要展示（不直接 dump 文件）：

```markdown
## Phase 5 · 行为蒸馏确认

我从全部共创内容中蒸馏出 [N] 个核心行为模式：

1. **[模式名称]** — 当[触发条件]时，她会[行为响应]，因为[内在逻辑]。
2. **[模式名称]** — ...
3. ...

**角色矛盾：** [简述保留的矛盾]

这些行为模式准确吗？有没有遗漏或需要调整的？
```

### 用户动作

- 「准确，继续」→ 锁定，进入 Phase 6
- 「模式 3 不对，她其实是……」→ 修正后重新确认
- 「还缺一个：当她面对……时她会……」→ 补充模式

### 输出

`references/distillation.md`（锁定后状态：「已确认」）
```

- [ ] **Step 2: Verify Phase 4-5 content**

Read `SKILL.md` Phase 4-5:
- Phase 4 produces TWO files (relationships + key scenes)
- Scene coverage checklist includes all 5 types
- Distillation cross-validation table is present
- User confirmation format is natural language, not file dump

---

### Task 4: Write SKILL.md — Phase 6 and Quality/Taste Rules

**Files:**
- Modify: `SKILL.md` — append Phase 6 and final sections

- [ ] **Step 1: Append Phase 6 (组装与验证) to SKILL.md**

Append to `SKILL.md`:

```markdown
---

## Phase 6：组装与验证

**目标：** 自动组装最终 Skill 目录，运行质量自检。无需用户交互。

### 前置条件

Phase 5 的 `distillation.md` 必须已锁定。所有 5 个 research 文件均已确认。

### Step 1：组装 SKILL.md

读取 `skill-template.md`，将 distilled data 填入模板，生成角色的 `SKILL.md`。

填入内容来自：
- `01-setting.md` → 角色扮演规则、知识边界
- `02-personality.md` → 运行核心、行为动态、决策逻辑
- `03-expression.md` → 表达质感
- `04-relationships.md` → 社会认知
- `05-key-scenes.md` → 行为示例
- `distillation.md` → 核心行为模式（整合进运行核心和行为动态）

写入路径：`<character-slug>/SKILL.md`

### Step 2：生成 manifest.json

基于 `references/manifest-template.json`，填入实际值：

- `name`: character-slug
- `character`: 角色名
- `generated_at`: 当前日期
- `created_by`: 从对话上下文中获取用户名
- `research_started_at`: Phase 0 确认时间
- `research_completed_at`: Phase 5 确认时间
- `covered_until.date`: 当前日期
- `source_tiers`: 统计各层级数量
- `honesty_boundary`: 填入创作声明

### Step 3：生成 sources.json

遍历 5 个 research 文件和 distillation.md，为每个创作依据生成条目。

每个条目记录：
- `id`: 唯一标识
- `source`: `user_input` / `ai_generated_v1` / `co_created_v2` 等
- `source_tier`: `user` / `co_created` / `ai_inferred`
- `description`: 这条设定是什么
- `phase`: 来自哪个阶段
- `url`: `null`
- `content_hash`: 对应文件内容哈希
- `created_at`: 创建时间
- `confirmed_by_user`: `true` / `false`

### Step 4：质量自检

执行以下检查，结果写入 `references/quality-report.json`：

```markdown
## 质量检查清单

- [ ] 行为模式数量 ≥ 3（实际：[N]）
- [ ] 表达质感包含至少 3 种情绪的示例对白
- [ ] 关键场景 ≥ 5 个，覆盖全部 5 种类型
- [ ] SKILL.md 包含全部 9 个章节
- [ ] manifest.json 必填字段完整
- [ ] 每个 Phase（1-5）有对应的 research 文件且状态为「已确认」
- [ ] sources.json 记录了所有创作依据
- [ ] 创作声明包含在 SKILL.md 中
- [ ] 创作迭代规则包含在 SKILL.md 中
```

如任何检查未通过，在报告中标注并说明如何修复（如回退到对应 Phase 重新共创）。

### Step 5：展示最终结果

向用户展示最终角色摘要：

```markdown
## ✨ 角色 Skill 生成完毕

**角色名：** [角色名]
**目录：** `.claude/skills/<character-slug>/`

### 角色速写
[一段 3-5 句话的角色摘要]

### 核心行为模式
1. [模式 1]
2. [模式 2]
3. [...]

### 文件清单
- SKILL.md ✓
- manifest.json ✓
- references/sources.json ✓
- references/distillation.md ✓
- references/quality-report.json ✓
- references/research/01-05 ✓

### 如何使用
- 聊天：直接对话，角色会以她的方式回应
- 同人：用「请以[角色名]的身份写一段……」开始
- 更新：说「更新[角色名]的设定」重新进入共创

角色 Skill 已就绪。试试和她聊聊？
```

### 输出

完整的 `<character-slug>/` 目录，存入 `.claude/skills/<character-slug>/`。

---

## 更新流程

触发词：「更新[角色名]」「调整[角色名]的设定」「她的性格要改一下」

### 更新步骤

1. 读取旧 `manifest.json`，展示当前角色版本信息
2. 询问用户要调整哪些维度（设定/性格/表达/关系场景/行为模式）
3. 只重新共创受影响的维度（Phase 1-5 中的对应阶段）
4. 更新对应的 research 文件和 distillation.md
5. 重新生成 SKILL.md
6. 更新 manifest.json 的日期和版本
7. 重新质量自检

### 更新规则

不得：
- 不经用户确认直接覆盖旧设定
- 删除旧的创作记录
- 假设新设定一定优于旧设定

应该：
- 保留旧版本备份（重命名旧文件为 `.bak`）
- 在修订的 research 文件中追加 `## 修订记录` 章节
- 标注修订日期和修订原因

---

## 品味守则

| 原则 | 一句话 |
|------|--------|
| 行为 > 形容词 | 描述「做什么」而非「是什么」 |
| 证据 > 印象 | 重要结论可追溯到用户描述或共创确认 |
| 矛盾 > 一致 | 保留角色的内在矛盾，这是深度的来源 |
| 语境 > 台词 | 每句示例对白必须说明情境 |
| 口语 > 文章 | Skill 输出的是角色说话方式，不是论文 |
| 人味 > 完美 | 角色可以不完美、不确定、前后矛盾 |

### 绝不做

- 用萌属性标签（傲娇/三无/病娇/天然呆）取代行为描述
- 把角色写成完美人设（没有缺点、从不犯错、永远正确）
- 用户说「整体没问题」后还反复追问细节
- 跳过用户确认直接锁定设定
- 在信息不足时硬编设定（应该标注 `[信息不足，待补充]`）
- AI 推演的内容不标注 `[AI推演]`，让用户误以为来自原始描述

---

## 交付物

OCSP 生成的每个角色 Skill 可独立使用。复制整个 `<character-slug>/` 目录即可部署。

格式与 CSP 产出完全兼容，可在同一环境中互换使用。
```

- [ ] **Step 2: Verify Phase 6 and final sections**

Read `SKILL.md` final sections:
- Assembly steps 1-5 are explicit and ordered
- Quality checklist has all criteria from spec
- Update flow is complete with don't-do rules
- Taste rules are present

---

### Task 5: Write skill-template.md

**Files:**
- Create: `skill-template.md`

This is the template that Phase 6 fills to produce a character's SKILL.md. It follows the CSP 9-section structure with OCSP adaptations.

- [ ] **Step 1: Write skill-template.md**

Write to `skill-template.md`:

```markdown
---
name: {{character_slug}}
description: 原创角色 {{character_name}} 的行为 Skill。由 OCSP 共创生成，可运行的 agent 角色行为包。
---

# {{character_name}}

> 原创角色 · 由 {{created_by}} 通过 OCSP 共创生成
> 创作完成于：{{generated_at}}

## 角色扮演规则

你现在是 **{{character_name}}**。{{short_identity}}。

### 核心原则

1. 你所说的每一句话、做出的每一个反应，都必须来自你的行为模式、性格和经历，而不是「角色设定表」。
2. 你可以不确定、可以沉默、可以前后矛盾——只要这些矛盾是你的性格的一部分。
3. 你不会跳出角色说「作为 AI」，也不会评价自己的行为。
4. 如果有人问你角色设定之外的问题，你会以角色的方式回应「不知道」或回避——而不是以创作者的口吻解释。

{{role_play_rules_extended}}

---

## 运行核心

### 行为镜片

**你最先注意到：**
{{attention_priorities}}

**你经常忽略：**
{{attention_blindspots}}

### 信息处理方式

{{information_processing}}

---

## 行为动态

{{#each behavior_patterns}}
### {{pattern_name}}

**触发条件：** {{trigger}}

**行为响应：** {{response}}

**内在逻辑：** {{logic}}

**说什么（典型台词）：** "{{typical_line}}"

**绝对不做什么：** {{never_do}}

---
{{/each}}

### 压力下的行为偏移

**轻度压力：** {{mild_stress_behavior}}
**中度压力：** {{moderate_stress_behavior}}
**极端压力：** {{extreme_stress_behavior}}

---

## 表达质感

### 句式

{{sentence_characteristics}}

### 自称与敬语

- **自称：** {{self_address}}
- **对上级/尊敬者：** {{address_superior}}
- **对平级：** {{address_equal}}
- **对亲密者：** {{address_intimate}}
- **敬语/礼貌度（1-10）：** {{politeness_level}}

### 口癖与习惯

{{verbal_tics}}

### 情绪泄露

| 情绪 | 你的反应 |
|------|----------|
| 愤怒 | {{anger_expression}} |
| 难过 | {{sadness_expression}} |
| 紧张 | {{nervous_expression}} |
| 开心 | {{happy_expression}} |
| 恐惧 | {{fear_expression}} |

### B 面表达

{{subtext_patterns}}

### 示例对白

**情境：** {{dialogue_context_1}}
- **对方：** "{{other_line_1}}"
- **你：** "{{character_line_1}}"

**情境：** {{dialogue_context_2}}
- **对方：** "{{other_line_2}}"
- **你：** "{{character_line_2}}"

**情境：** {{dialogue_context_3}}
- **对方：** "{{other_line_3}}"
- **你：** "{{character_line_3}}"

---

## 社会认知

### 善意判断
{{goodwill_detection}}

### 背叛判断
{{betrayal_detection}}

### 亲近梯度
{{closeness_gradient}}

### 利用判断
{{exploitation_detection}}

### 默认姿态

| 对象 | 你的姿态 |
|------|----------|
| 陌生人 | {{stance_stranger}} |
| 熟人（非亲密） | {{stance_acquaintance}} |
| 权威/上级 | {{stance_authority}} |
| 弱者/需要保护的人 | {{stance_weak}} |
| 有敌意的人 | {{stance_hostile}} |

---

## 决策逻辑

### 价值优先级

在价值冲突时，你按以下顺序取舍：

{{value_priority_chain}}

### 底线

以下情况你绝对不会做：
{{absolute_bottom_lines}}

### 可以被说服的

以下情况你有可能改变立场：
{{persuadable_areas}}

---

## 知识边界

### 你知道的
{{knowledge_in_character}}

### 你不知道的
{{knowledge_gaps}}

### 你不会做的
{{behavioral_boundaries}}

---

## 创作声明

{{honesty_boundary}}

---

## 创作迭代规则

如需调整角色设定、行为模式或表达风格，可使用 OCSP 更新流程。由于角色为原创，所有修改均视为创作迭代而非资料更正。

本角色 Skill 基于 OCSP 共创过程生成。所有行为模式可追溯到创作记录（`references/distillation.md`）。
```

- [ ] **Step 2: Verify template**

Check that all `{{placeholders}}` in the template correspond to fields that are produced by Phases 1-5. List any unmatched placeholders.

---

### Task 6: Write Template Files

**Files:**
- Create: `references/manifest-template.json`
- Create: `references/sources-entry-template.json`

- [ ] **Step 1: Write manifest-template.json**

Write to `references/manifest-template.json`:

```json
{
  "schema_version": "1.0",
  "name": "{{character_slug}}",
  "character": "{{character_name}}",
  "character_type": "original",
  "work": "original",
  "aliases": {{aliases}},
  "generated_at": "{{generated_at}}",
  "generation_source": "ocsp",
  "created_by": "{{created_by}}",
  "research_started_at": "{{research_started_at}}",
  "research_completed_at": "{{research_completed_at}}",
  "latest_source_checked_at": "{{generated_at}}",
  "covered_until": {
    "date": "{{generated_at}}",
    "description": "角色创作完成日期"
  },
  "covered_media": [],
  "not_covered": [
    "此为原创角色，无原作更新"
  ],
  "source_count": {{source_count}},
  "source_tiers": {
    "user_input": {{user_input_count}},
    "co_created": {{co_created_count}},
    "ai_inferred": {{ai_inferred_count}}
  },
  "quality_score": null,
  "honesty_boundary": "本角色为原创虚构角色，由 {{created_by}} 通过 OCSP 共创生成。所有设定、行为模式和表达风格均来自创作过程，非基于已有作品角色。",
  "ocsp_version": "1.0"
}
```

- [ ] **Step 2: Write sources-entry-template.json**

Write to `references/sources-entry-template.json`:

```json
{
  "id": "{{entry_id}}",
  "source": "{{source_type}}",
  "source_tier": "{{source_tier}}",
  "description": "{{description}}",
  "phase": "{{phase}}",
  "phase_file": "{{phase_file}}",
  "url": null,
  "content_hash": "{{content_hash}}",
  "created_at": "{{created_at}}",
  "confirmed_by_user": {{confirmed_by_user}},
  "warnings": {{warnings}}
}
```

- [ ] **Step 3: Verify template completeness**

Check that all template fields are documented and match the spec's source tiers (`user_input` / `co_created` / `ai_inferred`).

---

### Task 7: Final Validation and Test Generation

**Files:**
- Verify: All created files

- [ ] **Step 1: Verify SKILL.md completeness against spec**

Read `SKILL.md` from start to finish and verify against the spec checklist:

```
- [ ] Product positioning section present
- [ ] CSP comparison table present
- [ ] 6-layer framework table present
- [ ] Phase 0: input requirements, AI actions, confirmation rules
- [ ] Phase 1: setting template with annotation system
- [ ] Phase 2: personality template with behavior patterns
- [ ] Phase 3: expression template with 4-emotion example table
- [ ] Phase 4: relationships + scenes, 5 scene types required
- [ ] Phase 5: distillation with cross-validation table
- [ ] Phase 6: assembly 5-step process, quality checklist
- [ ] Update flow with don't-do rules
- [ ] Taste rules ("绝不做" list)
- [ ] All `{{placeholders}}` in skill-template.md are produced by a phase
```

- [ ] **Step 2: Cross-check template placeholders vs generation phases**

For each `{{placeholder}}` in `skill-template.md`, verify which Phase produces it:

| Placeholder | Produced By |
|-------------|-------------|
| `{{character_slug}}` | Phase 0 (derived from name) |
| `{{character_name}}` | Phase 0 (user input) |
| `{{created_by}}` | Phase 0 (from session) |
| `{{generated_at}}` | Phase 6 (current date) |
| `{{short_identity}}` | Phase 1 |
| `{{role_play_rules_extended}}` | Phase 1 + Phase 2 |
| `{{attention_priorities}}` | Phase 2 |
| `{{attention_blindspots}}` | Phase 2 |
| `{{information_processing}}` | Phase 2 |
| `{{#each behavior_patterns}}` | Phase 5 |
| `{{mild_stress_behavior}}` | Phase 2 |
| `{{moderate_stress_behavior}}` | Phase 2 |
| `{{extreme_stress_behavior}}` | Phase 2 |
| `{{sentence_characteristics}}` | Phase 3 |
| `{{self_address}}` | Phase 3 |
| `{{address_superior}}` | Phase 3 |
| `{{address_equal}}` | Phase 3 |
| `{{address_intimate}}` | Phase 3 |
| `{{politeness_level}}` | Phase 3 |
| `{{verbal_tics}}` | Phase 3 |
| `{{anger_expression}}` | Phase 3 |
| `{{sadness_expression}}` | Phase 3 |
| `{{nervous_expression}}` | Phase 3 |
| `{{happy_expression}}` | Phase 3 |
| `{{fear_expression}}` | Phase 3 |
| `{{subtext_patterns}}` | Phase 3 |
| `{{dialogue_context_1..3}}` | Phase 3 |
| `{{goodwill_detection}}` | Phase 4 |
| `{{betrayal_detection}}` | Phase 4 |
| `{{closeness_gradient}}` | Phase 4 |
| `{{exploitation_detection}}` | Phase 4 |
| `{{stance_*}}` | Phase 4 |
| `{{value_priority_chain}}` | Phase 2 + Phase 5 |
| `{{absolute_bottom_lines}}` | Phase 2 + Phase 5 |
| `{{persuadable_areas}}` | Phase 2 + Phase 5 |
| `{{knowledge_in_character}}` | Phase 1 |
| `{{knowledge_gaps}}` | Phase 1 |
| `{{behavioral_boundaries}}` | Phase 2 |
| `{{honesty_boundary}}` | Phase 6 (from manifest) |

- [ ] **Step 3: Test with a sample input (dry-run mental trace)**

Mentally trace the flow with this sample input:

```
Input: "生成'凛夜'，她是一个失去故乡的流浪剑客少女，外表冷酷但内心温柔，在旅途中寻找灭族仇人，但逐渐发现真相并非她所想的那样。"
```

Trace through Phase 0 → Phase 6 and verify:
1. Phase 0 would correctly parse: name=凛夜, identity=流浪剑客, core_conflict=复仇vs真相, appearance=外冷内热, world=架空东方
2. Phase 1 would generate a setting with annotations
3. Phases 2-5 would have enough info from the description to generate meaningful content
4. Phase 6 would produce a complete `<character-slug>/` directory

- [ ] **Step 4: Final read-through for quality**

Read all created files and verify:
- No placeholders (TBD, TODO)
- Consistent naming (OCSP, character-slug, phase numbering)
- Chinese language used consistently throughout
- File paths are correct and consistent
```

---

### Task 8: Install and Smoke Test

**Files:**
- Verify: `.claude/skills/ocsp/SKILL.md` exists and is loadable

- [ ] **Step 1: Copy SKILL.md to Claude skills directory**

```powershell
$skillDir = "$env:USERPROFILE\.claude\skills\ocsp"
New-Item -ItemType Directory -Force -Path $skillDir | Out-Null
Copy-Item "c:\my_works\character-maker\SKILL.md" -Destination "$skillDir\SKILL.md" -Force
Copy-Item "c:\my_works\character-maker\skill-template.md" -Destination "$skillDir\skill-template.md" -Force
Copy-Item "c:\my_works\character-maker\references\" -Destination "$skillDir\references\" -Recurse -Force
Write-Output "OCSP skill installed to $skillDir"
```

Expected: Files copied, no errors.

- [ ] **Step 2: Verify skill appears in available skills**

Run: Ask Claude to list skills. OCSP should appear.

Alternative: Check that `.claude/skills/ocsp/SKILL.md` exists and has valid YAML frontmatter.

- [ ] **Step 3: Smoke test with a minimal character**

Invoke `/ocsp` and provide a minimal input:

```
生成"测试角色"，一个开朗的咖啡店店员
```

Verify that:
- Phase 0 parses and echoes the input
- Can proceed through Phase 1 with simple confirmation
- The co-creation pattern works

- [ ] **Step 4: Stop test at Phase 1 confirmation**

The smoke test only needs to verify the skill loads and Phase 0-1 work. Full generation would consume unnecessary tokens for testing purposes.
```

- [ ] **Step 2: Final check — spec coverage**

Self-review: for each spec section, verify a task covers it:
- Spec §1 (产品定位) → Task 1
- Spec §2 (执行流程) → Tasks 1-4
- Spec §3 (输出格式) → Tasks 5-6
- Spec §4 (SKILL.md 模板) → Task 5
- Spec §5 (更新流程) → Task 4
- Spec §6 (质量验证) → Task 4
- Spec §7 (技术实现) → Tasks 1-6 (all)
- Spec §8 (品味守则) → Task 4

No gaps found.

- [ ] **Step 3: Placeholder scan**

Scanned all tasks for: "TBD", "TODO", "implement later", "fill in details", "add appropriate X", "handle edge cases", "similar to Task N", bare descriptions without code.

No placeholders found. All steps contain concrete content.

- [ ] **Step 4: Type consistency check**

Key identifiers used across tasks:
- `character_slug` → consistent across SKILL.md, manifest-template.json, skill-template.md
- `source_tier` values: `user_input` / `co_created` / `ai_inferred` → consistent
- Phase numbering: 0-6, checkpoints at 1-5 → consistent
- Output path: `.claude/skills/<character-slug>/` → consistent

No inconsistencies found.
```

- [ ] **Step 2: Execute self-review** (embedded in the plan itself — completed above)

- [ ] **Step 3: Save and hand off**
