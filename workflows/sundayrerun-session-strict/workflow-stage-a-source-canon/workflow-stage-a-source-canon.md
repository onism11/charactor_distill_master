---
name: workflow-stage-a-source-canon
description: Process raw character materials inside the full workflow. Internal router for Agent A work only: Step 1 Source Crawl, Step 2 Rawcut, and Step 3 Canon.
---

# Workflow Stage A: Source, Rawcut, Canon

This is the lightweight router for the material trace phase inside the workflow. Do not use this file as the execution prompt for every stage.

## Step Files

- Session 1 Source Crawl: read [step1-source-crawl.md](./step1-source-crawl.md).
- Session 2 Rawcut: read [step2-rawcut.md](./step2-rawcut.md).
- Session 3 Canon: read [step3-canon.md](./step3-canon.md).

## Rule

In a workflow session, load only the matching step file plus the artifacts named by `../0workflow.md`. Do not preload all three step files.

## Outputs By Step

- Step 1 writes `source_archive/`, including `source_archive/source_index.md`, `source_archive/source_manifest.json`, and `source_archive/run_manifest.json` when scripted (legacy equivalent: `reference/source_crawl.md`).
- Step 2 writes `source_archive/character_rawcut.md` (scene-index ordered; every source unit becomes a `character_slice` / `plot_spine` / `auxiliary_boundary` entry), `check/Rawcut Evidence Brief.md`, and `source_archive/rawcut_slices_index.json` (with `entry_type`) — all always written.
- Step 3 writes `canon.md` and optional `au_overrides.md` only when the user provides AU settings. Canon remains character-centered, but must retain necessary institutional/system causes when they shape the character's choices or counterfactual limits.

## Scope

- Do not write `persona.md`, `analysis.md`, `action.md`, `theme.md`, or routed `SKILL.md` in this skill.
- Do not create source maps, audit files, examples, or extra docs unless the user explicitly asks.
- Do not generate an independent `world.md`; world/setting material is internalized into `canon.md`.
