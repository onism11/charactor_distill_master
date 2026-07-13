---
name: material-trace-step2-rawcut
description: Step 2 of material trace. Process source_archive/ in source_index order, turning every source unit into a character_slice / plot_spine / auxiliary_boundary entry, to produce source_archive/character_rawcut.md (raw scene slices + plot spine), Rawcut Evidence Brief, and rawcut_slices_index.json, without canon or persona inference.
---

# Material Trace Step 2 - Rawcut

## Purpose

按 source_index 顺序逐单元处理 archive，产出一个保留剧情脊柱顺序的原料证据篮，供 canon 与 distillation 使用。每个 source unit 都进入 handoff：命中目标角色的保留原文 scene slice，未命中的保留剧情梗概脊柱，可疑/边界材料单独隔离。This step does not write canon, personality analysis, or final persona.

Inputs:

- `source_archive/`
- `source_archive/source_index.md`
- checkpoint alias gate: target name, `confirmed`, `suspicious`, `excluded`, and target period/version

Output:

- `source_archive/character_rawcut.md`
- `check/Rawcut Evidence Brief.md`（always）
- `source_archive/rawcut_slices_index.json`（always；轻量机器索引，每条 entry 一行，便于 checker / S4 / S6 定位）

For very large archives, script-assist first: **按 source unit 建 scene-index，并同时产出两类候选**——hit candidates（命中目标名 / 高置信称谓的 scene 窗口，供 `character_slice`）与 no-hit plot_spine candidates（其余单元的 1-3 句梗概线索，供 `plot_spine`）。**不要只扫命中名**：每个 source unit 都要进入候选，确保无命中单元也有 plot_spine。模型只结构化候选，不必把全文读入上下文。

---

## Step 2: Scene-Index Raw Cut

按 `source_index` 的主网页 / 子任务 / 章节顺序**逐单元处理整个 archive**，保留剧情脊柱顺序，不只按确认名挑命中。**每个 source unit 都进入 handoff**，分为三类 `entry_type`：

- `character_slice`：单元命中目标角色（确认名，或 `alias_confidence: high` 的称谓）→ 保留原文 scene slice，进 `## 1`。
- `plot_spine`：单元无目标角色命中 → 保留 1-3 句剧情梗概，进 `## 3`，供下游理解因果连续性，不抄全文。
- `auxiliary_boundary`：可疑指代 / 时期外 / 资料页 / 他者视角强相关材料 → 进 `## 2`，仍可索引。

This step is the hard evidence basket. It must be **raw cut**, not second-stage rewriting.

### 称谓与 alias_confidence 分级

目标角色的不同称谓（头衔 / 关系名 / 身份名）与代词，**不再一律挡在主 cut 外**。判定分两条独立轴：**称谓 ⊥ 时期、客观事实 ⊥ 主观评价**。按"指代确定性"分三级：

- `high`：文件级归属明确 + 上下文/时期明确指向目标角色 + 客观陈述（"{头衔} 做了 X" 等价于 "{角色} 做了 X"）→ 进 `character_slice` 主 cut，metadata 标 `alias_confidence: high`。
- `mid`：指向目标但时期不明，或含透过他人视角的主观评价 → 进 `auxiliary_boundary`，标 `mid`；高价值时列入检查点交用户裁决。
- `low`：可能指向同场景的他人，或指代不明 → 进 `## 2` 可疑区，标 `low`，不进主 cut。

官方名 / 译名 / 用户在检查点 `confirmed` 的别名始终视为 `high`。判不准 high/mid 的**高价值**称谓**由 S1 / 主 agent 在 S1→S2 检查点 preflight 裁定**（基于 `source_index` / `source_manifest`，见 `0workflow.md`）；S2 只消费已裁决的 alias gate，**不自行扩张、不中途再问**。

Alias gate fields（检查点记录）：

- `scope`: global, source-limited, period-limited, or excluded.
- `anchor`: the first clear evidence that ties the name to the target.
- `collision_risk`: high/medium/low risk of hitting another person or ordinary wording.

For each hit, extract the smallest useful fragment that still preserves context:

- dialogue: the target line plus 1-3 nearby lines if needed
- narration: the paragraph containing the hit plus adjacent paragraph if the action spans both
- wiki/profile text: the subsection or bullet containing the hit
- script text: scene heading, speaker, and relevant lines

### 结构化来源规则

若 source unit 本身已是结构化爬取稿（自带剧情梗概 / 出场人物 / 情境标记等字段）：

- 命中判定只认单元正文与出场人物字段；模板说明、规则注释中的字面命中无效，不得据此保留单元。
- 不整 unit 复制：只摘录目标角色相关的情境块进"原文裁剪"；单元自带的梗概 / 设定字段直接复用为对应 metadata，不重写。
- 无目标命中的 unit 作为 `plot_spine` 进 `## 3 剧情脊柱索引`（单元 id + 1-3 句梗概），并进入 index/Brief，不只堆在文件尾。

Output this step into `source_archive/character_rawcut.md`. Legacy packages may place the same file at `reference/character_rawcut.md`, but the final package root should not keep rawcut as a main file.

### Large Rawcut Handoff

`check/Rawcut Evidence Brief.md`（human-readable）与 `source_archive/rawcut_slices_index.json`（machine）固定产出，作为下游导航层。

`rawcut_slices_index.json` 每条 entry 一行：slice/unit id, source unit, `entry_type` (character_slice | plot_spine | auxiliary_boundary), hit word, participants, period/status, line/range pointer, short reason, `alias_confidence` (high/mid/low；仅 character_slice 经称谓命中时记)。**三类 entry 都入索引**，便于 S4/S6 导航。

当 rawcut 体量大时（命中 > 500 / 文件 > 200 KB / slice > 150 / 单一 alias 覆盖 > 20% source units / rawcut > 20% source text），完整 `character_rawcut.md` 仅作证据仓，下游 S4/S6 **默认通过 index + Brief 导航定位，不全读**。

Do not pre-generate `targeted_probes/` by default; canon can request them when a specific causal gap appears.

### Output 2: check/Rawcut Evidence Brief.md

`Rawcut Evidence Brief` 是下游（S4 canon / S6 distill）的导航图，不是 checker，不写人格结论。必须包含：

- 总数：character_slice / plot_spine / auxiliary 各自数量。
- **高价值人物证据**：4-8 个高价值人物 slice 簇，每簇标 source unit / slice 范围 / 事件名或关键词；优先覆盖早期价值形成、核心关系、关键行动、价值冲突、失败或转折、默认时期声线；材料支持多少写多少。
- **Publication/full-complex 模式**：4-8 是普通材料的导航密度，不是上限。材料复杂、证据充足时，Evidence Brief 必须覆盖所有高价值人物证据簇；不得因为固定数字而丢低压互动、失败转折、关系变化、行动边界或关键声线证据。
- **剧情脊柱节点**：按顺序列出关键 plot_spine 节点（事件因果链），供 S4 canon 建制度 / 剧情因果；不必每条高价值，重在连续性。
- 缺口与使用边界，例如来源预筛、无显式确认名、时期归属不稳、结构化来源缺标题等。

禁止：写通过 / 不通过；写最小修改建议；替代 `canon.md`；写人格分析。

---

## Output 1: source_archive/character_rawcut.md

raw cut 是 trace 阶段的**原料层**。不分类、不归纳、不写人格结论。
单位是 **scene slice**：一个连续情节片段 = 一个条目。metadata 只写一次。

### 文件结构

```markdown
# Character Rawcut

## 0. 头部信息（写一次）

目标角色：{name}
确认搜索名：{aliases}
alias gate：{每个 confirmed alias 的 scope / anchor / collision_risk；suspicious/excluded 摘要}
目标时期 / 版本：{period or 未指定}
排除同名：{exclusions or 无}

---

## 1. Scene Slices

按原文出现顺序列举。每个 slice 强制包含 8 个 metadata 字段 + 原文裁剪 + ≤ 1 句备注；出现设定为可选第 9 项。

### S001：{场景标题}

- 来源：{file / wiki / 章节 / 页码}
- 命中词：{显式触发本 slice 的角色名 / 别名}
- 参与者：{本场景涉及的所有人物（含目标角色 + 其他角色）}
- 时期 / 状态：{主线 / 后日谈 / 战后 / 童年 / 不明}
- 是否疑似：{否 / 部分疑似（说明疑似的具体指代）}
- 场景类型：{对话 / 行动 / 旁白 / 书信 / 语音 / wiki 条目}
- 裁剪原因：{一句话说明为什么保留}
- 剧情梗概：{1-3 条要点，长单元可至 5 条；只写事件不写判断}
- 出现设定：{可选；仅当本场景出现影响理解的新概念时，每概念一行}

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
2. **重要辅助材料**：`mid`/`low` 称谓片段（时期不明、含主观评价、或可能指向他人）；与目标角色强相关的他者视角 / 辅助角色对话；后期 / 侧面 / 辅助阶段不宜污染主线 § 1 的材料；高频出现却无法判为 `high` 的实体。（判为 `high` 的同指代称谓走 § 1 `character_slice`，不进此区。）

每条写成 **3 字段卡**（rawcut 只搬运，不做判断）：

- {V001}：{主题}
  - 来源：{文件名 / 段落位置}
  - 触发词·归属：{他 / 哥哥 / 某称号 / 文件级归属 / 相关他者——说明为何不能并入 § 1}
  - 原文摘录：
    > {代表性片段，够 canon 判断即可——全文已在 `source_crawl`，不必重抄长段}

密度规则（防 § 2 变第二个 source_crawl）：每个来源文件只保留**最关键的 1–5 条**；若某类材料在 `source_crawl` 大量出现但无显式名字命中，**至少保留一条代表性卡**。代表性选择锚：优先选出现次数最多、或与目标角色互动最密集的段落。

---

## 3. 剧情脊柱索引（plot_spine）

无目标角色命中的 source unit 在此按顺序留 1-3 句剧情梗概（只写事件，不写判断），形成剧情因果脊柱。每条标 source unit id，并进入 Evidence Brief 的"剧情脊柱节点"块（不只堆在文件尾）。

- {U0001 单元标题}：{1-3 句事件梗概}
- {U0005 单元标题}：{1-3 句事件梗概}
```

Write the alias gate as literal resolved text, not shell or template expressions. A header such as `confirmed=$(...)` is invalid.

### 硬约束

- 同一连续场景**不得拆成多条**。一场景 = 一 slice，metadata 写一次。
- 原文必须是主体，备注 ≤ 1 句。
- 不写 "他表现出 X" / "她展示了 Y" 这类二次叙述。
- 不在 trace 阶段做人格判断，不写 "他很谨慎" / "她很温柔"。
- **不强行分类**到 "直接台词 / 关键行动 / 重大选择" 等栏目；分类是 distill 阶段的职责，不是 trace 的职责。
- `mid`/`low` 称谓、未确认指代、无显式名字的辅助材料放 § 2；只有确认名与 `alias_confidence: high` 的称谓可进 § 1。
- metadata 8 字段顺序固定（出现设定为可选第 9 项）；character_slice 经称谓命中时附 `alias_confidence`。剧情梗概只写事件，不写人格判断。
- **大源分批**：若 `source_archive/` 过大、单次放不下全量扫切，按来源文件或章节分批执行 rawcut——每批读入一个或多个 source unit，**每个 unit 都产出对应 entry**（命中→character_slice，未命中→plot_spine，可疑/边界→auxiliary），不只切命中名；各批完成后合并成完整 `source_archive/character_rawcut.md`（保留原文出现顺序、统一编号），§ 2 / § 3 同理分批后合并。

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
- 剧情梗概：星期日审问砂金，以基石保管权施压，要求其交代隐瞒的信息。

原文裁剪：
> 星期日："我从不承担任何风险。基石必须由家族来保管。"
> 砂金（沉默）。
> 星期日："你以为我看不出你的犹豫吗？把它告诉我，是更体面的方式。"

简短备注：
- 该场景含两个高频概念词（风险 / 体面），可供 distill 提取语言风格。
```

---
