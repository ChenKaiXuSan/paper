---
title: "BioGait-VLM: A Tri-Modal Vision-Language-Biomechanics Framework for Interpretable Clinical Gait Assessment"
authors: "Erdong Chen, Yuyang Ji, Jacob K. Greenberg, Benjamin Steel, Faraz Arkam, Abigail Lewis, Pranay Singh, Feng Liu"
venue: "arXiv / MICCAI 2026 provisional acceptance"
year: 2026
reading_date: 2026-08-26
status: skimmed
tags:
  - medical-ai
  - gait-analysis
  - vision-language-model
  - biomechanics
  - explainable-ai
---

# BioGait-VLM: A Tri-Modal Vision-Language-Biomechanics Framework for Interpretable Clinical Gait Assessment

## 基本信息

- **作者：** Erdong Chen, Yuyang Ji, Jacob K. Greenberg, Benjamin Steel, Faraz Arkam, Abigail Lewis, Pranay Singh, Feng Liu
- **会议/期刊：** arXiv；作者团队公告为 MICCAI 2026 provisional acceptance
- **年份：** 2026
- **阅读日期：** 2026-08-26
- **阅读状态：** `skimmed`
- **标签：** `medical-ai`, `gait-analysis`, `vision-language-model`, `biomechanics`, `explainable-ai`
- **论文：** [arXiv:2603.08564](https://arxiv.org/abs/2603.08564)
- **代码：** 暂无
- **数据集：** 自建 DCM 临床队列 + 多来源 gait clips
- **项目主页：** 暂无

## 一句话总结

BioGait-VLM 把 RGB 视频、3D skeleton biomechanics 和语言推理放进同一 tri-modal framework，以临床运动学 token 约束 VLM 的时序证据，从而同时完成 gait pathology classification 与自然语言解释。

## 研究问题与动机

医疗视频模型容易依赖背景、衣着等视觉捷径，同时普通 VLM 对细粒度生物力学变化缺少结构化理解。论文希望把 3D pose-derived biomechanics 显式注入视觉语言模型，让预测理由更贴近临床运动学证据。

## 核心方法

视觉支路采用冻结的 InternVL3.5-1B；Temporal Evidence Distillation 使用 motion queries 聚合时序信息；Biomechanical Tokenization 将 HSMR/SKEL 产生的姿态/运动变量编码到语言空间。三类信息共同用于多类 gait pathology classification 和解释生成。

## 数据集与评价指标

统一实验包含约 1181 个 gait clips。自建 DCM 队列含 30 名患者、239 段 1080p/30fps 视频；subject-disjoint 后整体约 915 train / 266 test。评价 Accuracy、Macro-F1、类别 F1，以及专家对解释 grounding/usefulness/consistency 的评分。

## 主要结果

完整模型 Accuracy 约 68.1%、Macro-F1 52.9%；DCM F1 约 98.1%，abnormal gait F1 约 80.4%。TSN baseline 约 37.1% Accuracy / 35.9% Macro-F1。去掉 TED 后 Macro-F1 约 42.4%，去掉 biomechanics branch 后约 43.8%。专家盲评也显示完整模型解释更容易被选择为最佳。

## 优点

- 将 biomechanics 作为结构化 token，而不只是一般 pose feature。
- subject-disjoint split 比 clip-level random split 更符合临床验证要求。
- 同时评价诊断性能和解释质量。

## 局限

- DCM 临床队列仅 30 名患者，跨医院/跨相机泛化仍不足。
- biomechanics branch 依赖上游 HMR/SKEL，pose bias 可能进入解释。
- VLM 产生的合理文字不等于因果医学解释，需要更严格临床验证。

## 个人评价

这篇对临床视频研究非常贴近。最大的价值是把“临床知识如何进入模型”具体化为 biomechanical token，而不是只在 attention map 后解释。

## 与我的研究关联

可以把 gait-cycle phase、trunk inclination、pelvic rotation、左右不对称、关节 ROM 等显式作为 clinical tokens，与 RGB/flow/keypoint backbone 融合；也可用于脊柱疾病视频分类，研究哪些运动变量真正支持诊断。

## 后续阅读

- SIMSPINE。
- MedVCR。
- CARE-PD。
- 对 biomechanical token 做 uncertainty / cross-site validation。
