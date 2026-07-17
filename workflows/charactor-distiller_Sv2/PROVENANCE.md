# S-v2 Workflow Snapshot

这是 2026-06-18 `S-v2` 矩阵实验实际使用的工作流输入快照，不等同于表现异常出色的 `variant-B-before` 成品包。

## 来源

- 原 `0workflow.md`、skill1、skill3、skill4、skill5：提交 `a5fdc40`。
- skill2 的主目录结构：提交 `a5fdc40`。
- `2distill-skill/2distill-skill-before.md`：提交 `5c952d7`，即实验明确调用的矩阵版 Distill prompt。

这种混合来源是实验本身的真实状态：S-v2 运行时用 `git show 5c952d7:...` 读取旧矩阵版 Distill，其余阶段使用当时最新文件。核心发布版只额外清理了已省略 donor 的引用、规范了阶段相对路径，并将 `0workflow.md` 改名为带合法 frontmatter 的 `SKILL.md`；生成规则未改。

## 发布范围

- `SKILL.md`
- `1material-trace-skill/` 的主路由与三个步骤
- `2distill-skill/2distill-skill-before.md`
- `3theme-research-skill/3theme-research-skill.md`
- `4auto-route-skill/` 的主 Route 与空白 Memory 模板
- `5audit-test-skill/` 的 Audit 与隔离答题提示

历史 donor、hard/soft/hardaction 变体、旧 versions、原始材料、角色成品和实验答卷均未纳入发布。
