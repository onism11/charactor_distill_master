# 0workflow.md - Lean-First 角色蒸馏工作流

本文件是当前主线 workflow。旧 11-session hybrid / heavy hydration 版本仅作历史与 strict reference，不再作为默认 fallback。

对外唯一蒸馏入口是本 workflow。`workflow-stage-*` 目录都是内部阶段文档，不是可单独替代 controller 的公开蒸馏 skill。

Material Trace 已拆成三份 step 文件；当前主线只按需调用其中的轻量部分：

- `workflow-stage-a-source-canon/step1-source-crawl.md`：可借 source 收集 / 轻索引规则。
- `workflow-stage-a-source-canon/step2-rawcut.md`：只在 skinny strict 中借最小证据导航，不原样恢复重 rawcut。
- `workflow-stage-a-source-canon/step3-canon.md`：作为 Agent 1 的 canon 结构参考。

`workflow-stage-b-distill/workflow-stage-b-distill.md` 是 workflow 内部唯一 Stage B distill 主线。hard / soft / hardaction 只作 reference donor，不直接合入。

---

## 0. 总览

默认生产链路：

按照以下分工调用子agent：
```text
Agent 1: Prep & Canon
  -> strict preflight（材料体量 / alias / 时期 / plot_spine 需求；不默认产 strict 文件）
  -> Agent 2: Distill
  -> Agent 3: Route + Checker
  -> Agent F: Answer Test
  -> Agent G: Answer Review / Benchmark Comparison
  -> 核心包完成
  -> 询问是否追加 Theme Research
```

默认原则：

- 原始材料完整保留在 `source/`，下游按需回查原文。
- 默认做 strict preflight，但不默认产完整 skinny strict；只有触发升级条件才生成 strict 产物。
- 每次 run 默认跑 F/G 测试；如用户明确只要产包不测，可跳过 F/G 并在报告中标明。

严格边界：

- **skinny strict 不是 current heavy/hydration 原样 fallback**。
- 难材料升级时只补最小证据导航，不恢复重 scene-index / 全量 slice 机器 / 大 manifest。
- 声线证据、台词、低压互动、情绪 register 必须优先保留为可喂 persona 的证据，不得被降级到无用辅助区。
- 时期边界只做 gate；不得反复强调到压过人物本身。

---

## 1. 产物契约

主文件夹只保留核心运行文件与必要目录：

```text
SKILL.md
canon.md
persona.md
analysis.md
action.md
memory.md
long_memory.md
source/
check/
```

可选或严格模式产物：

```text
source/source_index.md
source/source_manifest.json
source/alias_gate.md
source/plot_spine.md
source/character_rawcut.md
check/Rawcut Evidence Brief.md
theme.md
```

产物说明：

| 产物 | 来源 | 必须包含 |
|---|---|---|
| `source/` | Agent 1 | 用户材料原文；轻爬材料；不得摘要覆盖原文 |
| `canon.md` | Agent 1 | 资料边界、人物事实、时间线、时期边界、alias 裁决、必要外部因果指针 |
| `analysis.md` | Agent 2 | 价值排序、动机因果、关系张力、失败模式、复杂反事实判断依据 |
| `persona.md` | Agent 2 | 声线、人格底座、对话适配、情绪 register、低压互动方式 |
| `action.md` | Agent 2 | 行动方式、拒绝 / 审问 / 保护 / 决策边界；保持简洁，不写成流程方案 |
| `SKILL.md` | Agent 2 或 3 | 路由规则；persona + analysis always-on；action/canon/theme 条件触发 |
| `source/character_rawcut.md` | skinny strict | 围绕确认 alias、人物高价值证据、声线证据、必要剧情 / 制度因果的连续原文摘录；服务定位回查，不做完整 scene-index |
| `check/Canon Check.md` | Agent 3 | 事实、时期、alias、剧情根因、声线证据、audit level |
| `check/Package Check.md` | Agent 3 | route、掉魂、声线、边界、污染检查 |
| `check/Answer Test.md` | Agent F | 隔离答题，不含评分 |
| `check/Answer Review.md` | Agent G | 评分、benchmark 对比、最小返修建议 |
| `theme.md` | 可选 Theme Research | 授权后生成；不进入核心交付链 |

---

## 2. Agent Spec

### Agent 1: Prep & Canon

读：

- 用户提供的本地文本 / md / docx 转写 / 网页材料。
- 必要时轻爬补充来源。
- 可参考 `workflow-stage-a-source-canon/step3-canon.md` 的 canon 结构。

写：

- `source/`：原文材料。
- `canon.md`。
- 必要时 `source/alias_gate.md`。

必须完成：

- 资料边界：材料覆盖范围、默认时期、未覆盖时期。
- alias / 同指代裁决：高风险称谓、头衔、关系名、组织身份不得静默硬判。
- 事实与时间线：按剧情顺序写，不压成短 overview。
- 必要外部因果指针：只记录直接解释人物处境、行动边界或悲剧根因的组织 / 剧情条件；不得扩写成制度分析主干。
- 下钻指针：写明哪些问题需要回查 `source/`。

不做：

- 不产大 manifest。
- 不产完整 rawcut。
- 不产 scene-index 机器。
- 不写人格结论或文体规则。

通过标准：

- 用户或主 agent 能看出“这是哪个时期、哪些材料、哪些称谓已裁决”。
- canon 不把系统性原因误写成角色个人问题，同时不让外部因果抢走人物主位。
- 后续 Distill 可以按 canon 指针回查原文。

### Agent 2: Distill

读：

- `canon.md`。
- `source/` 原文；按需定位回查连续上下文。
- `workflow-stage-b-distill/workflow-stage-b-distill.md`。

写：

- `analysis.md`。
- `persona.md`。
- `action.md`。
- `SKILL.md` 初稿或完整包。

必须完成：

- 先理解人物，再组织文件；不要用字段填空替代判断。
- 对关键判断主动回查 `source/` 原文，尤其价值形成、核心关系、关键行动、失败或转折、声线证据。
- `analysis.md` 承载复杂人物理解。
- `persona.md` 承载声线、情绪、人际距离和低压互动。
- `action.md` 只写行动模式与边界，保持比 persona/analysis 更轻。

不做：

- 不改 `workflow-stage-b-distill/workflow-stage-b-distill.md`。
- 不新增事实密度 / 短答 / 全面回答 / 时期强调等运行时强制行。
- 不让 action 变成普通对话发动机。
- 不把 checker 意见预写成正文防御性禁令。

通过标准：

- `persona + analysis` 是灵魂地基，`action` 条件触发。
- 包体能回答普通对话、关系、反事实、制度因果和压力行动题。
- 没有明显英语模板漂移、流程腔、评分器腔。

### Agent 3: Route + Checker

读：

- `canon.md` / `persona.md` / `analysis.md` / `action.md` / `SKILL.md`。
- 必要时抽查 `source/`。
- benchmark 母本和用户指定的质量参照；不得让 benchmark 内容泄漏给 Agent F。

写：

- `SKILL.md` 终稿（如 Agent 2 只产初稿）。
- `check/Canon Check.md`。
- `check/Package Check.md`。

Canon Check 必须检查：

- **Plot Spine Gate**：当问题确实依赖制度 / 剧情根因时，包体是否能解释，不得把系统性原因误写成角色个人问题。
- **Voice Evidence Gate**：标志性台词、低压互动、情绪 register 是否保留为可喂 persona 的证据。
- **Causal Adequacy Gate**：面对“为什么制度失败 / 谁该负责 / 悲剧如何发生”能否答出因果链。
- **Character Priority Gate**：外部因果、时期边界、剧情锚点是否压过人物本身；若压过，优先降权而不是继续加约束。
- **Period Boundary Gate**：默认时期与后日谈边界是否清楚，但不得过度时期中心化。
- **Alias Gate**：高风险同指代是否裁决。
- **Audit Level Gate**：明确标注 `lightweight pass` / `skinny strict pass` / `publication pass`。
- **Contamination Gate**：theme / route / 评分偏好是否反向污染 canon，尤其事实密度偏见。

Package Check 必须检查：

- 是否掉魂：对照参考母本 B 或用户指定标杆，看灵魂感、声线、情绪、判断方式。
- 是否过度分析：普通对话是否变成分析稿。
- 是否过度行动流程化：审问 / 行动是否像方案书而不是人物自然行为。
- 是否只懂人物不懂剧情根因。
- 是否存在事实错误、时期漂移、边界污染。

Checker 只写通过 / 不通过、薄弱点、最小修改建议；不重写正文。

### Agent F: Answer Test

读：

- tested package。
- 题单。
- `workflow-stage-fg-audit/answer-agent-prompt.md`。

禁止读：

- 评分标准。
- Expected / reference / benchmark answers。
- 其他变体答案。
- comparison / evaluation / checker 报告。

写：

- `check/Answer Test.md`。

通过标准：

- 每题重新按包判断路线。
- 不输出自评、分数、路由报告、文件报告、前言、总结或续问。

### Agent G: Answer Review / Benchmark Comparison

读：

- `check/Answer Test.md`。
- 评分标准 / smoketest / reference answer。
- benchmark 答案和对比材料。

写：

- `check/Answer Review.md`。
- 必要时 `check/Benchmark Comparison.md`。

评分重点：

- 灵魂感。
- 正确声线。
- 适当情绪。
- 事实正确。
- 必要剧情 / 制度根因。
- 不过度分析。
- 不以“越全面越好”奖励回答；包体覆盖密度和回答质感分开评。

---

## 3. Skinny Strict 升级协议

触发条件：

- 长篇全文、材料总量明显超过一次 distill/canon 可读余量，或 source 分散到多章节 / 多网页且缺少定位索引。
- alias / 同指代在 Agent 1 preflight 后仍不清，尤其目标角色常以名、姓、头衔、身份、重生 / 遗体 / 记忆状态混称。
- 角色强依赖组织、战争、政治、制度、社会结构，且现有筛选材料不足以直接写出 plot spine。制度复杂本身不是自动升级理由；若用户材料已经高度筛选、上下文可读、alias 已裁决，先走 lean，再由 checker 判定是否补 strict。
- Agent 3 发现非人物中心因果遗漏。
- Agent G 发现“懂人物但不懂剧情根因”。
- 用户要求发布级审计。

skinny strict 只补五项：

1. `source/source_index.md` 与 `source/source_manifest.json`：用于定位来源和回查，不做重摘要。
2. `source/alias_gate.md`：高风险称谓 / 身份 / 关系名裁决。
3. `source/plot_spine.md`：轻量剧情 / 制度因果链，只覆盖会改变人物处境、行动边界或悲剧根因的 source unit。
4. `source/character_rawcut.md`：瘦 rawcut / evidence excerpt，只摘连续原文与相邻上下文，覆盖确认 alias、人物高价值证据、关键声线证据、必要剧情 / 制度节点；不恢复完整 rawcut。
5. `check/Rawcut Evidence Brief.md`：薄导航图，只含高价值人物证据 4-8 簇 + 必要剧情 / 制度指针，并指向 `source/character_rawcut.md` 与原 source。

skinny strict 明确不做：

- 不恢复 current heavy/hydration 原样流程。
- 不生成大规模 scene-index。
- 不生成完整 slice 机器。
- 不把全台词或标志性语言降级到辅助区。
- 不把事实密度偏好写进 route / distill。

skinny strict 通过后：

- 回到 Agent 2，按新增证据导航回查 source。
- 不直接把 strict 产物搬进 persona/analysis/action。
- 再跑 Agent 3 + F/G。

---

## 4. Theme Research

Theme 不在核心链路内。

核心包完成后询问用户是否运行 Theme Research：

- yes：运行 `workflow-stage-d-theme`，生成 `theme.md`，再重跑 Agent 3 route 检查。
- no：结束核心包；可提供网页端 theme prompt 和推荐文件清单。

Theme 只加深哲学、政治、宗教、文学、伦理、心理或审美坐标，不覆盖 `persona.md`、`analysis.md` 或 `canon.md`。

---

## 5. 返修协议

- Checker 不通过时，只返修 checker 指出的最小部位。
- 返修 agent 只读当前产物 + checker 报告 + 必要 source，不读 benchmark 答案。
- 改完后重新提交同一 checker。
- 不因单次 answer-test 分数低就扩张 distill prompt；先判断是方差、source 缺口、canon 缺口、route 缺口，还是 persona/analysis 真缺口。

---

## 6. 运行记录

每次 run 的 `check/Answer Review.md` 或 `check/Benchmark Comparison.md` 头部必须记录：

- run id / 日期。
- mode：`lean` / `skinny strict` / `publication`。
- tested package 路径。
- 题单路径。
- answer prompt 路径。
- benchmark 路径。
- 是否跳过 Theme。
- 是否跳过 F/G；若跳过，必须写明用户授权或原因。

实验产物放入 `experiments/<日期>-<主题>/`，结论同步 `experiments/INDEX.md`、`workbench/STATE.md`、`workbench/LOG.md`。

---

## 7. 术语表

- `lean`：默认三 agent 生产链路。
- `skinny strict`：复杂材料的最小证据护栏，不等于 heavy/hydration。
- `heavy/hydration`：历史重水合链路，保留参考，不作为默认 fallback。
- `plot_spine`：必要剧情 / 制度因果指针，防止“懂人物但不懂根因”，但不取代人物主干。
- `voice evidence`：台词、低压互动、情绪 register 等可喂 persona 的声线证据。
- `F/G`：答题与评分隔离；F 不见评分，G 不重答。

