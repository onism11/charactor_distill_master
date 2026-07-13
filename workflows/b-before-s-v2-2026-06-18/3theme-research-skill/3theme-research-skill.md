---
name: theme-research
description: Generate theme.md for a virtual character by researching philosophical, literary, ethical, political, religious, psychological, or aesthetic concepts that deepen the character's dialogue. Use after character_rawcut.md, canon.md, and draft persona/action files exist. This skill defaults to web research unless the user forbids it, compares character evidence against external philosophy/wiki sources, and outputs a grounded theme layer without overwriting core persona.
---

# Theme Research Skill

## Purpose

Generate `theme.md`: a depth layer that helps a character speak with richer ideas, sharper debate frames, and more consistent worldview.

This skill does **not** rewrite `persona.md`. It adds conceptual support for deeper conversations.

`theme.md` has two valid output modes:

- `web-verified`: use external sources to ground philosophy / religion / politics / literature coordinates. This is the default when web access is allowed and the user wants a final thought-lineage layer.
- `local-only`: use only provided character materials. Use this when the user forbids web access, web access is unavailable, or the pipeline is intentionally doing a local pass. Mark the output as local-only / unverified, and do not present it as external intellectual-history validation.

---

## Inputs

Prefer these files:

- `character_rawcut.md`
- `canon.md`（已含内化的世界设定材料）
- `persona.md`
- `action.md`

Minimum viable input:

- target character name
- at least one evidence file containing dialogue, action, or canon

If persona/action are missing, still produce `theme.md`, but keep claims closer to trace/canon evidence.

---

## Research Policy

Choose the output mode before research.

Use `web-verified` by default for philosophy and concept grounding unless the user forbids web access or the run is explicitly local-only.

In `local-only` mode:

- skip web browsing and state that no external sources were used
- treat every framework as a character-evidence-derived interpretive coordinate
- do not claim the output is a philosophy, theology, political theory, or literary-history determination
- use `参考来源：local-only; based on ...` instead of external links

Preferred sources:

- Stanford Encyclopedia of Philosophy
- Internet Encyclopedia of Philosophy
- Britannica
- official or stable wiki pages for myth, religion, literature, psychology, politics, and aesthetics
- academic publisher pages, university pages, or primary text references when available

In `web-verified` mode, use broad reference pages and encyclopedias to discover possible coordinates, but verify and cite only the concepts that survive the Theme Gate and map back to character evidence.

Avoid relying on:

- unsourced fandom interpretation as philosophy evidence
- random SEO summaries
- social-media takes
- quote images

When using web sources:

- cite links in `theme.md`
- separate external concepts from character evidence
- never claim the character "is" a philosopher, doctrine, or school unless canon explicitly says so

---

## Workflow

### Step 0: Select Output Mode

Before extracting seeds, decide and record one mode:

```markdown
输出模式：web-verified / local-only
选择理由：{web allowed and external calibration needed / user forbids web / local pipeline pass / web unavailable}
```

If the mode is `local-only`, the final `theme.md` must say this near the top.

### Step 1: Extract Theme Seeds

From character materials, list recurring conceptual seeds:

```markdown
主题种子：
- 自由 / 秩序
- 救赎 / 控制
- 记忆 / 身份
- 牺牲 / 责任
- 命运 / 选择
```

Use only repeated or high-impact evidence. Do not research every noun.

### Step 2: Apply The Theme Gate

Before web research, filter theme seeds through this gate.

A seed can become a formal framework only if it passes all three checks:

```markdown
角色证据强：至少 2 条材料支持，或 1 条核心剧情/台词强支持。
能改善对话：能生成更好的辩论框架、分析角度、意象规则或拒绝边界。
不会覆盖 persona：不会把角色改写成哲学讲师、政治论文作者、治疗师或某个主义的代言人。
```

If a seed is useful but evidence is not strong enough, put it under `启发性联想` instead of the formal framework. If it is weak and does not improve dialogue, omit it.

Gate examples:

```markdown
通过：{概念 A}
理由：至少有两处关键材料支持同一问题张力；能让角色在相关话题中更准确地区分前提、代价和边界。

不通过：{概念 B}
理由：只有词汇或气质相似，没有足够文本证据；容易把角色误写成某个学派标签。
```

### Step 3: Choose Research Queries

For each seed, make practical queries:

```markdown
"{seed A} philosophy"
"{seed A} {seed B} ethics"
"{recurring concept} literary motif"
"{conflict pair} moral philosophy"
```

Also query Chinese terms if the character materials are Chinese:

```markdown
"{中文主题 A} {中文主题 B} 哲学"
"{中文母题} {冲突关键词} 文学"
```

Skip this step only in `local-only` mode. If skipped, explicitly mark the final output as local-only / unverified instead of silently omitting sources.

### Step 4: Map Concepts Back To Character Evidence

For each researched concept, answer:

- What problem does this concept name?
- Which character evidence actually supports this connection?
- What does this concept help the character say?
- What would be an overreach?

Do not add concepts that sound impressive but do not improve dialogue.
External philosophy/wiki material is only an explanatory coordinate. It must not turn into a character identity statement.

### Step 5: Generate theme.md

Write only `theme.md` unless the user asks for notes.

#### 颗粒度硬约束

- 正式框架 **≥ 4 个**（强证据，可独立支撑深度对话）
- 次级框架 **≥ 4 个**（证据偏弱，不应压过正式）
- 复杂角色材料足够时可以多写，不必卡死 4+4；但每新增一个框架都必须通过 Theme Gate，不能为了数量补标签。
- 启发性联想必须用**否定式格式**（"X：有 Y，但缺 Z"），**≥ 4 条**
- 每个正式 / 次级框架四层结构强制：命名 / 角色证据对应 / 能支撑的对话深度 / 使用边界 + 参考来源
- 若材料真不够 4+4+4，显式写"材料不足，已写 N 个 / 建议追加 M 个材料类型"，不允许悄悄省略

#### 模板

```markdown
# {角色名}：哲学谱系与问题域

这个文件不是默认运行人格。它只在用户明确要求哲学、政治哲学、宗教、文学母题、思想谱系或问题域比较时调用；普通闲聊、剧情问答和角色扮演优先使用 `persona.md`、`analysis.md` 与 `canon.md`。

模式说明：{web-verified：已使用外部来源校准概念坐标 / local-only：仅使用本地角色材料，未经外部思想史来源校验}

默认调用规则：
- 平时角色对话、日常闲聊、普通剧情问答，不调用本文件。
- 一般人物分析、价值评价、关系判断 → 默认用 `analysis.md`，不进入本文件。
- 只有当问题明确落在外部概念坐标层（哲学谱系、神学结构、政治哲学框架、文学母题、概念辨析）时，才在 analysis 之上叠加本文件。
- 即使问题涉及哲学 / 谱系，也应**先**用 `analysis.md` 给出角色立场，再把本文件作为外部坐标叠加；不要绕过 analysis 直接用 theme 出答案。
- 本文件中的内容大多属于解释性框架，不应伪装成原作文本直述。
- 若本文件与 `./reference/` 冲突，以 `./reference/` 为准；若与 `./analysis.md`、`./canon.md` 张力较大，以较贴近文本的写法为先。


## 1. 正式框架（≥ 4 个，四层结构强制）

### 1.1 {框架名}

- 命名：这个框架命名的问题是什么。
- 角色证据对应：哪些材料反复呈现这个张力（至少 2 处原文或具体事件）。
- 能支撑的对话深度：能让角色在哪类话题中更准确地区分前提 / 代价 / 边界。
- 使用边界：这个框架解释结构，不能把角色改写成学派代言人；输出口吻仍归 `persona.md` 管。
- 参考来源：{web-verified 模式写稳定来源链接或来源名，如 Stanford Encyclopedia / Britannica / 学者论文；local-only 模式写 `local-only; based on character_rawcut.md / canon.md / analysis.md`}

### 1.2 {框架名}

（同上四层结构）

### 1.3 {框架名}

（同上）

### 1.4 {框架名}

（同上）

（若材料能支撑更多正式框架，继续扩；若不到 4 个，显式写 "材料不足，已写 N 个"）

## 2. 次级框架（≥ 4 个，证据偏弱，不应压过正式）

### 2.1 {框架名}

- 命名：
- 角色证据对应：
- 适合解释什么：
- 不适合解释什么（边界）：
- 参考来源：{web-verified 外部来源 / local-only 本地证据文件}

### 2.2 - 2.N {至少 4 个，同上结构}

## 3. 启发性联想（≥ 4 条，否定式格式）

**只允许"否定式"列举**。格式：

- {外部概念 / 学派名}：{有 X 的相似性}，但 {缺 Y / 缺乏关键二元 / 角色证据不支持把它当身份定义}
  - 用法：{只能作为联想，不能作为角色定义}

至少 4 条。每条都要明确"为什么不够"——只有"为什么像"会让下游模型直接把角色塞进该学派。

参考好范例：

- 灵知主义：有"世界是被造的幻象"的痛感与突破冲动相似性，但缺核心二元论与解脱叙事；只能作为联想，不能作为角色定义。
- 大审判官式权威主义：有对"自由负担过重 / 多数人需要面包与奇迹"的同情相似性，但角色并非源自宗教权威本身，且仍保留对个体的牺牲意识；只能作为联想。

## 4. 更稳的问题框架

把外部概念翻译成更适合角色的问题。

格式：与其问 "{角色是不是某主义}"，不如问 "{角色如何处理 X 与 Y 的冲突}"。

写 ≥ 4 个开放问题，不要写成闭合答案。

## 5. 调用提醒

适合调用：
- 明确哲学 / 主题 / 谱系 / 概念辨析请求
- 用户要求对照外部思想史背景理解角色
- 用户在 analysis 已经给出立场后，要求进一步外推到思想谱系

不适合调用：
- 普通闲聊、关系互动、事实问答、低压 RP
- 材料不足但用户要求定论的请求
- analysis 已经能稳定回答的范围

提醒：
- theme 只能加深解释，不能替 persona 说话。
- 外部概念只能作为坐标，不能把角色改写成学派代言人。
- local-only theme 只能说明"本地材料可推出的问题坐标"，不能冒充外部思想史校验。
- 证据不足时回到 `character_rawcut.md`、`canon.md` 和 `analysis.md`。
```

---

## Quality Rules

Good theme entry（四层结构齐全）：

```markdown
### 1.1 {框架名}

- 命名：{外部概念} 可以作为 "{问题}" 的解释坐标。
- 角色证据对应：角色材料反复呈现 "{证据张力}"，至少 2 处可佐证（{原文 1}；{原文 2}）。
- 能支撑的对话深度：在 "{某类话题}" 中帮助角色更准确地区分 {前提 / 代价 / 边界}。
- 使用边界：这个框架只能解释冲突结构，不能把角色改写成学派代言人；输出口吻仍归 `persona.md` 管。
- 参考来源：{稳定来源链接或来源名}
```

Bad theme entry（标签化 / 缺四层）：

```markdown
### {名人 / 学派标签}
这个角色很像 {标签}，所以他说话要像 {标签刻板印象}。
```

Bad 启发性联想（只有"像"没有"缺"）：

```markdown
- 灵知主义：这个角色很有灵知主义的感觉。（缺 "但缺 Y" 部分，会让下游误用）
```

Why bad:

- no specific evidence
- no usable dialogue rule
- reduces character to a label

---

## Handoff

Send `theme.md` to later routing and audit skills together with:

- `persona.md`
- `action.md`
- `character_rawcut.md`
- `canon.md`

`theme.md` is optional depth support. If it conflicts with persona Layer 0 or canon, persona/canon win.

---

## Self-Check

- [ ] Did you research only themes supported by character evidence?
- [ ] Did you choose and state `web-verified` or `local-only` mode?
- [ ] If web-verified, did you browse by default unless the user forbade it?
- [ ] If local-only, did the output clearly mark itself as local-only / unverified?
- [ ] Did you cite external sources used, or local evidence files in local-only mode?
- [ ] If web-verified, did every promoted external coordinate get its own stable web source?
- [ ] Did you keep external philosophy separate from canon?
- [ ] Does `theme.md` read like an interpretive coordinate, not an audit table or philosophy lecture?
- [ ] 正式框架是否 ≥ 4 个，每个含四层结构（命名 / 角色证据对应 / 对话深度 / 使用边界 + 参考来源）？
- [ ] 次级框架是否 ≥ 4 个？
- [ ] 启发性联想是否 ≥ 4 条，且全部用"否定式"格式（"X：有 Y，但缺 Z"）？
- [ ] 更稳的问题框架是否 ≥ 4 个开放问题，而非闭合答案？
- [ ] 默认调用规则是否声明"先 analysis 后 theme"的叠加顺序？
- [ ] Did formal frameworks have strong character evidence?
- [ ] Did weaker but useful concepts stay under `启发性联想`?
- [ ] Did the opening and calling rules prevent theme from becoming default runtime?
- [ ] Did you avoid turning the character into a philosophy lecturer?
- [ ] Did you output `theme.md` only unless asked for extra notes?
- [ ] 字段不足时是否显式写"材料不足，已写 N 个"，而非悄悄省略？
