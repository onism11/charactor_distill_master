# Before Workflow - Session-Strict Publication Pipeline

本文件是 `sundayrerun` before 版的当前正式 workflow。它以 `0workflow-before.md` 为回滚快照和结构底稿，但默认主线改为 full session-strict。

`0workflow-before.md` 保留不动；本文件承担当前运行契约。

对外唯一蒸馏入口是本 workflow。`workflow-stage-*` 目录都是内部阶段文档，不是可单独替代 controller 的公开蒸馏 skill。

## 0. 总览

默认链路：

```text
S0: Controller Preflight
  -> S1: Source Crawl
  -> S1 Gate: period / alias / source coverage checkpoint
  -> S2: Rawcut
  -> S3: Rawcut Checker
  -> S4: Canon
  -> S5: Canon Checker
  -> S6: Distill
  -> S7: Distill Checker + Smoke
  -> S8: Theme Research
  -> S9: Route
  -> S10: Skill Checker
  -> F: Answer Test
  -> G: Answer Review / Benchmark Comparison
```

轻量链只允许作为显式实验模式，不得作为正式主线。

正式蒸馏、发布前验收、用户未指定省略时，一律跑 full session-strict。

严格不等于 hard mode。严格只体现在隔离、等待、回查、厚度、覆盖和验收；不得新增固定主题强调，不得把制度、时期、事实链、哲学词压过人物。

默认执行分组采用 before 版的 7-agent 纪律，不是每个 session 都 fresh：

| Agent | 默认负责 | 隔离要点 |
|---|---|---|
| A | S0-S5：preflight / source / rawcut / canon / trace-side checker | 可在同一上下文内完成材料链和结构自检；复杂材料可临时拆出 S2/S3 或 S4/S5，但不是默认。 |
| B | S6 Distill | 必须与 A 隔离；只读定稿 canon/source_archive/check 和 before distill prompt。 |
| C | S7 Distill Checker + Smoke | 必须与 B 隔离；只审查，不重写。 |
| D | S8 Theme Research | 可独立运行；theme 不反向改 core package。 |
| E | S9 Route + S10 Skill Checker | 可同上下文先布线再做 package check；只允许最小修 SKILL/runtime 小问题，不大改三件套。 |
| F | Answer Test | 必须隔离；不得读 benchmark / reference / comparison。 |
| G | Answer Review / Benchmark Comparison | 必须与 F 隔离；只评审，不重答，不回流给 F。 |

关键隔离边界是 A|B、B|C、F|G。其余 session 是 workflow 阶段，不等同于必须新开 agent；合并时只保留 agent 启动 prompt artifact。

Agent 启动 prompt 映射：

| Agent | 启动 prompt | 默认保存 |
|---|---|---|
| A | `workflow-prompts/Agent-A-prep-canon.md` | `prompts/Agent-A-prompt.md` |
| B | `workflow-prompts/Agent-B-distill.md` | `prompts/Agent-B-prompt.md` |
| C | `workflow-prompts/Agent-C-distill-checker.md` | `prompts/Agent-C-prompt.md` |
| D | `workflow-prompts/Agent-D-theme.md` | `prompts/Agent-D-prompt.md` |
| E | `workflow-prompts/Agent-E-route-checker.md` | `prompts/Agent-E-prompt.md` |
| F | `workflow-prompts/Agent-F-answer-test.md` | `prompts/Agent-F-prompt.md` |
| G | `workflow-prompts/Agent-G-answer-review.md` | `prompts/Agent-G-prompt.md` |

## 1. 运行目录契约

每次 run 输出目录固定为：

```text
<run>/
  canon.md
  analysis.md
  persona.md
  action.md
  theme.md
  SKILL.md
  memory.md
  long_memory.md
  source_archive/
  check/
  monitor/
  prompts/
```

最终运行主文件只保留 8 个：`canon.md`、`analysis.md`、`persona.md`、`action.md`、`theme.md`、`SKILL.md`、`memory.md`、`long_memory.md`。

`source_archive/` 存全部证据与机器索引；`check/` 存 checker、brief、answer/review；`monitor/` 存 session 状态；`prompts/` 保存本次实际发给每个 agent 的 prompt。

## 2. Controller 硬纪律

主线程只调度，不生产正文。

- 主线程不得替任何 producer agent 写 `canon.md` / `analysis.md` / `persona.md` / `action.md` / `SKILL.md`。
- 按默认 7-agent 分组调度；只有跨关键隔离边界时才必须 fresh agent，`fork_context=false`。
- `fork_context=false` 是主线程启动隔离 agent 时不继承主线程上下文的调度参数，不是写给子 agent 的正文；A 内部 S0-S5、E 内部 S9-S10 等合并阶段不需要。
- 主线程启动每个 agent 时，第一句话必须使用对应 `workflow-prompts/Agent-*.md` 的完整内容，不得临时改写、删减质量要求或把 workflow 摘要替代启动 prompt。
- 主线程必须等待当前阶段完成或当前 agent 提交正式 handoff，并检查产物存在后，才能进入下一阶段/agent；不得用主线程临时产物替代未完成的 worker 产物。
- 如果 agent 限流、报错、停在 dirty tree 或未写完，主线程只能暂停、重发许可或重启同阶段；不得自己接手写正文。
- 每个 agent 的实际启动 prompt 必须保存到 `prompts/Agent-{A-G}-prompt.md`；不再按 session 生成 `S{N}-prompt.md`。
- 每个 agent 任务开始前必须先创建 `monitor/Agent-{A-G}.status.md`，写入 `started`；任务完成后更新为 `completed` / `blocked` / `failed`，并标注读取文件、写入文件、风险。每个阶段按需更新。
- Checker 只审查，不重写 producer 正文。需要修复时退回对应 producer session 重新跑。

## 3. S0 Preflight

S0 必须在 S1 前完成：

- 目标角色名。
- 作品 / 来源范围。
- 用户指定默认时期。
- 若材料显示明显跨时期，且用户未指定默认时期，必须列出候选时期并停下等用户裁决。
- 若用户明确授权自主判断，主线程可选择默认时期，但必须写入 `monitor/S0-preflight.md`：候选、选择、依据、风险。
- 可疑 alias / title / relationship name 必须列入 `alias_gate`，不能静默当 confirmed alias。

S0 输出：

```text
monitor/S0-preflight.md

```

S0 不写角色正文，不替 S1 裁切材料。

## 4. Session 契约

### S1: Source Crawl

执行 agent：A（启动模板：`workflow-prompts/Agent-A-prep-canon.md`）

调用：`workflow-stage-a-source-canon/step1-source-crawl.md`

写入：

- `source_archive/source_manifest.json`
- `source_archive/source_index.md`
- `source_archive/run_manifest.json`
- `source_archive/alias_gate.md` 或等价 alias gate 段落


纪律：

- `source_archive/` 是全源证据仓，不以目标角色为筛选条件。
- S1 不裁决默认时期，只产候选、覆盖、alias 风险，交给 S1 Gate 明确记录。
- 可疑 alias / title / relationship name 必须进入 alias gate，不得静默扩张为 confirmed alias。

### S1 Gate: Period / Alias / Coverage

主线程检查 S1 产物：

- period 是否已由用户指定或由 S0 授权裁决。
- alias gate 是否区分 confirmed / suspicious / excluded。
- source coverage 是否足以进入 S2。

若 period 或 alias 风险会污染 rawcut，暂停并让用户裁决；若用户已授权自主判断，主线程记录裁决和风险。

Gate 输出：`monitor/S1-gate.md`

### S2: Rawcut

执行 agent：A（启动模板：`workflow-prompts/Agent-A-prep-canon.md`）

调用：`workflow-stage-a-source-canon/step2-rawcut.md`

写入：

- `source_archive/character_rawcut.md`
- `source_archive/rawcut_slices_index.json`
- `check/Rawcut Evidence Brief.md`

纪律：

- rawcut 是原料层，不写 persona / analysis 结论。
- scene-index 保持连续原文定位，不只切命中名。
- publication/full-complex 模式下，Evidence Brief 不限 4-8 簇；材料复杂时覆盖所有高价值人物证据簇。
- 不新增任何角色主题方向强调。

### S3: Rawcut Checker

执行 agent：A（启动模板：`workflow-prompts/Agent-A-prep-canon.md`）

写入：`check/Rawcut Check.md`

必须检查：

- source unit 是否都有 entry 或明确边界。
- confirmed / suspicious alias 是否被错误混用。
- character_slice / plot_spine / auxiliary_boundary 是否完整进入 index。
- Evidence Brief 是否足够导航 S4/S6，而不是写人格结论。
- 是否出现只扫命中名、丢剧情脊柱、丢低压互动、丢失败转折。

### S4: Canon

执行 agent：A（启动模板：`workflow-prompts/Agent-A-prep-canon.md`）

写入：`canon.md`、可选 `check/Canon Source Notes.md`

纪律：

- canon 是事实、时间线、时期边界、alias 裁决、剧情/关系因果图谱。
- 不写 persona 结论，不把制度或时期当主题强调。
- 非人物中心制度/组织/政治/军事因果只作为剧情根因和行动边界的低权重指针，不能压过人物主体。

### S5: Canon Checker

执行 agent：A（启动模板：`workflow-prompts/Agent-A-prep-canon.md`）

写入：`check/Canon Check.md`

必须检查：

- 事实、时间线、时期边界、alias 裁决是否可追溯。
- 是否遗漏会改变人物处境、行动边界或悲剧根因的非人物中心因果。
- 是否把可疑称谓当 confirmed alias。
- 是否把 canon 写成人格分析或主题论文。
- 是否存在因制度/时期锚点过重导致人物主体下降。

### S6: Distill

执行 agent：B（启动模板：`workflow-prompts/Agent-B-distill.md`）

调用：`workflow-stage-b-distill/workflow-stage-b-distill.md`

必须读取：

- `canon.md`
- `source_archive/source_index.md`
- `source_archive/rawcut_slices_index.json`
- `check/Rawcut Evidence Brief.md`
- `source_archive/character_rawcut.md` 中与 Evidence Brief 对应的关键 slice
- 必要时按 `source_index` 定位回查 `source_archive/` 原文
- `workflow-stage-b-distill/workflow-stage-b-distill.md`

写入：

- `analysis.md`
- `persona.md`
- `action.md`
- `check/Distill Self Check.md`


硬纪律：

- 禁止读取 benchmark、旧答案、评分报告、旧角色包、comparison 文件。
- 必须生成厚版 `analysis.md` / `persona.md` / `action.md`。
- 本 session 不是 summary，不是 lightweight。
- 轻量的是 source/prep，不是 persona/analysis/action。
- publication run 的 `analysis/persona/action` 合计默认不低于 18k 中文字符或同等信息密度；若低于下限，必须在 Self Check 中说明是材料不足，而不是主动省略。
- 不改 `workflow-stage-b-distill/workflow-stage-b-distill.md`。
- 不把 Evidence Brief coverage 写成固定主题强调；它们只是证据覆盖槽位。

### S7: Distill Checker + Smoke

执行 agent：C（启动模板：`workflow-prompts/Agent-C-distill-checker.md`）

写入：`check/Distill Check.md`、`check/Distill Smoke.md`

S7 只审查，不重写。

必须判定：

- 是否 summary 化。
- `analysis/persona/action` 是否达到 publication 密度。
- 是否覆盖 Evidence Brief 的关键证据簇。
- `persona` 是否有活人感、低压 register、对象差异。
- `analysis` 是否有深度但不把普通回答推成论文。
- `action` 是否条件触发，不压过 persona。
- 是否存在 hard 模式式内容过载：制度、时期、事实链、主题词是否压过人物。
- 是否存在通用 AI 腔、客服腔、评分器腔。

### S8: Theme Research

执行 agent：D（启动模板：`workflow-prompts/Agent-D-theme.md`）

写入：`theme.md`、可选 `check/Theme Notes.md`

纪律：

- Theme 只在深层思想 / 哲学 / 比较框架中触发。
- 默认可联网查询，但必须记录来源。
- 不反向改写 persona / analysis / action。

### S9: Route

执行 agent：E（启动模板：`workflow-prompts/Agent-E-route-checker.md`）

调用：`workflow-stage-e-route/workflow-stage-e-route.md`

写入：`SKILL.md`、`memory.md`、`long_memory.md`

必须保留：

- `persona + analysis` always-on。
- `action` 条件触发。
- `canon` 只在事实、时期、剧情、关系、反事实因果需要时触发。
- `theme` 只在深层思想 / 哲学 / 比较框架中触发。
- `memory` command-gated。
- 每个新问题重新判断 route。
- 用户指定的 genre / 格式 / 行数只有在不违背 persona 声线、拒绝风格与边界时才遵循；不得生成“表层形式总是跟随用户”的泛化规则。
- 不得新增“短答不是薄答”“先给结论”“全面回答”等文体强制行。

### S10: Skill Checker

执行 agent：E（启动模板：`workflow-prompts/Agent-E-route-checker.md`）

写入：`check/Skill Check.md`、`check/Route Check.md`

必须检查：

- 主文件是否只有 8 个。
- route 是否 obey persona > analysis > conditional action/theme/canon/memory。
- memory 是否 command-gated。
- `SKILL.md` 是否把 action 写成默认人格，或把 canon/theme 写成默认压制层。
- 是否新增文体强制行。
- 是否存在把用户 genre / 格式 / 行数置于 persona 声线、拒绝风格与边界之上的泛化规则；固定用 style-pollution 请求检查。
- 是否存在其他角色 memory 污染。
- 是否列出 8-10 个常见题型的预期 route，并能看出普通对话不会过度分析、action 不会流程腔、制度锚点不会压人物。

## 5. F/G Answer Test 隔离

### F: Answer Test

执行 agent：F（启动模板：`workflow-prompts/Agent-F-answer-test.md`）

F Answer Test 只读 tested package、题单、answer-agent prompt。
F 禁止读取 benchmark、reference answer、评分标准、comparison、其他变体答案。
F 只写 `check/Answer Test.md`，不自评。

每次正式 benchmark 必须保存实际 answer-agent prompt 到 `<run>/prompts/Agent-F-prompt.md`。

F 输出：`check/Answer Test.md`、`prompts/Agent-F-prompt.md`。

### G: Answer Review / Benchmark Comparison

执行 agent：G（启动模板：`workflow-prompts/Agent-G-answer-review.md`）

G Answer Review 只评审不重答。
G 可读 F 输出、benchmark、rubric、reference answer、comparison。
G 的结论不得回流给 F。

G 输出：`check/Answer Review.md`、`check/Benchmark Comparison.md`、`prompts/Agent-G-prompt.md`。

## 6. 静态验收

每次 workflow 修改后至少检查：

- `0workflow.md` 不再把轻量链列为默认。
- `0workflow.md` 明确写主线程不能接手 producer 文件。
- `0workflow.md` 明确采用 7-agent 分组、关键边界 fresh 隔离、prompt artifact
- `workflow-stage-b-distill/workflow-stage-b-distill.md` 没有被改动。
- route 中没有新增“短答不是薄答 / 先给结论 / 全面回答”类规则。

## 7. 流程试跑验收

正式验证 run 应满足：

- 产生 `prompts/`、`monitor/`、`source_archive/`、`check/`。
- S6 由独立 agent 生成，主线程未写正文。
- S6 output 达到 publication 厚度，或被 S7 拦截。
- F 用固定题单隔离答题。
- G 对 content A / B before / rerun2 做横评。
- 重点看：活人感、深度、敢拒绝、低压自然度、不过度制度锚点、不过度简略。

