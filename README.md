# 角色蒸馏器 | Character Distill Master

> 把原始剧情、台词、人物故事与设定，蒸馏成有判断、有声线、能长期运行的角色 Skill。

它不是人设摘要器。

角色蒸馏器从材料证据出发，依次建立角色事实、人格结构、思考方式、行动逻辑与运行路由。它关心的不只是“这个人有什么特点”，而是人物为什么这样判断、如何面对不同对象、矛盾何时暴露，以及说出口时是否仍然像他本人。

最终得到的不是一张标签表，而是一套可以对话、分析、行动并接受测试的角色运行包。

## 核心能力

- **证据驱动**：从连续原文追踪人物，而不是凭印象补全。
- **保留灵魂**：优先保护 `persona + analysis` 的厚度，让角色拥有矛盾、偏见与独立判断。
- **分层编译**：Material Trace → Distill → Theme → Route → Audit，各阶段只处理自己的问题。
- **按需加载**：日常对话不预载全部材料，复杂事实与主题研究在需要时才进入上下文。
- **可验证**：用隔离答题与评审检查路由、声线、边界和极端场景，而不是只检查 Markdown 格式。

## Skill

### [charactor-distiller_Sv2](./workflows/charactor-distiller_Sv2/)

2026-06-18 实际生成并测试过的五阶段版本，保留较厚的人格与分析层，并使用轻量场景矩阵改善对象和情境切换。

本次发布只保留运行所需主文件：Skill 入口、Material Trace、矩阵版 Distill、Theme、Route、空白 Memory 模板、Audit 与隔离答题提示。

## 设计来源

`persona.md` 的结构设计参考了 [dot-skill](https://github.com/titanwings/colleague-skill)。角色蒸馏器在此基础上扩展了原材料证据追踪、Analysis、Canon、Theme、运行路由与隔离测试链路。

## 使用方式

1. 将 `workflows/charactor-distiller_Sv2/` 作为 Skill 目录，从 `SKILL.md` 开始。
2. 按 Skill 入口调用对应阶段文件，不要一次性预载全部 prompt。
3. 将角色原始材料放入独立运行目录，最终角色包与 workflow 源码分开保存。

## 隐私边界

本仓库只发布工作流主文件，不包含原始角色材料、蒸馏成品、运行日志、测试答卷、真实用户画像或历史对话记忆。`memory.md` 与 `long_memory.md` 均为空白模板。

S-v2 的准确文件来源见 [PROVENANCE.md](./workflows/charactor-distiller_Sv2/PROVENANCE.md)。
