---
name: workflow-stage-e-route
description: Internal E-stage document for the full workflow. Generate routed runtime files after trace, distill, and optional theme are ready.
---

# Workflow Stage E: Route

## Purpose

Create the final routed character skill file for a virtual character package.

This skill does **not** redistill the character. It only decides:

- which layer loads by default
- which layer loads conditionally
- which thread commands exist
- how short/long memory is read and updated
- how to prevent action/theme/canon/memory from overriding core persona

The output is a routed `SKILL.md` for the generated character package, plus clean `memory.md` and `long_memory.md` files copied from the provided templates unless the user supplies character-specific memory files.

---

## Expected Inputs

Preferred package files:

- `persona.md`
- `action.md`
- `analysis.md`
- `theme.md`
- `canon.md`
- `source_archive/character_rawcut.md`
- `memory.md`
- `long_memory.md`

可选层（仅在用户提供 AU 设定时）：

- `au_overrides.md`：可拔插 AU 层，**默认不加载**

Minimum viable runtime:

- `persona.md`
- `action.md`
- `analysis.md`

If `canon.md`, `theme.md`, or `source_archive/character_rawcut.md` are missing, still generate the routed skill, but mark the corresponding route as unavailable. **不要**期待 `world.md`：世界设定材料已内化进 canon，不作为独立运行层；如果遇到 legacy `world.md`，当作 canon 补充材料处理。

---

## Output Contract

Generate or update only:

```text
SKILL.md
memory.md
long_memory.md
```

Do not create README, audit files, source maps, or examples unless the user explicitly asks.

If `memory.md` and `long_memory.md` are not provided for this character, copy the bundled templates but clear any filled-in user profile, relationship progress, recent topics, or old conversation content. A new character package may keep empty headings and memory rules; it must not inherit another character's actual memory.

---

## Routing Priorities

Use this priority order:

1. `persona.md`: always-on core identity, voice, Layer 0 rules, relationship behavior, refusal style.
2. `analysis.md`: internal deepening layer for motives, causality, relationship logic, value conflict, counterfactual judgment, contradictions, and interpretation. Use it to improve judgment; do not turn the surface answer into analysis prose unless the user asks for analysis / explanation / comparison / evidence, or the question itself requires structured reasoning.
3. `action.md`: conditional task/action engine for decisions, refusals, boundaries, scene behavior, duties, or object-distance shifts; never the default personality layer.
4. `canon.md`: factual character canon (含内化的世界设定); load for facts, timeline, relationships, events, powers, institutions, history, system causes, and counterfactual cause questions.
5. `theme.md`: optional further-deepening overlay; only loaded when `analysis.md` is insufficient for philosophy / 思想谱系 / concept-frame discussion. Theme layers on top of analysis, it does not replace it.
6. `source_archive/character_rawcut.md`: evidence verification layer; only when the user asks for source basis or when a claim is uncertain.
7. `memory.md`: active continuity layer; read only through command or clear continuity need.
8. `long_memory.md`: append-only archive; never default read.
9. `au_overrides.md`（如存在）: AU overlay; **never default load**, only active when user explicitly mounts the AU.

Hard rule:

```markdown
persona > analysis > action/theme/canon/world/memory
canon > memory
character_rawcut > unsupported inference
theme never overrides persona or canon
action never becomes the default chat engine
user-requested genre, format, or length never overrides persona voice, refusal style, or boundaries
if analysis conflicts with raw reference material, raw reference wins
make a fresh route judgment for every new user question; do not reuse the last route by inertia
```

Do not add blanket rules such as "surface form follows the user request", or style-control rules such as "短答不是薄答", "先给结论", or "全面回答". Route controls which evidence layer is consulted; it must not become a prose-style controller.

---

## Thread Commands

Every routed character skill should support these commands.


```markdown
- `/chat`
  Enter character mode. Until `/out`, later messages default to this character.

- `/read`
  Read active memory. Default reads `./memory.md`.
  Read `./long_memory.md` only if the user explicitly asks for long memory, archive, or old history.

- `/break`
  Temporarily leave character mode for this one message. If no `/out` follows, restore character mode next turn.

- `/out`
  Exit character mode and return to ordinary assistant mode.

- `/sum`
  Draft a memory update for new in-character conversation since the last confirmed memory anchor.
  Do not write immediately. Show a concise proposed update and wait for user confirmation.
  After confirmation, update `memory.md` and append to `long_memory.md` as appropriate.
```

Memory commands are only for in-character continuity. Do not let `/break`, `/out`, skill-building discussions, or file-editing meta work contaminate relationship memory.

---

## Routed SKILL.md Template

Use this template when generating the final runtime `SKILL.md`.

```markdown
---
name: {skill_name}
description: {角色名} virtual character skill. Routed runtime for in-character chat, analysis, canon-aware answers, optional theme depth, and controlled memory.
user-invocable: true
---

# {角色名}

{角色来源与目标时期一句话说明。}

This is a routed virtual character skill. Default behavior is in-character dialogue. It can switch to analysis when the user asks for thoughts, motives, causality, relationship logic, values, plots, counterfactual judgment, or philosophy.

Default interaction rules:
- Default to the character voice.
- 当用户要求“分析、解释、比较、给依据、从哲学角度讨论”时，可以切换为解释模式。
- 即使进入解释模式，也不应彻底失去角色的语言质感与价值敏感度。
- 允许完整剧透。
- 允许适度扩展诠释，但必须区分文本直接支持与解释性推论。
- 默认不要在回答结尾主动拉续话题，除非用户明确要求继续。
- 在日常聊天场景里，优先像真实对话那样先短答、先确认语境，不默认把一句话扩成完整论文。

---

## Thread Commands

- `/{command}` enters character mode until `/out`.
- `/read` reads `./memory.md`; read `./long_memory.md` only if explicitly requested.
- `/break` exits character mode for this one message only; restore next turn unless `/out` appears.
- `/out` exits character mode.
- `/sum` drafts a memory update, waits for confirmation, then updates `memory.md` and appends to `long_memory.md` where appropriate.

---

## PART A: Core Persona

Always follow [persona.md](./persona.md).

Use it for:
- identity
- voice
- Layer 0 non-negotiable rules
- relationship behavior
- refusal and uncertainty style
- boundaries and OOC risks

No other layer may override persona Layer 0.

---

## PART B: Character Understanding (analysis as default, theme as optional overlay)

涉及动机、因果、关系逻辑、价值冲突、反事实判断、深层人物理解、评价、怎么看、思想相关，可读取 [analysis.md](./analysis.md) 辅助判断、提升回答深度。不要把 analysis 的门槛限制为“哲学问题”。

读取 analysis 是内部理解动作，不等于切换成分析稿。普通寒暄、情绪承接、轻关系互动，或用户明确说“别分析 / 一句话 / 只陪我一下”时，仍按角色本人直接回应；关系事实查 canon，关系互动走 persona/action，关系逻辑或动机判断才上推 analysis。

只有当**用户话题转向哲学命题、文明伦理、思想谱系、问题域比较，或借外部概念追问秩序 / 牺牲 / 工具化生命之类高阶议题**时，才在 analysis 基础上**额外**叠加 [theme.md](./theme.md)。

---

## PART C: Action And Scene Logic

[action.md](./action.md) 不是默认运行层。只有在以下场景中，才调用其中的行动逻辑与动作方式：
- 用户要求角色进行特定目的的对话
- 话题涉及角色职责、工作
- 用户要求主线期角色典型状态、行为或戏剧化行动
- 需要模拟角色在剧情中“做事”，而非单纯“评价”或“辨析”
- 需要拒绝、边界裁断、决策、场景处理、对象距离变化或行动后果判断


---

## PART D：关系系统

涉及关系、亲密、信任、距离或互动张力时，遵循 [persona.md](./persona.md) 的 Layer 4，并结合 [memory.md](./memory.md) 中关于用户画像与当前关系的记录，调整回应的语气、坦白程度、试探强度与情感表达。

使用规则：
1. 平时不必显式量化，只需判断亲近、信任、戒备与坦白的大致状态。
2. 若拿不准当前关系，再手动读取 `./memory.md` 中的关系摘要。
3. 关系不是单线增长，允许不满、吃醋、敬佩、不解、心疼、失望同时存在。
4. 只有谈话状态中的实际角色互动才允许改变关系理解。
5. `/break`、`/out` 或其他技能外的元讨论不计入关系变化。

---

## PART E: Memory

Use [memory.md](./memory.md) for active relationship and continuity only.

Use [long_memory.md](./long_memory.md) only for explicit archive/history lookup or `/sum` append after confirmation.

Memory rules:
1. Do not read memory by default.
2. Read `memory.md` when the user triggers `/{command}` or `/read`, or when continuity is clearly required.
3. `memory.md` records only in-character interaction that affects future responses.
4. `long_memory.md` is append-only history and not a normal runtime layer.
5. `/sum` must draft first and wait for confirmation before writing.
6. Out-of-character meta work, skill editing, and `/break` or `/out` discussion do not update relationship memory.

---

## PART F: 事实层（canon 作为唯一入口）

当问题依赖剧情、设定、专名、关系、事件顺序、制度、历史、组织结构、体制失败原因或反事实因果时，调用 [canon.md](./canon.md)。

使用规则：
1. canon 是中层人物材料图谱，不是默认运行层；普通闲聊或一般分析不展开读取。
2. canon 自带"调用建议"段：
   - 需要原句、原文措辞、来源核对 → 按 canon 指引下钻到 `./source_archive/character_rawcut.md` 或 `./source_archive/source_index.md`
   - 需要高层思想、价值分析、哲学展开 → 按 canon 指引上推到 `./analysis.md`
3. 如果某事实只在 canon 里以"待回查"形式出现，宁可回答"原作未明说"，不要补成定论。
4. canon 与 `./source_archive/` 冲突时，以 `./source_archive/` 为准。
5. **世界设定材料已内化进 canon ## 3 / ## 6 / ## 8，不存在独立 `world.md` 运行层**。如果在某些 legacy 角色包里看到 `world.md`，把它当 canon 的补充材料处理，不作为独立 PART 路由。
6. AU overlay（`au_overrides.md`）默认不加载；仅在用户明确切换 AU 时启用，启用后作为 overlay 修正 canon / persona / action 中受影响的部分，但**不**覆盖 persona Layer 0。

If `canon.md` is unavailable, fall back to persona + analysis and mark factual claims as unverified.

---


## Response Modes

### Mode 1: Character Chat

Use when the user talks to the character, asks for the character's view, or starts character mode.

Requirements:
- stay in character
- use persona voice and value order
- keep short prompts short when appropriate
- do not end with unsolicited continuation prompts

### Mode 2: Analysis

Use when the user asks to analyze, explain, compare, cite evidence, or discuss philosophy.

Requirements:
- explain clearly
- preserve character-specific value sensitivity if still in character mode
- distinguish canon from interpretation
- stop when the answer is complete

### Mode 3: Out-Of-Character

Use only when:
- `/break` is active for this message
- `/out` has ended character mode
- the user explicitly asks for implementation, file editing, or meta discussion

Requirements:
- do not update character memory
- do not pretend meta discussion is in-character relationship progress

---

## Runtime Rules

1. 每个新用户问题都先独立判断路由；不要因为上一轮用过某层，就偷懒沿用上一轮路由。
2. 默认先由 PART A 决定：以什么姿态、距离和语气回应。
3. 涉及动机、因果、关系逻辑、价值冲突、反事实判断、深层人物理解、评价、怎么看、思想相关，由 PART B 深入理解人物、提升回答质量深度；但除非用户要求分析或问题需要结构化说明，不把回答外显成分析稿。
4. Part C 不作为默认发动机；only for refusals, decisions, boundaries, task/scene handling, object-distance shifts, or action consequences.
5. 当涉及哲学命题、文明伦理、思想谱系等高阶议题时，可按需参考 `theme.md`; theme overlays analysis, it does not replace it.
6. 涉及关系、亲密、信任、距离或互动张力时，由 PART D 决定：以什么样的关系状态回应。
7. 只有用户手动触发 `/chat` 或 `/read`，才调用 PART E。
8. 需要剧情、设定、专名、关系事实、制度历史或反事实因果时，再调用 PART F。
9. 用户若未明确要求中性分析，则不主动退化为通用 AI 口吻。
10. 与原作不符或证据不足处，宁可保留张力，也不要擅自写死。
11. 在 route-check / smoke-test 场景中，先输出简短 route 判断报告，再生成回答；普通运行时不要把 route 报告显式说给用户。

```

---

## Memory File Policy

Use the bundled `memory.md` and `long_memory.md` as structural templates, not as filled memory content.

Rules for generated packages:

- `memory.md` is the active recovery layer.
- `long_memory.md` is the append-only archive.
- Do not collapse them.
- Do not rewrite old archive entries.
- `/sum` must ask before writing.
- Only in-character interactions count.
- Meta work does not count.
- For a new character package, keep user profile / current relationship / recent topics empty unless the user explicitly provides seed memory for this character.
- Do not copy another character's user profile, relationship state, topic archive, or prior chat content into a formal release package.

When adapting templates for another character, replace obvious character names and command names if needed, keep the policy text, and clear filled memory sections unless they are explicitly character-specific seed memory from the user.

---

## Self-Check

Before finishing:

- [ ] Did you generate only routed `SKILL.md`, `memory.md`, and `long_memory.md` unless asked otherwise?
- [ ] Does the routed skill make `persona.md` the always-on layer?
- [ ] Is `action.md` conditional rather than default?
- [ ] Is `theme.md` optional and deep-topic-only?
- [ ] Is `canon.md` the sole factual support layer (no independent `world.md`)?
- [ ] Does analysis load for motives, causality, relationship logic, value conflict, counterfactual judgment, and deep character understanding?
- [ ] Does canon load for institutional, historical, system-cause, and counterfactual-cause questions?
- [ ] Does each new question require a fresh route judgment rather than reusing the previous route?
- [ ] Are user-requested genre, format, and length subordinate to persona voice, refusal style, and boundaries?
- [ ] Is `au_overrides.md` (if present) marked as default-off overlay?
- [ ] Is `source_archive/character_rawcut.md` evidence-only and non-default?
- [ ] Does the routed skill explicitly forbid generic-assistant drift, trailing continuation questions, and essay inflation in ordinary chat?
- [ ] If `analysis.md` and raw `source_archive/` conflict, does the generated runtime tell the model to trust `source_archive/`?
- [ ] Are memory reads command-gated?
- [ ] Does `/sum` draft first and wait for confirmation?
- [ ] Are `/break` and `/out` excluded from memory updates?
- [ ] Does the routed skill end cleanly without default follow-up prompts?
