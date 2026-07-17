---
name: material-trace-step2-rawcut
description: Step 2 of material trace. Use source_archive/ and source_archive/source_index.md plus confirmed target names to produce source_archive/character_rawcut.md as raw scene slices, without canon or persona inference.
---

# Material Trace Step 2 - Rawcut

## Purpose

Cut the archived source materials by confirmed target names into a raw evidence basket for later canon and distillation. This step does not write canon, personality analysis, or final persona.

Inputs:

- `source_archive/`
- `source_archive/source_index.md`
- checkpoint alias gate: target name, `confirmed`, `suspicious`, `excluded`, and target period/version

Output:

- `source_archive/character_rawcut.md`
- if rawcut is large: `source_archive/rawcut_slices_index.json` plus `check/Rawcut Evidence Brief.md` or `check/Rawcut Digest.md`

For very large archives, use script-assisted candidate extraction first: scan confirmed names/aliases, collect surrounding scene windows or chapter sections, then let the model structure only those candidate slices. Do not read the full archive into model context unless necessary.

---

## Step 2: Name-Based Raw Cut

After crawl is done, cut the material strictly by confirmed person names.

This step is the hard evidence basket for distillation. It must be **raw cut**, not second-stage rewriting.

Search keys allowed in the main cut:

- official name
- translated name
- checkpoint `confirmed` aliases that behave like names
- if the user gave multiple target names, run all of them

Do **not** use these to expand the main cut unless the user later asks for a broader second pass:

- titles only
- relationship labels only
- pronouns only
- inferred aliases from your own reading

Alias gate fields:

- `scope`: global, source-limited, period-limited, or excluded.
- `anchor`: the first clear evidence that ties the name to the target.
- `collision_risk`: high/medium/low risk of hitting another person or ordinary wording.

Only `confirmed` enters `## 1` main cut. `suspicious` may only enter `## 2` or later targeted probes. `excluded` is not searched except for collision checks.

For each hit, extract the smallest useful fragment that still preserves context:

- dialogue: the target line plus 1-3 nearby lines if needed
- narration: the paragraph containing the hit plus adjacent paragraph if the action spans both
- wiki/profile text: the subsection or bullet containing the hit
- script text: scene heading, speaker, and relevant lines

### 结构化来源规则

若 source unit 本身已是结构化爬取稿（自带剧情梗概 / 出场人物 / 情境标记等字段）：

- 命中判定只认单元正文与出场人物字段；模板说明、规则注释中的字面命中无效，不得据此保留单元。
- 不整 unit 复制：只摘录目标角色相关的情境块进"原文裁剪"；单元自带的梗概 / 设定字段直接复用为对应 metadata，不重写。
- 无目标内容的 unit 不进 § 1；在 `## 3. 未命中单元梗概索引` 留一行（单元 id + 1 句梗概），供 canon 理解剧情连续性。

Output this step into `source_archive/character_rawcut.md`. Legacy packages may place the same file at `reference/character_rawcut.md`, but the final package root should not keep rawcut as a main file.

### Large Rawcut Handoff

If any trigger is met, keep the full `source_archive/character_rawcut.md` as evidence storage and also write sidecar files for later sessions:

- confirmed-name hits > 500
- `source_archive/character_rawcut.md` > 200 KB
- scene slices > 150
- one alias covers more than 20% of source units
- rawcut size exceeds 20% of source text

Required sidecars:

- `source_archive/rawcut_slices_index.json`: slice id, source unit, hit word, participants, period/status, line/range pointer, short reason.
- `check/Rawcut Evidence Brief.md` or `check/Rawcut Digest.md`: human-readable guide to periods, high-value scenes, relationship clusters, and gaps.

Do not pre-generate `targeted_probes/` by default; canon can request them when a specific causal gap appears.

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
2. **重要辅助材料**（新增）：文件级归属明确但正文缺目标角色显式名字的关键片段；与目标角色强相关的他者视角 / 辅助角色对话；后期 / 侧面 / 辅助阶段不宜污染主线 § 1 的材料；高频出现却无显式名字命中的实体（如某角色在 `source_crawl` 大量出现但从未被目标角色名字切到）。

每条写成 **3 字段卡**（rawcut 只搬运，不做判断）：

- {V001}：{主题}
  - 来源：{文件名 / 段落位置}
  - 触发词·归属：{他 / 哥哥 / 某称号 / 文件级归属 / 相关他者——说明为何不能并入 § 1}
  - 原文摘录：
    > {代表性片段，够 canon 判断即可——全文已在 `source_crawl`，不必重抄长段}

密度规则（防 § 2 变第二个 source_crawl）：每个来源文件只保留**最关键的 1–5 条**；若某类材料在 `source_crawl` 大量出现但无显式名字命中，**至少保留一条代表性卡**。代表性选择锚：优先选出现次数最多、或与目标角色互动最密集的段落。

---

## 3. 未命中单元梗概索引（仅结构化来源时）

- {U0001 单元标题}：{1 句事件梗概}
- {U0005 单元标题}：{1 句事件梗概}
```

Write the alias gate as literal resolved text, not shell or template expressions. A header such as `confirmed=$(...)` is invalid.

### 硬约束

- 同一连续场景**不得拆成多条**。一场景 = 一 slice，metadata 写一次。
- 原文必须是主体，备注 ≤ 1 句。
- 不写 "他表现出 X" / "她展示了 Y" 这类二次叙述。
- 不在 trace 阶段做人格判断，不写 "他很谨慎" / "她很温柔"。
- **不强行分类**到 "直接台词 / 关键行动 / 重大选择" 等栏目；分类是 distill 阶段的职责，不是 trace 的职责。
- 未确认指代与无显式名字的辅助材料必须放 § 2，**禁止**并入 § 1。
- metadata 8 字段顺序固定（出现设定为可选第 9 项），便于下游 distill 按字段提取。剧情梗概只写事件，不写人格判断。
- **大源分批**：若 `source_archive/` 过大、单次放不下全量扫切，按来源文件或章节分批执行 rawcut——每批读入 archive 中一个或多个来源单元，按确认名切片输出 slices；各批完成后合并成完整 `source_archive/character_rawcut.md`（保留原文出现顺序、统一 S 编号）。§ 2 可疑区同理分批收集后合并。

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
