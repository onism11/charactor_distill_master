# Character Distill Master

Private archive of two Sunday character-distillation workflows.

这个仓库保存两条经过实际运行与比较的角色蒸馏生产线：一条强调完整的 session-strict 隔离与末端审计；另一条保留 B-before 时代更宽松、更靠近人物灵魂层的生成结构。

## Workflows

### [`sundayrerun-session-strict`](./workflows/sundayrerun-session-strict/)

`sundayrerun` 的完整 session-strict workflow 源码快照。

- A–G 分阶段运行与独立上下文边界。
- Source/Rawcut/Canon → Distill → Theme → Route → Answer/Review。
- Agent A–G 固定启动提示词。
- F/G 答题与评审隔离，支持 checker → repair → rerun 闭环。
- 同时保留 `0workflow-easy.md` 轻量入口。

版本标记：`sundayrerun-session-strict / snapshot-2026-07-13`。

### [`b-before-s-v2-2026-06-18`](./workflows/b-before-s-v2-2026-06-18/)

B-before 的 `S-v2-workflow-2026-06-18` 完整 Markdown 工作流快照。

- 五阶段 skill 结构：material trace、distill、theme、route、audit。
- 保留 before、hard、soft、hardaction 与历史 donor 文档。
- 包含 provenance、答题提示词与审计规范。
- 适合研究“厚 persona/analysis + 较薄外部纪律”的角色蒸馏路线。

版本标记：`B-before / S-v2 / 2026-06-18`。

## Privacy and packaging boundary

本仓库只保存 workflow 源码。以下内容没有上传：

- 原始角色材料与 source archives；
- distilled 成品角色包；
- runs、simulations、reports、monitor 与 telemetry；
- ZIP 备份和嵌套 Git 历史；
- 任何真实用户画像、关系记录和长期对话记忆。

原 workflow 中的 `memory.md` 与 `long_memory.md` 已替换为同名空白模板，保留运行契约但不携带私人数据。

## Suggested reading order

1. 先读目标目录的 `0workflow.md`。
2. 按 workflow 指向读取对应阶段文档。
3. 只有在目标阶段真正需要时，才载入 donor 或 checker 文件。

此仓库为私密工程归档，不代表公开发布版或稳定 API。
