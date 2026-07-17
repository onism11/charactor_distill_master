# S_微调 Workflow Snapshot

这是 2026-07-17 发布的 `S_微调` 工作流主体。公开目录沿用既有 `charactor-distiller_Sv2` 路径，以避免无必要的链接迁移；版本身份以本文件和 Git 提交为准。

## 来源身份

- 导出来源：本地 active `lastestversion/`，提交 `bfc37ef5763cf494e37371f24eb4338a3478c3c4`。
- 单一 Skill 入口迁移：`43cdb43`。
- 当前最后一项 workflow 语义改动：`3ea90e4`（S1 大源单门）。
- 生产语义底本：Before-B/S；不采用 rerun、newversion 或失败实验的全链复杂编排。

公开副本只做两类发布适配，不改变生产 prompt：`AGENTS.md` 移除了未公开的 workbench/历史归档指针；根 `SKILL.md` 移除了一个未随主体发布的历史 donor 路径。远端原有的空白 `memory.md` 与 `long_memory.md` 模板原样保留。

## 发布范围

- 唯一入口 `SKILL.md` 与公开协作边界 `AGENTS.md`
- `workflow-stage-a-source-canon/`
- `workflow-stage-b-distill/` 的主线
- `workflow-stage-d-theme/`
- `workflow-stage-e-route/` 与空白 Memory 模板
- `workflow-stage-fg-audit/`；通用 smoke 题型已内置在 Audit 主文件中
- `workflow-prompts/Agent-A...G` 按需 controller 启动提示

## 明确排除

- experiments、versions、历史角色包、原始材料、运行日志、workbench 与评分工作台
- hard/soft/hardaction reference donor 与角色专属 smoke/benchmark 答卷
- 真实用户画像、历史对话记忆与本地路径
- `answer-agent-prompt.md`；隔离答题只在明确测试时按需传入 `workflow-prompts/Agent-F-answer-test.md`
