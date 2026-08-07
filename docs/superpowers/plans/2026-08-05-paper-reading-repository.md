# Paper Reading Repository Implementation Plan — Historical

> **状态：已归档。** 本文件记录 2026-08-05 初始化仓库时的实施计划，不再作为当前维护规则使用。
>
> 当前有效规则请以仓库根目录的 [`AGENTS.md`](../../../AGENTS.md) 和 [`docs/superpowers/specs/2026-08-05-paper-reading-repository-design.md`](../specs/2026-08-05-paper-reading-repository-design.md) 为准。

## 历史背景

2026-08-05 的初始版本完成了以下工作：

- 创建公开仓库 `ChenKaiXuSan/paper`；
- 建立 `README.md`、`LICENSE` 和论文笔记模板；
- 建立五个主题目录：`3d-human-pose`、`360-vision`、`multiview`、`medical-ai`、`others`；
- 约定一篇论文对应一个 Markdown 文件；
- 使用 `skimmed`、`read`、`deep-read` 表示阅读状态；
- 使用 CC BY 4.0 许可原创笔记。

## 已被后续规则替代的部分

初始计划曾要求：

1. 论文笔记采用中英双语；
2. README 同时维护 `Recently Read` 和 `All Papers` 两个逐篇论文列表。

这两项要求已于 **2026-08-07** 被正式替代：

- 新论文笔记统一使用中文总结和分析；
- 完整论文列表迁移到独立的 [`PAPER_INDEX.md`](../../../PAPER_INDEX.md)；
- README 只保留仓库入口和索引链接；
- 已有历史双语笔记无需回溯重写。

本文件仅用于保留仓库初始化历史，后续 agent 不应从本文件推导当前格式要求。
