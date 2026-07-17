---
name: persona-skill-distiller
description: Distill character source material into an auditable routed persona package through isolated source crawl, rawcut, canon, distill, optional theme, route, and checker stages. Use when building, rerunning, auditing, or publishing a virtual-character skill package from source files.
---

# 角色蒸馏多 session 工作流规范

本文件是 Persona Skill Distiller 的唯一公开 skill 入口。它只规定阶段、输入、输出、验收与返修；具体生成模板按当前阶段到对应 `workflow-stage-*` 文件读取。阶段目录是内部提示资源，不是可单独调用的 skill。

Material Trace 已拆成三份 step 文件；原材料阶段以当前 `workflow-stage-a-source-canon/step1-source-crawl.md`、`step2-rawcut.md`、`step3-canon.md` 为准。

---

## 0. 总览

一句话定位：

> Persona Skill Distiller：面向虚拟角色的多阶段蒸馏与评测工作流。

流程：

```text
Session 1: Source Crawl
  -> 人工检查点
  -> Session 2: Rawcut
  -> Session 3: Rawcut Checker
  -> Session 4: Canon
  -> Session 5: Canon Checker
  -> Session 6: Distill
  -> Session 7: Distill Checker
  -> (可选) Session 8: Theme Research
  -> Session 9: Route
  -> (可选) Session 10: Skill Checker
```

核心边界：

- 成品是角色 `SKILL.md`；生产线是 artifact-based workflow。
- 阶段之间通过文件交接，不继承上一阶段聊天上下文。
- Create 只生成；Checker 只验收；用户裁决是否返修或进入下一阶段。

### 唯一入口与按需加载

- 只通过本 `SKILL.md` 启动蒸馏。
- 启动一个执行角色时，只读 `workflow-prompts/` 中与该角色对应的一份启动提示，再读当前 session 的一份阶段文件与本 session 明列的产物；不得预载其他阶段或全部提示。
- `workflow-prompts/` 只属于生产控制面，不复制进最终角色包，也不属于角色运行时上下文。
- F/G 只在用户明确要求答题测试或横评时启动；S1-S10 生产链不读 F/G prompt。
- active 中不得新增 `answer-agent-prompt.md`。答题隔离边界以按需传入的 `workflow-prompts/Agent-F-answer-test.md` 为准。

控制器提示映射：

| 执行角色 | 对应阶段 | 按需启动提示 |
|---|---|---|
| A | S1-S5 中 controller 指定的当前单一 session | `workflow-prompts/Agent-A-prep-canon.md` |
| B | S6 Distill | `workflow-prompts/Agent-B-distill.md` |
| C | S7 Distill Checker | `workflow-prompts/Agent-C-distill-checker.md` |
| D | 可选 S8 Theme Research | `workflow-prompts/Agent-D-theme.md` |
| E | S9 Route 或可选 S10 Skill Checker；两者保持文件交接隔离 | `workflow-prompts/Agent-E-route-checker.md` |
| F | 可选 Answer Test | `workflow-prompts/Agent-F-answer-test.md` |
| G | 可选 Answer Review / Benchmark Comparison | `workflow-prompts/Agent-G-answer-review.md` |

---

## 1. 产物契约

| 产物 | 来源 session | 必须包含 | 格式参考 |
|---|---|---|---|
| `source_archive/` | S1 | 全量来源、source units、编码/章节策略、体量/覆盖索引；含 `source_index.md`、`source_manifest.json`、`run_manifest.json` | `workflow-stage-a-source-canon/step1-source-crawl.md`；旧版等价产物为 `reference/source_crawl.md` |
| `source_archive/character_rawcut.md` + 大 rawcut sidecar | S2 | `## 0` 头部；`## 1 Scene Slices` 的 7 字段 metadata；`## 2 可疑/辅助区` 的 3 字段卡；必要时附索引/摘要 | `workflow-stage-a-source-canon/step2-rawcut.md` |
| `canon.md` | S4 | `## 0`-`## 10` 全结构；内化世界设定；不生成独立 `world.md` | `workflow-stage-a-source-canon/step3-canon.md` |
| `analysis.md` / `persona.md` / `action.md` | S6 | 各自必填子结构；时期声明；可运行人格；行动边界 | `workflow-stage-b-distill/workflow-stage-b-distill.md` |
| `theme.md` | 可选 S8 | 主题研究产物；区分角色证据与外部概念；不得覆盖 persona/canon | `workflow-stage-d-theme/workflow-stage-d-theme.md` |
| `SKILL.md` | S9 | 路由规则、调用建议、memory/command 策略；persona always-on, action/theme/canon conditional | `workflow-stage-e-route/workflow-stage-e-route.md` |
| `check/{Stage} Check.md` | S3/S5/S7/S10 | 通过/不通过、薄弱点、最小修改建议；不改写正文 | `workflow-stage-fg-audit/workflow-stage-fg-audit.md` |

最终角色包主目录默认只保留 8 个运行主文件：

```text
canon.md
analysis.md
persona.md
action.md
theme.md
SKILL.md
memory.md
long_memory.md
```

证据仓与机器索引放入 `source_archive/`；所有 checker、brief、digest、smoke 或人工验收报告放入 `check/`。

---

## 2. Session Spec

### Session 1: Source Crawl

- 读：用户提供的原材料文件（本地 `.md` / `.txt` / URL / 粘贴材料等）。
- 指令来源：`workflow-stage-a-source-canon/step1-source-crawl.md`。
- 输出：`source_archive/` 全量存档 + `source_archive/source_manifest.json` + `source_archive/source_index.md` + `source_archive/run_manifest.json`；旧版运行可输出 `reference/source_crawl.md`，但必须能映射为 archive + index。
- 通过标准：原文完整保存、来源可回查、来源顺序保留、不按目标角色预筛、不把长段改写成摘要。
- 分批规则：大文件按文件或章节分批归档，最终合并为完整 archive；禁止用上下文放不下作为截断或摘录理由。

### 主 agent / 用户检查点

- 检查：`source_archive/source_index.md` 是否覆盖全部材料，来源列表是否完整，体量是否与输入同量级，关键来源是否能回查到原文。
- 同时裁决三件事：编码是否可信；章节 / source unit 是否可信；alias gate 是否明确为 `confirmed / suspicious / excluded`，并记录 `scope / anchor / collision_risk`。
- 读取 `source_index.md` 的“大源”标记；只有“大源：是”才允许 Session 2 启用 Large Rawcut Handoff。
- 通过：进入 Session 2。
- 不通过：补材料或修正来源后重跑 Session 1。

### Session 2: Rawcut

- 读：`source_archive/` + `source_archive/source_index.md` + 目标角色名 / 别名 / 排除对象 / 目标时期。旧版运行可读 `reference/source_crawl.md`。
- 指令来源：`workflow-stage-a-source-canon/step2-rawcut.md`。
- 输出：`source_archive/character_rawcut.md`。
- 通过标准：严格按确认名切；主 cut 不并入未确认指代或无名辅助材料；scene slice 不按单句拆；原文裁剪为主体，备注只处理硬信息缺口。
- 分批规则：按来源文件或章节分批扫切，完成后合并为统一编号、原文顺序一致的 `source_archive/character_rawcut.md`。
- 大 rawcut：`source_archive/character_rawcut.md` 仍是完整证据仓；后续默认读取 `source_archive/rawcut_slices_index.json` + `check/Rawcut Evidence Brief.md` 或 `check/Rawcut Digest.md`，只按需回查目标 slice。

### Session 3: Rawcut Checker

- 读：`source_archive/character_rawcut.md` + `source_archive/source_index.md`；必要时抽查 `source_archive/` 或旧版 `reference/source_crawl.md`。
- 判据：
  - 目标角色名、确认别名、排除对象、目标时期是否已记录或可从用户输入明确推出。
  - Source Crawl 是否产出全量 raw evidence，而不是重写摘要；本地文本输入是否完整照录、未按目标角色预筛、未摘要。
  - `source_archive/character_rawcut.md` 是否按 scene slice 输出，而不是单句切分。
  - 每个 slice 是否包含 7 个 metadata 字段：来源、命中词、参与者、时期/状态、是否疑似、场景类型、裁剪原因。
  - 原文裁剪是否为主体，备注是否不超过 1 句。
  - 同一连续场景是否未拆成多条重复条目。
  - 是否避免在 trace 阶段做人格判断、二次叙述，或强行分类到“直接台词 / 关键行动 / 重大选择”等栏目。
  - 未确认指代与无显式名字辅助材料是否全部进入 `## 2`，并写成 3 字段卡：来源、触发词/归属、原文摘录；不得污染 `## 1` 主 cut。
  - 当前阶段是否未生成 persona、theme、world、source-map、audit 等额外产物。
- 输出：`check/Rawcut Check.md`，只写通过/不通过 + 薄弱点 + 最小修改建议。

### Session 4: Canon

- 读：`source_archive/character_rawcut.md` 或大 rawcut sidecar + 用户指定的目标时期；必要时用 `source_archive/source_index.md` / `source_archive/source_manifest.json` 定位回查原文。
- 指令来源：`workflow-stage-a-source-canon/step3-canon.md`。
- 不读：Step 1/Step 2 的执行指令；Self-Check；后续 distill/route/checker 规则。
- 输出：`canon.md`。
- 通过标准：生成中层事实资产，而不是剧情概述、人格分析或工程合规报告。

### Session 5: Canon Checker

- 读：`canon.md` + `source_archive/character_rawcut.md`；必要时读 WD 标杆或用户提供的质量参照。
- 判据：
  - `canon.md` 是否写事实而非人格结论，并按剧情节点顺序保留，不压成短 overview。
  - 文件开头是否声明它是 `reference` 与 `analysis.md` 之间的中间层，并写明冲突处理。
  - `## 0` 是否在材料支持时写出阶段画像；材料不足时是否正确退化，未硬凑空节点；`## 0.2` 是否按窄触发落全自述身份并使用三件套。
  - `## 1` 是否是薄速查，重要身份是否挂 `## 0` 的 T 编号，未重复整段人生或阶段差异。
  - `## 2` 是否保留独立的 `2.{N} 关键记忆清单`；关键记忆是否是人格生成资产而非剧情摘要；材料明显支持多条时是否至少写出 3 条证据卡。
  - `## 3` 是否优先用角色视角定义概念，而不是让机制名或世界观名词喧宾夺主。
  - `## 5` 是否完整展开 `核心情感 / 主要矛盾 / 不能写偏 / 来源` 四件套，字段名不得近义替换。
  - `## 6` 是否至少保住一个可直接复用的主时期口吻块；材料支持时是否保留第二时期差异。
  - `## 8` 是否是按侧面主题组织的人物侧面卡，字段名固定为 `可用侧面 / 使用边界 / 来源`，不得退化成 raw 索引、文件分桶或来源隔离区。
  - canon 推断是否只做文本内推断，并按“推断了什么 / 依据 / 没有继续推断到哪”三件套约束。
  - `## 3`-`## 7` 是否都有实质子结构：概念 3-8 行、剧情按时序、关系四件套、口吻三块、易写偏不少于 5 条。
  - 若材料明确支持后续阶段、战后回望、回乡、启程、自我整合等延续节点，是否进入 `## 0` 或 `## 4`，而不是只落在 `## 8`。
  - `## 9` 是否封顶不超过 5 条，并作为缺口区容纳“待回查”和“未归位”，每条说明为何还在这里。
  - `## 10` 是否写明优先查 canon 的情况、下钻 reference 的指针、上推 analysis/theme 的指针与冲突处理。
  - 是否消除“只有节标题、子结构为空”的失败模式；空缺处是否写“材料不足，待补”。
  - 是否避免生成独立 `world.md`，世界设定材料是否归并进 canon 的 `## 3` / `## 6` / `## 8`。
  - 是否仅当用户提供 AU 设定时才生成 `au_overrides.md`；未提供 AU 时不得生成。
  - 当前阶段是否未创建额外 upstream 产物，也未写 final persona 或 theme analysis。
- 输出：`check/Canon Check.md`，只写通过/不通过 + 最小修改建议。

### Session 6: Distill

- 读：定稿 `canon.md` + `source_archive/character_rawcut.md` 或大 rawcut sidecar + 用户目标用途 / 默认时期。
- 指令来源：`workflow-stage-b-distill/workflow-stage-b-distill.md` 全文。
- 输出：`analysis.md` / `persona.md` / `action.md`。
- S6 不生成 theme；若本轮需要主题层，完成 S7 后进入可选 Session 8。
- 通过标准：产物可运行、可审计，但不把 persona/action 写成证据报告。
- 默认时期：跨时期差异明显时先裁决；用户授权自主执行时由主 agent 写明选择、依据和风险，未授权时暂停。

### Session 7: Distill Checker + Smoke Test

- 读：`analysis.md` / `persona.md` / `action.md` + `canon.md` + Audit 阶段文件内置的 smoke questions。
- 判据：
  - persona 是否可运行，而不是证据报告或形容词人格。
  - 是否出现审计腔，例如“现有材料能支持的是...”作为运行态默认口吻。
  - 声线是否与 canon 的口吻依据一致，低压/高压/亲密/拒绝场景是否能区分。
  - 阶段是否未混淆，后期或 AU 未反向覆盖默认时期。
  - action 是否只规定任务条件和职责边界，不覆盖 persona Layer 0。
- 输出：`check/Distill Check.md`，含通过/不通过 + smoke test 结果 + 最小修改建议。

### Session 8（可选）: Theme Research

- 触发：用户明确要求主题层，或本轮用途明确需要哲学、文明伦理、思想谱系、文学 / 政治 / 宗教 / 心理等高阶坐标；否则直接进入 Session 9。
- 读：定稿 `canon.md` + `analysis.md` + `persona.md`；必要时按 canon 指针回查 rawcut / source。
- 指令来源：`workflow-stage-d-theme/workflow-stage-d-theme.md`。
- 输出：`theme.md`。
- 通过标准：只增加概念坐标，不反向改写 persona / analysis / action / canon；默认使用联网 `web-verified` 模式，除非用户禁网、环境不可联网或本轮明确要求 local-only。

### Session 9: Route

- 读：所有定稿产物：`canon.md` / `persona.md` / `analysis.md` / `action.md` / `relationship.md` / `theme.md` / memory templates 等。
- 指令来源：`workflow-stage-e-route/workflow-stage-e-route.md` 全文。
- 输出：最终角色包的 `SKILL.md`；必要时按模板放置 `memory.md` / `long_memory.md`。
- 通过标准：只布线，不重写人格；persona always-on；每个新问题先独立判断路由；analysis 可静默处理动机、因果、关系逻辑、价值冲突、反事实判断与深层人物理解，但不把普通对话自动改写成分析稿；action 仅处理拒绝、任务行为、决策、边界、场景与对象距离变化；canon/source_archive 处理事实、时间线、制度、历史与反事实因果。

### Session 10（可选）: Skill Checker

- 读：`SKILL.md` + 所有上游产物 + smoke questions。
- 判据：
  - `SKILL.md` 是否正确路由到各层。
  - 默认时期是否明确。
  - 日常聊天、analysis、action、theme、canon、reference、memory 的触发边界是否清晰。
  - 是否要求每个新问题重新判断路由，而不是沿用上一轮。
  - 深层理解问题（动机、因果、关系逻辑、价值冲突、反事实判断）是否触发 analysis，且普通寒暄、情绪承接、轻关系互动不因此外显成分析稿。
  - 制度、历史、体制失败与反事实因果问题是否触发 canon/source_archive。
  - memory 读取是否 command-gated；`/sum` 是否先 draft 再等确认。
  - 运行态回答是否接近标杆：自然、角色声线稳定、不过度工程化、不默认尾随追问。
  - 若 `analysis.md` 与 raw reference 冲突，运行规则是否要求以 raw reference 为准。
- 输出：`check/Skill Check.md`，含通过/不通过 + 整体质量报告。

---

## 3. 返修协议

- Checker 输出“不通过”时，用户决定是否返修；用户裁决权不可委托。
- 返修时，Create session 只读“当前产物 + checker 报告”，只改 checker 指出的节/字段，不触碰其他部分。
- 改完后重新提交同一 Checker session 验收。
- 连续返修不超过 2 轮仍不通过时，升级为用户人工介入，或回退上一阶段重做。
- Checker 报告只能给最小修改建议，不替 Create 重写正文。

---

## 4. Create / Checker 纪律

按交接文档 §8：

- Create 只生成，不审查；只解决当前指定阶段；不顺手修改相邻模块；不自带长篇 self-check；不把人话改成工程腔；输出应是最终文件正文，而不是解释自己将如何生成。
- Checker 只审查，不改写；给出通过/不通过；指出最薄弱处；给最小修改建议；不因为结构完整就判高分；优先看人物资产性、颗粒度、是否能喂 distill、是否接近标杆。
- 用户负责决定是否通过、是否回滚、是否把 checker 意见交回 Create，并防止 AI 顺手优化。

---

## 5. 术语表

- `agent`：工人，固定职责的执行者。
- `session`：房间，一个会积累上下文、也会被旧上下文污染的窗口。
- `skill`：公开操作入口；本项目只指本文件，以及流水线最终产出的角色 `SKILL.md`。
- `workflow`：工序安排，规定哪个阶段读取哪些文件、输出哪些文件、由哪个 checker 验收。
- `artifact`：交接文件，阶段之间继承文件而不是继承聊天记忆。
- `checker`：质检员，只验收和指出最小修改建议，不重写正文。

---

## 6. 轻量版本（基于 S1-S10 阶段编号）

本节是日常运行版。上文 `## 0`-`## 5` 保留完整 session spec，其中 S8 Theme Research 与 S10 Skill Checker 均为可选；轻量版只收窄默认执行链路，不改变各 session 的产物格式和质量边界。

### 6.1 轻量版流程

```text
Session 1: Source Crawl
  -> 主 agent / 用户检查点
  -> Session 2: Rawcut
  -> Session 4: Canon
  -> Session 5: Canon Checker
  -> 用户 / 主 agent 裁决
  -> Session 6: Distill
  -> (可选) Session 8: Theme Research
  -> Session 9: Route
  -> 人工快速试聊 / 结束
```

默认跳过：

- Session 3: Rawcut Checker
- Session 7: Distill Checker + Smoke Test
- Session 10: Skill Checker

跳过不是删除。它们仍然以上文 session spec 为准，只在严格模式升级时插回。

### 6.2 轻量版默认产物

| 产物 | 来源 | 说明 |
|---|---|---|
| `source_archive/` | Session 1 | 全量 source 保存、unit manifest、run manifest 与索引。 |
| `source_archive/character_rawcut.md` | Session 2 | 保持 raw scene slice，不写 summary/persona。 |
| `canon.md` | Session 4 | 中层事实资产，仍必须走 Canon Checker。 |
| `check/Canon Check.md` | Session 5 | 轻量版唯一默认 checker。 |
| `analysis.md` / `persona.md` / `action.md` | Session 6 | 核心运行人格与行动边界。 |
| `theme.md` | 可选 Session 8 | 调用时默认联网生成 `web-verified` 主题层；只作深层主题 overlay，不覆盖 persona/analysis/canon。 |
| `SKILL.md` + memory files | Session 9 | 只布线，不重写人格。 |

### 6.3 轻量版检查点

沿用上文 `主 agent / 用户检查点`。用户授权自主执行时，主 agent 可完成此检查点；未授权时停下等用户裁决。

### 6.4 轻量版各阶段读取规则

#### Source Crawl

- 仍按 Session 1 的重度 spec 执行。
- 大输入按 `step1-source-crawl.md` 的 script-first 与 source unit 契约执行。

#### Rawcut

- 仍按 Session 2 的重度 spec 输出 `source_archive/character_rawcut.md`。
- 主 cut 只使用 confirmed names / aliases。
- suspicious 指代、称号、关系名、无名辅助材料进入 `## 2`，不得并入 `## 1`。
- 若 rawcut 很大，后续 session 应通过 `source_archive/rawcut_slices_index.json`、`check/Rawcut Evidence Brief.md`、digest、代表片段或 targeted 回查读取，不默认全读完整 rawcut。

#### Canon + Canon Checker

- Canon 仍按 Session 4 的重度 spec 生成。
- Canon Checker 仍按 Session 5 的完整判据执行；这是轻量版唯一默认 checker。
- Canon Checker 通过后，才能进入 distill。若不通过，按返修协议由用户或主 agent 决定返修 / 接受风险。

#### Distill

- 仍按 Session 6 的重度 spec 生成 `analysis.md` / `persona.md` / `action.md`。
- 若跨时期差异明显，必须先决定默认运行时期。用户授权自主执行时，主 agent 写明默认时期、依据和风险；未授权时停下等用户选择。

#### Session 8（可选）: Theme Research

- 读：`canon.md` + `analysis.md` + `persona.md`；必要时按 canon 指针下钻 `source_archive/character_rawcut.md` / `source_archive/source_index.md`。
- 指令来源：`workflow-stage-d-theme/workflow-stage-d-theme.md`。
- 输出：`theme.md`。
- 轻量纪律：
  - 默认使用联网 `web-verified` 模式；只有用户禁网、运行环境不可联网，或本轮明确要求 local-only 时才输出 local-only 并在文件头标记。
  - theme 只在哲学命题、文明伦理、思想谱系、文学/政治/宗教/心理等高阶话题中提供坐标。
  - theme 不得覆盖 `persona.md`、`analysis.md`、`canon.md`。
  - 不默认扩成 theme-library；材料不支持时生成薄版 `theme.md`，写明可用范围有限。
  - 不为 theme 额外加 checker；最终由 Route 与人工快速试聊发现明显污染。

#### Route

- 仍按 Session 9 的重度 spec 生成最终 `SKILL.md`。
- `theme.md` 作为 conditional overlay 接入：persona always-on，analysis 静默辅助深层理解，action 条件调用，theme 仅深层话题调用，canon/reference 只在事实或证据问题中下钻。

### 6.5 严格模式升级

出现以下情况时，按 S1-S10 编号临时插回对应 checker：

- 插回 Session 3 Rawcut Checker：source 很大、alias 风险高、rawcut 体量异常大/小、用户怀疑切错材料。
- 插回 Session 7 Distill Checker + Smoke Test：persona/action 工程腔、声线不稳、时期混淆或 analysis 疑似压过人格。
- 插回 Session 10 Skill Checker：准备发布角色包、routing/memory/command 规则复杂、或用户要求最终验收。

轻量版不默认跑 Session 10。它是发布前验收工具，不是日常蒸馏主链。
