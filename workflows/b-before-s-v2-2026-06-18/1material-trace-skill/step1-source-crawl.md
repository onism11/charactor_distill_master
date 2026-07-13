---
name: material-trace-step1-source-crawl
description: Step 1 of material trace. Capture or crawl raw character source materials into source_archive/ without filtering by target character. Use for Source Crawl sessions before rawcut/canon.
---

# Material Trace Step 1 - Source Crawl

## Purpose

Capture raw source materials before any character cut or canon writing. This step preserves source evidence and builds an index; it does not judge character relevance or write canon.

Outputs:

- `source_archive/`: full raw source captures, one file per source or normalized source unit.
- `source_archive/source_manifest.json`: machine-readable source/unit map with encoding, unit ranges, hashes, neighbors, and hit counts.
- `source_archive/source_index.md`: human-readable source list, unit section, file metadata, capture paths, size/hash notes, target-name hit counts if available, and coverage notes.
- `source_archive/run_manifest.json`: run metadata when scripting is used.
- Optional `source_archive/unit_cards.md`: semantic scene cards only when needed for causal recovery or strict review.
- Legacy equivalent: `reference/source_crawl.md` may be produced when an older run expects a single file, but it must preserve the same full raw evidence.

---

## Script-First Policy For Large Inputs

For a novel, directory of chapters, EPUB/HTML/TXT/MD bundle, or other large local input, do **not** read the full text into model context just to perform Source Crawl.

Default procedure:

1. Inspect file traits only: path, extension, file size, encoding if detectable, line count, chapter heading patterns, and at most small beginning/end samples when needed.
2. Write or run a deterministic local script suited to those traits.
3. The script copies/normalizes full raw text into `source_archive/`, writes `source_archive/source_manifest.json`, and writes `source_archive/source_index.md`.
4. The model reads `source_archive/source_index.md` and targeted samples, not the entire archive.
5. The human checks `source_archive/source_index.md` for coverage before Rawcut.

The script should preserve original wording and record enough metadata for later recovery: original path/URL, archive path, size or character count, content hash when practical, chapter/section outline when detectable, and hit counts for confirmed target names/aliases.

Read the full source in model context only when the source is small enough to fit cheaply, or when scripting cannot safely extract text and the user accepts the token cost.

### Source Unit Contract

For large or structured inputs, split source by the material's own structure: chapters/sections for novels, pages/headings for web materials, scenes for scripts. Small single-source material may be one unit.

`source_archive/source_manifest.json` records each unit:

- `unit_id`, source path/title, archive path, text range, byte/char/line counts
- detected title/chapter label, previous/next unit
- content hash when practical
- confirmed-name and alias-candidate hit counts

`source_archive/source_index.md` includes a compact unit section for human review. Do not ask the model to hand-write semantic cards for every unit; generate `source_archive/unit_cards.md` only when chapter causality is unclear, strict mode is requested, or canon needs a plot map.

### Encoding And Chapter Detection

- Probe encoding explicitly. Record the accepted encoding, failed encodings, and mojibake/replacement risk in `source_archive/source_index.md`; on Windows CJK paths, do not trust console rendering as the only evidence.
- For chapters/units, try common patterns before falling back: `第N章/回/节`, `Chapter N`, independent numeric headings, Markdown/HTML headings, short repeated title lines, and script scene labels.
- Do not implement multi-character heading labels as a character class. For example, use grouped alternatives like `(?:序章|楔子|尾声|终章|番外\\s*\\d*)`, not `[序楔尾终番外]`, because the latter treats any prose line starting with `外` or `终` as a chapter heading.
- Heading matches should be anchored to a whole short line, and should prefer explicit numeric/title markers over ordinary prose.
- `full_text` is allowed only as a documented fallback. If used, state which patterns failed and why the single unit is still acceptable.

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

Output this step into `source_archive/`, including `source_archive/source_index.md` and `source_archive/source_manifest.json`. If a legacy runner requires one combined file, `reference/source_crawl.md` is acceptable only as a lossless mirror of the same full raw evidence and should not be treated as a final-package main file.

Use this structure:

```markdown
# Source Index

目标角色：{name}（标注分析目标，不是 source_crawl 的过滤条件）
确认搜索名：{confirmed names}
采用编码：{encoding}
source manifest：`source_manifest.json`（本文件位于 `source_archive/` 内）

## 1. 来源清单

- 来源 1：{file / url / material title}
- 来源 2：...

## 2. Source Units

| Unit | 来源位置 | 存档路径 | 范围/体量 | 命中统计 | 章节策略 |
|---|---|---|---|---|---|
| U0001 | {chapter/title} | `source_archive/units/0001.txt` | {lines/chars} | {hit counts} | {pattern/fallback} |

## 3. Raw Captures

### 来源 1：{title}
- 类型：{file / url / paste / wiki / script}
- 存档路径：`source_archive/{normalized-name}.md`
- 体量：{bytes / chars / lines}
- 单元范围：{U0001-U00NN}
- 命中统计：{confirmed name hit counts, if available}
- 原文：
{若写入 `source_archive/source_index.md`，这里只放短样本或说明；完整原文在存档路径中。若写 legacy `reference/source_crawl.md`，此处按原顺序保留全文。}

### 来源 2：{title}
...
```

Rules:

- If the source is a web page or wiki page, fetch the page text before cutting by character name.
- If the source is already a local text file, **copy its full raw text** into `source_archive/` — no excerpting, no summarizing, no paraphrasing.
- Technical cleanup is allowed only for OCR noise, duplicated whitespace, broken bullets, and obvious markup debris.
- Do not convert prose into your own summary.
- Do not drop chronology markers, speaker labels, or section titles if they help later node ordering.
- Do not enforce automatic fetching; if the runtime environment lacks WebFetch, users are required to first paste the webpage text into a local .md/.txt file before continuing.
- **`source_archive/` 是全源证据仓，不以目标角色为筛选条件。** 连不含目标角色显式名字的段落也必须照录。禁止：按目标角色预筛 / 只摘看起来相关的片段 / 把长段改写成摘要 / 因目标时期是某期就删掉后期或辅助材料。是否"主干化"由 `character_rawcut` / `canon` 决定，**不在 Source Crawl 阶段判断**。
- **大文件分批**：若单个来源文件过大，按文件或章节分批归档到 `source_archive/`，保留原文顺序，并在 `source_archive/source_index.md` 记录分批关系。**禁止用"放不下"作为摘录式压缩的理由**——放不下就脚本化或分批，不是截断。
