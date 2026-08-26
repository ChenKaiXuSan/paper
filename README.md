# 论文阅读笔记

## 关于

本仓库用于整理研究论文阅读笔记，重点关注 3D 人体姿态、移动相机下的全局人体运动、多视角几何、360° 视觉、人体动作理解、医疗人工智能，以及体育与生物力学。

从 2026-08-07 起，新加入的论文笔记统一使用**中文总结与分析**。论文原标题、作者姓名、会议/期刊名称、模型名、方法名、数据集名和指标名等专有名词保留原始写法。

## 研究主题

- [Global Human Motion & Moving Camera](papers/global-human-motion/)
- [3D Human Pose & Mesh](papers/3d-human-pose/)
- [Multi-view Geometry & Camera](papers/multiview-geometry/)
- [360° / Omnidirectional Vision](papers/360-vision/)
- [Human Motion Understanding](papers/motion-understanding/)
- [Medical AI & Clinical Gait](papers/medical-ai/)
- [Sports & Biomechanics](papers/sports-biomechanics/)

## 研究专题 Collections

Collections 用于把多篇论文组织成研究问题、方法脉络和可直接使用的 Related Work 阅读路线：

- [Moving-camera World HMR](collections/moving-camera-world-hmr.md)
- [360° Selfie Human Reconstruction](collections/360-selfie-human-reconstruction.md)
- [Multi-view Pose Fusion](collections/multiview-pose-fusion.md)
- [Clinical Gait Analysis](collections/clinical-gait-analysis.md)
- [Sports Motion Analysis](collections/sports-motion-analysis.md)

## 论文索引

所有已收录论文统一维护在：

**[查看完整论文索引 →](PAPER_INDEX.md)**

README 不维护逐篇论文列表，以避免随着论文数量增加而失去可读性。

## 阅读状态

- `skimmed` — 已浏览论文结构、核心方法和主要结果
- `read` — 已完整阅读论文
- `deep-read` — 已深入分析、复现或进行系统性实验比较

## 新增论文笔记

1. 复制 [论文笔记模板](templates/paper-note-template.md)。
2. 将论文放入最相关的一级研究主题目录。
3. 使用 tags 表达跨主题属性，如 `slam`、`diffusion`、`vlm`、`clinical-gait`。
4. 文件名使用 `YYYY-short-paper-title.md`。
5. 更新 [PAPER_INDEX.md](PAPER_INDEX.md)。
6. 若论文改变某条研究脉络，可同步更新相关 [collection](collections/)。
7. 详细维护规则见 [AGENTS.md](AGENTS.md)。

## 许可协议

本仓库中的原创笔记采用 [CC BY 4.0](LICENSE) 许可；被引用论文的著作权仍归各自权利人所有。
