---
name: material-trace
description: Process raw character materials before distillation. Use when the user provides stories, dialogue, settings, scripts, voice lines, wiki text, links, or mixed notes and wants a raw evidence package for a target character. This skill first crawls or copies source text into `reference/`, then performs strict name-based raw cutting into `reference/character_rawcut.md`, and finally writes `canon.md` in ordered plot nodes without doing persona inference.
---

# Material Trace Skill

## Purpose

Turn messy raw materials into a clean evidence package for later character distillation and 把材料整理成适合角色 skill 使用的结构化资产。

This skill does **not** write the core persona. It only prepares:

- `reference/source_crawl.md`: crawled or copied raw source captures.
- `reference/character_rawcut.md`: scene-slice 级原料层，按确认人名命中，含 7 字段 metadata。
- `canon.md`: 中层事实整合，为角色 skill 提供高可用、快速检索的资料层。
- `au_overrides.md`：仅当用户提供 AU 设定时生成，可拔插，默认不加载（与 canon 同级）。
- （不生成独立 `world.md`：世界设定材料归并进 canon。）

Keep this cheap and useful. Do not overbuild classification systems, databases, source maps, or audit reports unless the user explicitly asks.

Hard workflow rule:

1. crawl or copy source text
2. cut by confirmed person names
3. summarize canon in node order

Do not reorder these three steps. Do not let canon writing happen before raw crawling and name cutting are done.

---

## Before Step 1: Ask For Target Names

Before processing, ask the user for:

```markdown
目标角色姓名：
已知别名 / 称呼：
需要排除的同名对象：
目标时期或版本（如果有）：
```

If the user already provided enough information, do not ask again. Proceed with the known names and aliases.

Confirmed names include:

- official name
- translated name
- nickname
- explicit name-like alias
- relationship name such as "哥哥", "家主", "殿下", only if user confirms it points to the target and wants it treated as a search name

Do not silently treat every title as an alias. Titles often refer to different people in different scenes.

---

## Step 1: Crawl / Source Capture

Before cutting by character name, first crawl or copy the raw material itself.

Supported source types:

- local `.md` / `.txt` / exported text
- wiki pages, profile pages, story pages, or user-provided URLs
- script-like material with scene labels
- mixed notes compiled by the user

Goal of this step:

- normalize the source into readable text
- record where the text came from
- preserve original wording
- keep source order
- do **not** write character conclusions yet

Output this step into `reference/source_crawl.md`.

Use this structure:

```markdown
# Source Crawl

目标角色：{name}
确认搜索名：{confirmed names}

## 1. 来源清单

- 来源 1：{file / url / material title}
- 来源 2：...

## 2. Raw Captures

### 来源 1：{title}
- 类型：{file / url / paste / wiki / script}
- 原文：
{按原顺序保留原文，必要时只做技术性清洗，不做总结改写}

### 来源 2：{title}
...
```

Rules:

- If the source is a web page or wiki page, fetch the page text before cutting by character name.
- If the source is already a local text file, copy or excerpt its raw text into the crawl output instead of immediately paraphrasing it.
- Technical cleanup is allowed only for OCR noise, duplicated whitespace, broken bullets, and obvious markup debris.
- Do not convert prose into your own summary.
- Do not drop chronology markers, speaker labels, or section titles if they help later node ordering.
- Do not enforce automatic fetching; if the runtime environment lacks WebFetch, users are required to first paste the webpage text into a local .md/.txt file before continuing.
---

## Step 2: Name-Based Raw Cut

After crawl is done, cut the material strictly by confirmed person names.

This step is the hard evidence basket for distillation. It must be **raw cut**, not second-stage rewriting.

Search keys allowed in the main cut:

- official name
- translated name
- user-confirmed explicit aliases that behave like names
- if the user gave multiple target names, run all of them

Do **not** use these to expand the main cut unless the user later asks for a broader second pass:

- titles only
- relationship labels only
- pronouns only
- inferred aliases from your own reading

For each hit, extract the smallest useful fragment that still preserves context:

- dialogue: the target line plus 1-3 nearby lines if needed
- narration: the paragraph containing the hit plus adjacent paragraph if the action spans both
- wiki/profile text: the subsection or bullet containing the hit
- script text: scene heading, speaker, and relevant lines

Output this step into `reference/character_rawcut.md`.

---

## Output 1: reference/character_rawcut.md

raw cut 是 trace 阶段的**原料层**。不分类、不归纳、不写人格结论。
单位是 **scene slice**：一个连续情节片段 = 一个条目。metadata 只写一次。

### 文件结构

```markdown
# Character Rawcut

## 0. 头部信息（写一次）

目标角色：{name}
确认搜索名：{aliases}
目标时期 / 版本：{period or 未指定}
排除同名：{exclusions or 无}

---

## 1. Scene Slices

按原文出现顺序列举。每个 slice 强制包含 7 个 metadata 字段 + 原文裁剪 + ≤ 1 句备注。

### S001：{场景标题}

- 来源：{file / wiki / 章节 / 页码}
- 命中词：{显式触发本 slice 的角色名 / 别名}
- 参与者：{本场景涉及的所有人物（含目标角色 + 其他角色）}
- 时期 / 状态：{主线 / 后日谈 / 战后 / 童年 / 不明}
- 是否疑似：{否 / 部分疑似（说明疑似的具体指代）}
- 场景类型：{对话 / 行动 / 旁白 / 书信 / 语音 / wiki 条目}
- 裁剪原因：{一句话说明为什么保留}

原文裁剪：
> {主体内容：保留原文，技术性清洗允许}
> {可保留 1-3 行上下文，让 distill 阶段能看懂触发条件}

简短备注：
- {最多一句，标 hard 信息缺口（"对手身份未明" / "时间点未定"），不写人格判断}

### S002：{场景标题}

（同上结构）

---

## 2. 可疑但重要的指代

独立段，与 § 1 平行。装的是"未确认人称 / 代词 / 称号"指向目标角色的片段，避免污染主 cut。

- {label}：{原文片段}
  - 上下文：{触发本指代的相邻片段}
  - 来源：{file / 章节}
  - 疑似原因：{为什么模糊}
  - 处置：{等待用户确认 / 不并入主 cut}
```

### 硬约束

- 同一连续场景**不得拆成多条**。一场景 = 一 slice，metadata 写一次。
- 原文必须是主体，备注 ≤ 1 句。
- 不写 "他表现出 X" / "她展示了 Y" 这类二次叙述。
- 不在 trace 阶段做人格判断，不写 "他很谨慎" / "她很温柔"。
- **不强行分类**到 "直接台词 / 关键行动 / 重大选择" 等栏目；分类是 distill 阶段的职责，不是 trace 的职责。
- 未确认指代必须放 § 2，**禁止**并入 § 1。
- metadata 7 字段顺序固定，便于下游 distill 按字段提取。

### 失败示例

```markdown
# 失败：把一段连续对话拆成 12 条独立 slice，metadata 重复 12 次
# 失败：把 "他在审问砂金时表现出极强的控制欲" 写进原文裁剪
# 失败：把 "那个戴白手套的男人" 直接当作目标角色写入 § 1
# 失败：在 trace 阶段就写 "Layer 0：他是一个秩序构想者"
# 失败：把同一场景的对话按 "台词 / 行动 / 关系" 拆成三条
```

### Good 示例

```markdown
### S007：审问砂金

- 来源：全台词.md / 主任务 2 双重赔偿
- 命中词：星期日
- 参与者：星期日、砂金、瓦尔特
- 时期 / 状态：主线
- 是否疑似：否
- 场景类型：对话
- 裁剪原因：体现高压审问 + "代价 / 风险" 主题词

原文裁剪：
> 星期日："我从不承担任何风险。基石必须由家族来保管。"
> 砂金（沉默）。
> 星期日："你以为我看不出你的犹豫吗？把它告诉我，是更体面的方式。"

简短备注：
- 该场景含两个高频概念词（风险 / 体面），可供 distill 提取语言风格。
```

---

## Step 3: canon.md

`canon.md` 是**经过二次压缩的人物材料层**，位于 `reference/` 与 `analysis.md` 之间：

- 比 `reference/` 更结构化、更适合调用
- 比 `analysis.md` 更贴近事实，不承担高层哲学展开

它服务于：

- 快速回答"原作里这个角色到底是什么人、做过什么、如何看待他人"
- 在不下钻 `reference/` 的前提下，提供可直接回填人物理解的中层材料

冲突处理：

- 若 canon 与 `reference/` 冲突，以 `reference/` 为准
- 若材料不足以定论，宁可写"待回查"，不补成定论
- 不在 canon 里展开高层哲学分析；那属于 `analysis.md`

世界设定材料归并约定：

- 世界 / 组织 / 制度 / 术语等设定材料**不另外生成 world.md**
- 影响角色判断的设定概念 → 进 `## 3. 核心设定概念`
- 设定术语影响角色措辞 → 进 `## 6. 口吻与表达依据`
- 仅作侧面背景的设定碎片 → 进 `## 8. 辅助理解材料`
- 跨世界比较 / AU 重写 → 用户驱动，单独写 `au_overrides.md`，不进 canon

颗粒度要求：canon 不是节标题清单，每一节都必须落到具体子结构。不要只把节标题写出来就交付。

---

### canon.md 结构总览

```markdown
# {角色名} Canon

{文件定位 3-5 行：中间层声明 + 与 reference / analysis 的关系 + 冲突处理}

## 使用原则

- 默认以 {目标时期} 的 {角色名} 为角色核心。
- {跨时期使用规则}
- 这里只整理：身份 / 关键记忆 / 核心设定概念 / 关键剧情节点 / 人物关系 / 口吻依据 / 易写偏点。
- 不在这里展开高层哲学分析；那属于 `analysis.md`。

## 阶段总览

### 阶段一：{阶段名}
- {阶段定位 1-3 句}
- 来源：
  - `{来源 1}`
  - `{来源 2}`

### 阶段二：{阶段名}
- ...

{以此类推，按剧情顺序写完所有主要阶段}

## 1. 基础身份认知
## 2. 背景故事与关键记忆
## 3. 核心设定概念
## 4. 关键剧情节点
## 5. 人物关系卡
## 6. 口吻与表达依据
## 7. 易写偏点
## 8. 辅助理解材料
## 9. 待进一步回查的问题
## 10. 调用建议
```

---

### ## 1. 基础身份认知 子结构

每一项都要按这三块写：

```markdown
### 可直接回填

- {角色作品归属、当前阶段定位}
- {官方资料确认的身份链}
- {不能被简化成什么} —— 写出常见误读
- {更准确的定位是什么} —— 用条目化展开

### 身份阶段差异

- `{阶段 A}`
  - {该阶段身份 1}
  - {该阶段身份 2}
- `{阶段 B}`
  - {该阶段身份 1}
  - {该阶段身份 2}

### 来源

- `{资料页 / 角色档案}`
- `{主任务 X / 章节名}`
- `{语音页 / 关于自己}`
```

---

### ## 2. 背景故事与关键记忆 子结构

按子主题切分，每个子主题再分"可直接回填"和"仅适合作为辅助理解"两层：

```markdown
### 2.1 {子主题名，例如 童年与某关系人}

#### 可直接回填

- {事实 1}
- {事实 2}
- {反差观察：当前阶段与某早期阶段的行为对照}

#### 仅适合作为辅助理解

- 结合：
  - `{来源 A}`
  - `{来源 B}`
- 可以推断：
  - {推论}
- 这是辅助理解，可以写进人物侧面，但不应写死成 {可能被误推成的硬结论}。

### 2.2 {子主题名，例如 某一关键职务或身份时期}
...

### 2.{N} 关键记忆清单

记忆清单要写成可下钻的事件，每条都展开成"事件经过 + 对角色的意义 + 来源"：

- `{记忆事件 1 简称}`
  - {事件具体经过，3-6 行，可分条}
  - 这件事对角色的冲击/意义：
    - {意义 1}
    - {意义 2}
  - 来源：`{来源 1}`、`{来源 2}`
- `{记忆事件 2 简称}`
  - ...
```

关键记忆清单是 canon 的高密度核心。每条记忆要写到下游 distill 直接拿来用而不必再回查 `reference/` 的程度。

---

### ## 3. 核心设定概念 子结构

每个概念用一个三级标题，配一段短说明（3-8 行）：

```markdown
### {概念 A，例如 角色追求的核心目标}

- 在 {角色名} 这里，"{概念 A}"不是 {常见浅层理解}，也不是 {另一种常见浅层理解}。
- 它首先意味着：
  - {内涵 1}
  - {内涵 2}
  - {内涵 3}

### {概念 B}

...

### 待回查

- {该角色思想体系里某个表达，原文层面的精确含义尚不确定，需后续回查}
```

每个核心概念都必须用角色自己的内涵去定义，不要用通用词典义。

---

### ## 4. 关键剧情节点 子结构

按剧情时间顺序写，不按主题重排。每个节点都要展开成"事件 + 重点不是 X 而是 Y + 来源"：

```markdown
### 4.1 {节点名}

- {事件一句话概括}
- 这里的重点不是 "{表面解读}"，而是：
  - {真正要传达的角色特征 / 行为模式 1}
  - {真正要传达的角色特征 / 行为模式 2}
- 来源：
  - `{来源章节}`

### 4.2 {节点名}

...
```

或者对结构性更强的节点用旧版字段格式：

```markdown
### 4.{N}：{节点名}
- 发生了什么：
- 角色做了什么：
- 直接后果：
- 可直接回填：
- 来源：
- 不要写成：
```

两种写法都允许，但同一份 canon 内不要混用。

---

### ## 5. 人物关系卡 子结构

每个重要关系按四件套写：

```markdown
### {关系对象 A}

- `核心情感`
  - {情感词 1}
  - {情感词 2}
  - {情感词 3}
- `主要矛盾`
  - {该关系的内部张力 1}
  - {该关系的内部张力 2}
- `不能写偏`
  - 不能只写成 "{常见误写 1}"
  - 不能只写成 "{常见误写 2}"
- `来源`
  - `{来源 1}`
  - `{来源 2}`

### {关系对象 B}

...
```

关系对象至少覆盖：亲密对象 / 旧师长或塑形者 / 对手或博弈对象 / 同盟或下属 / 弱者或求助者 这五类中实际存在的部分。

---

### ## 6. 口吻与表达依据 子结构

按时期 + 对象差异 + 直接依据三块：

```markdown
### {主要时期}

- {措辞特征：礼貌程度 / 句式长短 / 收束方式}
- 习惯：
  - {习惯 1，例如 先铺陈 再追问 最后宣告}
- 常见意象：
  - {意象 1}
  - {意象 2}
  - {意象 3}
- 准则感 / 情绪信号：{该角色说话常带什么样的判断词或节奏标记}

### {次要时期，例如 后期或战后}

- {相对主时期的变化}

### 对不同对象的差异

- 对 {对象 A}：{口吻调整}
- 对 {对象 B}：{口吻调整}
- 对 {对象 C}：{口吻调整}

### 直接依据

- `{语音页 / 关于 X}`
- `{资料页 / 着装、旅行、今后打算}`
- `{主线关键对话页}`
```

口吻依据是给 distill 阶段提取"原文口吻锚点"的原料层；不要写成性格分析，写成"角色说话长这样"的观察。

---

### ## 7. 易写偏点 子结构

写成一组"不要"，每条都贴具体误写场景：

```markdown
- 不要把 {角色名} 写成 {扁平化版本 1，例如 单一控制狂}。
- 不要把他/她写成 {扁平化版本 2，例如 空心的某种修辞机器}。
- 不要把他/她写成 {彻底放下某夙愿的后期版本}。
- 不要把他/她与 {关系对象 X} 的关系写成单一情绪块。
- 不要把 {某 NPC} 视角中的某句评价直接偷换成 "角色本质就是 X"。
- 不要忽略 {角色的某个关键内在驱力}；这是他/她 {走向某行为} 的起点，不是装饰。
- 不要因为某后期阶段他/她显得 {A}，就反向抹掉主时期的 {B}。
- 不要把 {某辅助形象} 直接等同于 "真实角色本性"。
```

至少 5 条，覆盖常见的扁平化、误推、阶段倒灌、附庸误读、单姿态过拟合。

---

### ## 8. 辅助理解材料 子结构

这一节装"不能当主骨架、但能补侧面"的材料。每条按 `可用结论` / `使用边界` / `来源` 三件套：

```markdown
### 8.1 {辅助材料来源，例如 某 NPC 视角下的本角色}

- `可用结论`
  - {可以从这一侧面安全推出的观察}
- `使用边界`
  - 这是 "辅助理解"，不是把角色定性成 {可能被误推的硬标签} 的证据。
  - 更准确的写法是：
    - {更稳妥的说法 1}
    - {更稳妥的说法 2}
- `来源`
  - `{来源 1}`
  - `{来源 2}`

### 8.2 {另一辅助材料}
...
```

辅助理解材料必须显式标注使用边界，不能让下游 distill 把它当硬证据。

---

### ## 9. 待进一步回查的问题

写成具体的回查任务，不要写空话：

```markdown
- {某事件 X} 中，{某细节 / 时间点 / 用词层次} 后续回填时建议再逐页核对。
- {某概念 Y} 在战斗文本与日常对话里的精确映射关系，需要回查原文行文。
- {某后期立场} 是否要纳入主线核心人物骨架，需按后续文件用途判断。
```

---

### ## 10. 调用建议

这一节告诉下游运行时何时优先查 canon、何时下钻、何时上推：

```markdown
优先查本文件的情况：
- 用户问 {角色名} 是谁、做了什么、为何走到这一步
- 用户问他/她与 {关键关系对象} 的关系
- 用户问 "{核心设定概念 A} / {概念 B} / {概念 C}" 这些专名或思想词
- 用户问 {主要时期} 与 {次要时期} 的差异

若本文件还不够，再下钻或上推：
- 具体原句、原文措辞、来源页码 → `./reference/character_rawcut.md` 或 `./reference/source_crawl.md`
- 高层思想、价值结构、哲学展开 → `./analysis.md`
- 涉及外部概念坐标（哲学谱系、神学结构、政治哲学、文学母题）→ `./analysis.md` 再叠加 `./theme.md`

冲突处理：
- canon 与 `./reference/` 冲突 → 以 `./reference/` 为准
- canon 与 `./analysis.md` 张力较大 → 较贴近文本的写法优先
```

`## 10. 调用建议` 是 canon 给运行时挂出的"下钻 / 上推"指针，是 auto-route SKILL.md 把 reference 与 world 收进 canon 单入口的必要配套。**不写这一段，canon 就是死的。**

---

### canon.md 通用规则

- 写事实，不写人格解释（人格解释归 distill 的 analysis.md）。
- canon 必须按剧情节点顺序，不要按主题重排事件。
- 不要把 canon 压缩成只有时间线。关键记忆、关系事实、对象差异、误写风险都必须保留。
- 优先使用 `可直接回填`、`辅助理解`、`来源`、`不要写成` 这类标签让事实更易复用。
- `不要写成` 只描述误读风险，不要变成人格分析。
- 显式标注不确定信息。
- 区分官方 canon 与用户 AU。
- 如果一条事实只来自未确认的可疑指代，放进 `## 9. 待进一步回查的问题`。

---

### 颗粒度示例

Good canon（事实 + 来源 + 角色行为锚点）：

```markdown
- 在 {阶段 X}，{角色名} 曾任 {具体职位}，参与 {具体事务}。
  - 这一阶段他对外 {具体行为模式}，首先建立的是 "{被外界感知到的形象}" 而不是 "{后期才显露的形象}"。
  - 来源：`{主任务 / 章节名}`
```

Bad canon（人格解释偷渡进事实层）：

```markdown
- 因为他深爱所有人，所以走向控制。
```

后者属于 distill 的 analysis.md，不属于 canon。

Bad canon（节标题清单，没有子结构）：

```markdown
## 5. 人物关系卡

### 知更鸟

（无核心情感 / 无主要矛盾 / 无不能写偏 / 无来源）
```

这是当前最常见的失败模式：节存在、子结构空。生成时**必须**展开所有子结构；展不开就写 "材料不足，待补"，不允许只留空标题。

---

## Output 3: 世界设定不另外生成 world.md

世界 / 组织 / 制度 / 术语等设定材料**不另外生成 world.md**。所有世界设定材料按对角色的影响归类合并进 canon.md：

- 影响角色判断的设定概念 → canon `## 3. 核心设定概念`
- 设定术语影响角色措辞 → canon `## 6. 口吻与表达依据`
- 仅作侧面背景的设定碎片 → canon `## 8. 辅助理解材料`

这是与 workdown benchmark 对齐的设计：workdown 没有独立 world.md，世界设定都内化进 canon。

例外：如果用户明确要做 **AU / 跨世界比较**，再手动单独写一份 world.md 挂载——此时由用户驱动，trace skill 不在默认流程里生成。

### AU 可拔插层（声明）

当用户提供 AU 设定（现代 / 校园 / 战后 / IF 线等）时，trace skill 生成 `au_overrides.md` 作为单独**可拔插**文件，**不进入** canon / analysis / persona / action 默认产物。

- 命名：`au_overrides.md`
- 位置：与 canon.md 同级
- 默认加载：否
- 加载时机：仅当用户在 auto-route 阶段明确挂载 AU 时，作为 overlay 生效
- 本轮 skill 不强制要求生成 `au_overrides.md`，仅声明命名与位置约定，避免本轮 scope 爆炸

如果用户没提 AU，**不生成本文件**。

---

## Handoff To Distillation

Send the next distillation skill:

- all of `reference/source_crawl.md`
- all of `reference/character_rawcut.md`
- all of `canon.md`
- confirmed alias list
- target period/version
- user exclusions
- `au_overrides.md`（仅当用户提供 AU 设定时才有此文件；distill skill 默认不读取，只在用户明确要求 AU 蒸馏时挂载）

The most important input for core persona distillation is `reference/character_rawcut.md`. Each scene slice 的 metadata 7 字段（来源 / 命中词 / 参与者 / 时期 / 是否疑似 / 场景类型 / 裁剪原因）是 distill 阶段按对象 / 场景 / 时期重组角色维度的抓手。

Do not send huge unrelated source dumps unless the next skill explicitly asks for raw verification.

---

## Minimal Workflow Summary

1. Ask for target name and confirmed search names.
2. Crawl or copy source text into `reference/source_crawl.md`.
3. Run strict name-based scene-slice cutting into `reference/character_rawcut.md`（scene slice 单位 + 7 字段 metadata，不分类）.
4. Write `canon.md` in ordered plot nodes（含世界设定材料的归并）。
5. 仅当用户提供 AU 设定时，单独生成 `au_overrides.md`；否则不生成。
6. Stop. 不生成独立 world.md。

---

## Self-Check

Before finishing:

- [ ] Did you ask for or infer the target name and confirmed aliases?
- [ ] Did Step 1 produce `reference/source_crawl.md` rather than a rewritten summary?
- [ ] Did Step 2 produce `reference/character_rawcut.md` 按 scene slice 而非单句切分？
- [ ] 每个 slice 是否都包含 7 个 metadata 字段（来源 / 命中词 / 参与者 / 时期 / 是否疑似 / 场景类型 / 裁剪原因）？
- [ ] 原文裁剪是否为主体，备注 ≤ 1 句？
- [ ] 同一连续场景是否避免被拆成多条重复条目？
- [ ] 是否避免了 trace 阶段做人格判断或二次叙述？
- [ ] 是否避免了强行分类到 "直接台词 / 关键行动 / 重大选择" 等栏目？
- [ ] 未确认指代是否全部放在 § 2，未污染 § 1？
- [ ] Does `canon.md` contain facts rather than personality conclusions?
- [ ] Does `canon.md` preserve ordered plot nodes rather than collapsing everything into a short overview?
- [ ] Does `canon.md` 开头声明自己是 reference 与 analysis 之间的中间层，并写明冲突处理？
- [ ] Does `canon.md` 包含 `使用原则` 与 `阶段总览` 两段，且阶段总览每条带来源？
- [ ] Does `## 1. 基础身份认知` 写出 `可直接回填` / `身份阶段差异` / `来源` 三块？
- [ ] Does `## 2. 背景故事与关键记忆` 按子主题切分，并包含 `关键记忆清单` 的事件级展开？
- [ ] Does `## 3. 核心设定概念` 给每个概念用角色自身视角写 3-8 行短段，不只列词条？
- [ ] Does `## 4. 关键剧情节点` 按剧情时间顺序展开，每节点都有 "重点不是 X 而是 Y" 或字段化结构？
- [ ] Does `## 5. 人物关系卡` 每个关系都写齐 `核心情感` / `主要矛盾` / `不能写偏` / `来源` 四件套？
- [ ] Does `## 6. 口吻与表达依据` 按时期 + 对象差异 + 直接依据三块写出？
- [ ] Does `## 7. 易写偏点` 至少 5 条，覆盖扁平化、阶段倒灌、附庸误读、单姿态过拟合？
- [ ] Does `## 8. 辅助理解材料` 每条都标 `可用结论` / `使用边界` / `来源`，不让下游误当硬证据？
- [ ] Does `## 9. 待进一步回查的问题` 写成具体回查任务而非空话？
- [ ] Does `## 10. 调用建议` 写明优先查本文件的情况 + 下钻 reference 的指针 + 上推 analysis 的指针 + 冲突处理？
- [ ] 是否消除了"只有节标题、子结构为空"的失败模式？空缺处是否写了 "材料不足，待补"？
- [ ] 是否**避免**生成独立 world.md？世界设定材料是否归并进 canon 的 ## 3 / ## 6 / ## 8？
- [ ] 仅当用户提供 AU 设定时才生成 `au_overrides.md`，没 AU 时是否未生成此文件？
- [ ] Did you avoid creating extra files beyond the requested upstream outputs unless asked?
- [ ] Did you avoid writing the final persona or theme analysis in this skill?
