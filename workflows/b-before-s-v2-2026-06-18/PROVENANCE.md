# S-v2 Workflow Snapshot

This directory reconstructs the workflow inputs used by the 2026-06-18 `S-v2` matrix experiment.

## Composition

- `0workflow.md`, skill1, skill3, skill4, and skill5: commit `a5fdc40` (`test: run distill matrix vs natural phase 3`).
- skill2 directory: commit `a5fdc40`, except `2distill-skill/2distill-skill-before.md`.
- `2distill-skill/2distill-skill-before.md`: commit `5c952d7`, the exact matrix prompt identified by the experiment gate summary.

This mixed provenance is intentional. The S-v2 experiment explicitly loaded the old matrix distill prompt with `git show 5c952d7:...`, while the rest of the workflow used the then-current files. Treat this directory as an immutable historical snapshot, not an active mainline.

## Included

- `0workflow.md`
- `1material-trace-skill/`
- `2distill-skill/`
- `3theme-research-skill/`
- `4auto-route-skill/`
- `5audit-test-skill/`

Experiment evidence remains at `experiments/2026-06-distill-matrix-vs-natural/`.
