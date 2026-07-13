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

目标角色：{name}（标注分析目标，不是 source_crawl 的过滤条件）
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
- If the source is already a local text file, **copy its full raw text** into the crawl output — no excerpting, no summarizing, no paraphrasing.
- Technical cleanup is allowed only for OCR noise, duplicated whitespace, broken bullets, and obvious markup debris.
- Do not convert prose into your own summary.
- Do not drop chronology markers, speaker labels, or section titles if they help later node ordering.
- Do not enforce automatic fetching; if the runtime environment lacks WebFetch, users are required to first paste the webpage text into a local .md/.txt file before continuing.
- **`source_crawl` 是全源证据仓，不以目标角色为筛选条件。** 连不含目标角色显式名字的段落也必须照录。禁止：按目标角色预筛 / 只摘看起来相关的片段 / 把长段改写成摘要 / 因目标时期是某期就删掉后期或辅助材料。是否"主干化"由 `character_rawcut` / `canon` 决定，**不在 `source_crawl` 阶段判断**。
- **大文件分批**：若单个来源文件过大、单次生成放不下全文，按文件分批照录（每批完整收录一个或多个文件，保留原文顺序），最终拼成完整 `source_crawl.md`。**禁止用"放不下"作为摘录式压缩的理由**——放不下就分批，不是截断。
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

## 2. 可疑指代 / 重要辅助材料隔离区

独立段，与 § 1 平行。**rawcut 只负责安全搬运原文，不判断可用方向 / 使用边界——那是 canon（`## 8 / ## 9`）的职责。**

本区收两类，均**禁止并入 § 1 主 cut**：

1. **可疑指代**：未确认人称 / 代词 / 称号指向目标角色的片段（原有范围）。
2. **重要辅助材料**（新增）：文件级归属明确但正文缺目标角色显式名字的关键片段；与目标角色强相关的他者视角 / 辅助角色对话；后期 / 侧面 / 辅助阶段不宜污染主线 § 1 的材料；高频出现却无显式名字命中的实体（如某角色在 `source_crawl` 大量出现但从未被目标角色名字切到）。

每条写成 **3 字段卡**（rawcut 只搬运，不做判断）：

- {V001}：{主题}
  - 来源：{文件名 / 段落位置}
  - 触发词·归属：{他 / 哥哥 / 某称号 / 文件级归属 / 相关他者——说明为何不能并入 § 1}
  - 原文摘录：
    > {代表性片段，够 canon 判断即可——全文已在 `source_crawl`，不必重抄长段}

密度规则（防 § 2 变第二个 source_crawl）：每个来源文件只保留**最关键的 1–5 条**；若某类材料在 `source_crawl` 大量出现但无显式名字命中，**至少保留一条代表性卡**。代表性选择锚：优先选出现次数最多、或与目标角色互动最密集的段落。
```

### 硬约束

- 同一连续场景**不得拆成多条**。一场景 = 一 slice，metadata 写一次。
- 原文必须是主体，备注 ≤ 1 句。
- 不写 "他表现出 X" / "她展示了 Y" 这类二次叙述。
- 不在 trace 阶段做人格判断，不写 "他很谨慎" / "她很温柔"。
- **不强行分类**到 "直接台词 / 关键行动 / 重大选择" 等栏目；分类是 distill 阶段的职责，不是 trace 的职责。
- 未确认指代与无显式名字的辅助材料必须放 § 2，**禁止**并入 § 1。
- metadata 7 字段顺序固定，便于下游 distill 按字段提取。
- **大源分批**：若 `source_crawl.md` 过大、单次放不下全量扫切，按来源文件分批执行 rawcut——每批读入 `source_crawl` 中一个或多个来源文件的 Raw Capture，按确认名切片输出 slices；各批完成后合并成完整 `character_rawcut.md`（保留原文出现顺序、统一 S 编号）。§ 2 可疑区同理分批收集后合并。

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
- 影响角色判断、被角色重新赋义、或已被主线绑到其权力 / 行动 / 终局链的设定概念 → 进 `## 3. 核心设定概念`（其中事件 / 机制 → `## 4`，头衔 → `## 0.2`）
- 设定术语影响角色措辞 → 进 `## 6. 口吻与表达依据`
- 未绑定核心链、仅作旁支背景的设定专名 / lore / 命名层，或侧面背景碎片 → 进 `## 8. 辅助理解材料`
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

## 0. 时间脊柱 / 阶段归位

## 1. 基础身份认知
## 2. 背景故事与关键记忆
## 3. 核心设定概念
## 4. 关键剧情节点
## 5. 人物关系卡
## 6. 口吻与表达依据
## 7. 易写偏点
## 8. 辅助理解材料
## 9. 待回查 / 未归位
## 10. 调用建议
```

---

### ## 0. 时间脊柱 / 阶段归位 子结构

`## 0` 是全篇 canon 的**时间坐标系**：先立一根可引用的人生时序骨架，再把身份/职务/关键记忆挂上去。它的产物是 T 编号，供下文各节（`## 1` / `## 2` / `## 4` 等）反向引用。

**条件触发（先读这条再决定怎么写）**：**只有材料支持分期时才建脊柱。** 生平简单、材料稀薄、单一阶段的角色，`## 0` 直接退化成"当前阶段 + 少量可确认前史 + 明确写'分期不足，缺乏依据'"，**禁止**硬切出 0.1 / 0.2 一堆空节点——结构完整但内容空心的时间轴比没有更糟。

**归位失败路由（一行规则，不另设栏目）**：任何无法定位、无法连线的材料 → 转 `## 9` 缺口区，**禁止在 `## 0` 内囤积**。`## 0` 不是材料池，不设 0.3 储存栏。

```markdown
## 0. 时间脊柱 / 阶段归位

（材料不支持分期时，本节只写：当前阶段 = {…}；可确认前史 = {…}；并注明"分期不足，缺乏依据"。以下 0.1 / 0.2 跳过。）

### 0.1 可确认阶段

只列能被材料支撑成"阶段"的时间节点，按时间顺序发 T 编号。每个阶段写一段**阶段画像**而不只是定位标签：这阶段他在做什么、处于什么位置或对外形象、由什么转变到此。**默认 2–4 句，材料足才写满；材料弱就一句，不为写满而注水。** `功能 / 位置或形象 / 关键转变` 等只是**写作提示，不是必答字段**——写成连贯几句即可，**不要逐项列成子标题或 `功能：… / 位置：…` 字段**。关键行动细节归 `## 2 / ## 4`，此处只点不复述。每条带来源。

- `T0`：{阶段名，例如 早期 / 某起点}
  - {阶段画像 2-4 句：在做什么 / 什么位置或形象 / 由什么转变到此}
  - 来源：`{来源}`
- `T1`：{阶段名}
  - {阶段画像 2-4 句}
  - 来源：`{来源}`
- `T2`：{当前核心身份期}
  - {阶段画像 2-4 句}
  - 来源：`{来源}`

（有几段写几段；材料只够一句的阶段就写一句画像 + 来源，不凑句数；不确定是否成段的，不写进 0.1，留给 0.2 做相对定位或转 ## 9。）

### 0.2 身份 / 职务定位记录

扫一遍材料里**角色第一人称自述、或叙述层明确写明**担任过的身份 / 职务 / 阶段性称号（"我曾作为 X" / "曾任 X" / "担任 X" / "身份为 X"），逐一落位。

窄触发——**只认**第一人称自述 + 叙述层明确写明。他人称呼、戏称、修辞、类比（"我就像一个 X"）**不进** 0.2；存疑的转 `## 9`。

每条两种写法择一：
- 能精确归位：`{身份/职务}` → 落在 `T?`（直接引用 0.1 的编号，**不重写**该阶段内容，防脊柱出现两份）。
- 只能相对定位：走**三件套**——
  - 推断了什么：`{身份/职务}` 介于 `T?` 与 `T?` 之间
  - 依据：`{哪句原文 / 哪条约束链，例如 年龄不可能、地位先后、叙述时点为"曾经"}`
  - 没有继续推断到哪：`{显式写明不推断什么，例如 不推断具体年份、不推断该期的具体事件，因无原文支撑}`

禁止：编造具体年份、编造该身份下的具体事件。
```

**失败示例（占位、零具体角色内容）**：

```markdown
# 失败：把类比"我就像一个 {X}"当成真担任过的职务，写进 0.2 并强行定位
# 失败：生平单一阶段的角色，被硬切成 T0/T1/T2/T3 + 一堆"材料不足"空节点
# 失败：把本该挂上时间轴的关键经历，直接丢进 ## 9 偷懒（## 9 只装真的归不了位的）
# 失败：0.2 里重写 0.1 已写过的阶段内容，导致脊柱出现两份
```

---

### ## 1. 基础身份认知 子结构

`## 1` 是**薄速查**，回答"这角色是谁、当前是什么身份"。它**引用 `## 0` 的 T 编号，不复述整段人生**——时序定位归 `## 0`，本节只做身份功能速查。

```markdown
### 可直接回填

- {角色作品归属、当前核心身份定位}
- {官方资料确认的身份链}
- {曾有过哪些重要身份} —— 每个标注对应 `## 0` 的哪个 T 编号
- {不能被简化成什么} —— 写出常见误读
```

---

### ## 2. 背景故事与关键记忆 子结构

按子主题切分，每个子主题再分"可直接回填"和"仅适合作为辅助理解"两层：

这些子主题是背景层，不替代 `2.{N} 关键记忆清单`。即使上面子主题已经写到关键事实，仍要在 `2.{N}` 再集中收束成可直接调用的记忆证据卡。缺这一节，视为本节未完成。

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

关键记忆清单是 canon 的**高密度核心**，要写成**可直接调用的证据卡**——保留足够的关键经过与关键原文，让下游 distill 在常见调用时**不必再回查 `reference/`**。

它必须作为 `2.{N}` 的**独立小节**存在，不能被压成 `K1-K4` 这种散卡，也不能被吸进 `## 4` 剧情节点里。材料明显支持多条关键记忆时，默认至少保留 3 条；宁可短卡并列，也不要把多条记忆压成一条总述。

**准入三条件，缺一不进正文**（不满足的转 `## 9`，不要塞进 `## 8 辅助理解材料` 装成果）：

- a. 能挂到 `## 0` 的某个 T 编号或相对阶段；
- b. 能连到一条关系 / 价值 / 思想转变 / 行动逻辑；
- c. 有来源。

每条按证据卡展开：

- `{记忆事件 1 简称}`
  - 挂点：`T?`（或相对阶段）
  - 事件经过：{把决定性的因果写完整，3-6 行，可分条；不要只留一句空摘要}
  - 关键原文 / 关键措辞：`{最有识别度、最值得下游直接复用的原文或措辞 1}`、`{原文/措辞 2}`
  - 连到：{关系变化 / 价值判断 / 思想转折 / 行动逻辑 中的一项，说清楚连到哪}
  - 来源：`{来源 1}`、`{来源 2}`
- `{记忆事件 2 简称}`
  - ...
```

**两条硬规则，平衡"更厚"与"不灌水"**：

- `关键原文 / 关键措辞` 用"原文"而非"原话"——来源可能是旁白 / 资料页 / 书信 / 战斗文本，不只是对白，别只找对白。它是为了保留**最容易在下游回答里被直接调用的文本钉子**——只留 1-2 句最有识别度的，**不是把整段 rawcut 大段搬进来**。
- 禁止只写一句空摘要而不留任何可复用原文；也禁止反过来把本节灌成 rawcut 副本。证据卡要"实"在关键因果与关键原文上，不在长度上。

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

`## 3` 先回答"这个概念对角色本人意味着什么"，再回答"它怎样改写了角色的关系、判断或终局选择"。如果写完更像百科条目，而不是人物定义，就不该占这一节的主位。

可留在 `## 3` 的，只有两类：一类是角色反复拿来解释自己、他人或世界的词；另一类是虽偏机制名、但已被主线直接绑到其身份、行动或终局的概念。其余设定名词不要为凑完整而排队上墙。

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

`核心情感` / `主要矛盾` / `不能写偏` / `来源` 四个字段名固定，不得改写成近义标签。关系定位、互动事实、对角色的约束可以写进这四件套的内容里，但不能替换四件套本身。

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

`## 6` 不是三个标题占位。每个时期都要能还原句法、意象、节奏或对象差异，至少写到下游可以直接借用，不要缩成几个抽象词。若材料明确存在主线后或战斗期的口吻差异，就保留第二时期；若无，再退化。

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

`## 8` 只装**人物侧面卡**：真实存在、能补人物独特侧面，但不默认抬升为主骨架。默认按**侧面主题**组织，不按文件名、来源名或设定名词分桶。

它不是 raw 文件索引，也不是世界观余料仓。凡是能改写角色身份、关键记忆、主线剧情、核心关系或核心概念的内容，都应回主干；凡是只剩设定专名、却解释不了人物的，不要硬塞这里。

若某来源可疑，就在 `使用边界` 里说清，或转 `## 9`；不要把 `## 8` 改造成“来源隔离区”。

```markdown
### 8.1 {侧面主题名，例如 某他者视角中的某种侧面}

- `可用侧面`
  - {这条材料能安全补出的人物侧面}
  - 适用范围：只代表 {阶段 / 视角 / 情境}，不能压过 `## 0` / `## 2` / `## 4` / `## 5` 的主骨架。
- `使用边界`
  - 这是 "辅助理解"，不是把角色定性成 {可能被误推的硬标签} 的证据。
  - 更稳妥的写法是：
    - {更稳妥的说法 1}
    - {更稳妥的说法 2}
- `来源`
  - `{来源 1}`
  - `{来源 2}`

### 8.2 {另一侧面主题名}

...
```

辅助理解材料必须显式标注使用边界，不能让下游 distill 把它当硬证据。`可用侧面` / `使用边界` / `来源` 三个字段名固定，不得改写。

---

### ## 9. 待回查 / 未归位（统一缺口区）

`## 9` 是全篇唯一的**缺口区**，不是成果区。**封顶 ≤ 5 条**——超过 5 条说明归位阶段没充分推理，回去再想，而不是先放弃再攒列表。

装两类，每条都要写明"为什么还在这里"：

- **待回查**：需要回原文逐页核对的具体事实 / 矛盾 / 用词层次。
- **未归位**：出现过、可能重要，但暂时无法挂上 `## 0` 时间轴、或无法连到关系 / 价值 / 思想转变的材料。

```markdown
- [待回查] {某事件 X} 中，{某细节 / 时间点 / 用词层次} 后续回填时建议再逐页核对。 —— 为什么还在这里：{原文未给出明确依据}
- [待回查] {某概念 Y} 在战斗文本与日常对话里的精确映射关系，需要回查原文行文。 —— 为什么还在这里：{两处用法是否同义未定}
- [未归位] {某材料 Z} 出现过但无法定位到具体阶段。 —— 为什么还在这里：{只有一处提及，无时间锚点，且无法连到任何关系/价值/思想转变}
```

**禁止**把本该归位的关键经历丢进这里偷懒；这里只装**真的**归不了位的。

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
- **重要性不按来源文件分级。** 同一份原材料里既可能有主骨架事实，也可能有侧面材料；一条信息进 `## 0` / `## 2` / `## 4` / `## 5` 还是 `## 8`，取决于它在人物理解中的**功能**，而不是它来自哪份文件。任何文件里只要有会改写主骨架的内容就进主干，不得因"来自次要文件"被降级塞进 `## 8`。
- **子结构字段名视为契约。** `2.{N} 关键记忆清单` 不能消失；`## 5` 必须是 `核心情感 / 主要矛盾 / 不能写偏 / 来源`；`## 8` 必须是 `可用侧面 / 使用边界 / 来源`。近义改写视为结构失败。
- **canon 承担文本内推断，不做文本外推断。** 这是 canon 比 rawcut 多、比 analysis 少的那一层职责。
  - 文本内推断（canon 的职责）：用约束传播定位时点——显式时间标记 → 身份 / 地位的先后与前提（年龄、资历）→ 叙述里的"曾经 vs 现在" → 关联可考的思想演变。例如"`{职务}` 应介于 `{T?}` 与 `{T?}` 之间"。
  - 文本外推断（越界，禁止）：靠叙事常识脑补原文没有的东西，例如"`{职务}` 期他可能失去过信仰"。
  - 所有推断走三件套：推断了什么 / 依据哪句原文或哪条约束链 / 没有继续推断到哪。第三项逼模型显式声明它**没**做的那步推断，防止"既然都推到这了"继续脑补。
- canon 必须按剧情节点顺序，不要按主题重排事件。
- 不要把 canon 压缩成只有时间线。关键记忆、关系事实、对象差异、误写风险都必须保留。
- 若材料明确支持后续阶段、战后回望、回乡 / 启程 / 自我整合这类延续节点，优先保留在 `## 0` 或 `## 4`，不要一律降进 `## 8`。
- 优先使用 `可直接回填`、`辅助理解`、`来源`、`不要写成` 这类标签让事实更易复用。
- `不要写成` 只描述误读风险，不要变成人格分析。
- 显式标注不确定信息。
- 区分官方 canon 与用户 AU。
- 如果一条事实只来自未确认的可疑指代，放进 `## 9. 待回查 / 未归位`。

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

- 影响角色判断、被角色重新赋义、或已被主线绑到其权力 / 行动 / 终局链的设定概念 → canon `## 3. 核心设定概念`（其中事件 / 机制 → `## 4`，头衔 → `## 0.2`）
- 设定术语影响角色措辞 → canon `## 6. 口吻与表达依据`
- 未绑定核心链、仅作旁支背景的设定专名 / lore / 命名层，或侧面背景碎片 → canon `## 8. 辅助理解材料`

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

Hard fail if any of these happen:

- `2.{N} 关键记忆清单` 缺失，或被压成散卡 / 剧情节点附属条。
- `## 5` 没有完整展开 `核心情感 / 主要矛盾 / 不能写偏 / 来源` 四件套。
- `## 8` 没有完整展开 `可用侧面 / 使用边界 / 来源` 三件套，或退化成文件分桶。
- 材料明确支持后续阶段 / 战后回望，但 canon 只在 `## 8` 轻描淡写，不进入 `## 0` 或 `## 4`。
- `## 3` 主要变成世界观条目表，而不是角色视角下的概念定义。

- [ ] Did you ask for or infer the target name and confirmed aliases?
- [ ] Did Step 1 produce `reference/source_crawl.md` rather than a rewritten summary?
- [ ] `source_crawl.md` 对本地文本输入是否**完整照录**、未按目标角色预筛或摘要（抽查：字节量与输入是否同量级）？
- [ ] Did Step 2 produce `reference/character_rawcut.md` 按 scene slice 而非单句切分？
- [ ] 每个 slice 是否都包含 7 个 metadata 字段（来源 / 命中词 / 参与者 / 时期 / 是否疑似 / 场景类型 / 裁剪原因）？
- [ ] 原文裁剪是否为主体，备注 ≤ 1 句？
- [ ] 同一连续场景是否避免被拆成多条重复条目？
- [ ] 是否避免了 trace 阶段做人格判断或二次叙述？
- [ ] 是否避免了强行分类到 "直接台词 / 关键行动 / 重大选择" 等栏目？
- [ ] 未确认指代与无显式名字辅助材料是否全部放在 § 2（3 字段卡：来源 / 触发词·归属 / 原文摘录），未污染 § 1？
- [ ] Does `canon.md` contain facts rather than personality conclusions?
- [ ] Does `canon.md` preserve ordered plot nodes rather than collapsing everything into a short overview?
- [ ] Does `canon.md` 开头声明自己是 reference 与 analysis 之间的中间层，并写明冲突处理？
- [ ] `## 0` 是否在材料支持时写出阶段画像（2–4 句、未逐项列字段）、不足时退化（未凑空节点 / 凑句数）？`## 0.2` 是否落全自述身份（窄触发 + 三件套）？
- [ ] `## 1` 是否压成基础身份速查、未重复阶段差异 / 来源、重要身份挂 `## 0` T 号？
- [ ] `## 2` 是否保留了独立的 `2.{N} 关键记忆清单`，并在材料明显支持多条时至少写出 3 条证据卡，而不是散进 `2.1-2.4` 或 `## 4`？
- [ ] `## 3` 是否优先用角色视角定义概念，而不是让机制名 / 世界观名词喧宾夺主？
- [ ] `## 5` 是否完整展开关系四件套，且字段名没有被近义替换？
- [ ] `## 6` 是否至少保住一个可直接复用的主时期口吻块；材料支持时，是否保住第二时期差异？
- [ ] `## 8` 是否写成按侧面主题组织的人物侧面卡，且字段名固定为 `可用侧面 / 使用边界 / 来源`？
- [ ] canon 的推断是否只做文本内推断、每条走三件套？`## 2` 关键记忆是否过准入三条件且写成证据卡？
- [ ] `## 3`–`## 7` 每节是否有实质子结构（概念 3-8 行 / 剧情按时序 / 关系四件套 / 口吻三块 / 易写偏 ≥5）？
- [ ] 若材料明确支持后续阶段 / 战后回望，这部分是否进入 `## 0` 或 `## 4`，而不是只落在 `## 8`？
- [ ] Does `## 9. 待回查 / 未归位` 封顶 ≤ 5 条，作为缺口区（非成果区）容纳"待回查"与"未归位"两类，每条写明为何还在这里？
- [ ] Does `## 10. 调用建议` 写明优先查本文件的情况 + 下钻 reference 的指针 + 上推 analysis 的指针 + 冲突处理？
- [ ] 是否消除了"只有节标题、子结构为空"的失败模式？空缺处是否写了 "材料不足，待补"？
- [ ] 是否**避免**生成独立 world.md？世界设定材料是否归并进 canon 的 ## 3 / ## 6 / ## 8？
- [ ] 仅当用户提供 AU 设定时才生成 `au_overrides.md`，没 AU 时是否未生成此文件？
- [ ] Did you avoid creating extra files beyond the requested upstream outputs unless asked?
- [ ] Did you avoid writing the final persona or theme analysis in this skill?
