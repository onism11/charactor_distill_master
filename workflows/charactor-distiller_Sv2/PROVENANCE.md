# S_微调 Workflow Snapshot

这是 2026-07-17 发布的 `S_微调` 工作流主体。公开目录沿用既有 `charactor-distiller_Sv2` 路径，以避免无必要的链接迁移；版本身份以本文件和 Git 提交为准。

## 来源身份

- 导出来源：本地 active `lastestversion/`，提交 `bfc37ef5763cf494e37371f24eb4338a3478c3c4`。
- 单一 Skill 入口迁移：`43cdb43`。
- 当前最后一项 workflow 语义改动：`3ea90e4`（S1 大源单门）。
- 生产语义底本：Before-B/S；不采用 rerun、newversion 或失败实验的全链复杂编排。

公开副本保持各 `workflow-stage-*` 生成 / 检查正文不变，只在公开编排与包装层做适配：

- `AGENTS.md` 移除未公开的 workbench 与历史归档指针。
- 根 `SKILL.md` 与 Agent B/C 移除未随主体发布的 donor、角色专属 benchmark 引用。
- 根 `SKILL.md` 与 Agent D/E 增加最小启动契约，并把 Theme 与 F/G 答题 Audit 明确为基础包完成后的后置选项。
- README 与 LICENSE 负责公开使用说明和授权；空白 `memory.md`、`long_memory.md` 模板保持原样。

这些改动会调整公开 controller 的执行顺序，但不改写 Source、Rawcut、Canon、Distill、Theme、Route 或 Audit 的阶段正文。

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
