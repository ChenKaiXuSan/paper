---
title: "EventGait: Towards Robust Gait Recognition with Event Streams"
authors: "Senyan Xu, Shuai Chen, Chuanfu Shen, Kean Liu, Zhijing Sun, Chengzhi Cao, Xueyang Fu"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-08-26
status: skimmed
tags:
  - gait-analysis
  - event-camera
  - temporal-modeling
  - multimodal-learning
---

# EventGait: Towards Robust Gait Recognition with Event Streams

## 基本信息

- **作者：** Senyan Xu, Shuai Chen, Chuanfu Shen, Kean Liu, Zhijing Sun, Chengzhi Cao, Xueyang Fu
- **会议/期刊：** CVPR 2026
- **年份：** 2026
- **阅读日期：** 2026-08-26
- **阅读状态：** `skimmed`
- **标签：** `gait-analysis`, `event-camera`, `temporal-modeling`, `multimodal-learning`
- **论文：** [arXiv:2605.22139](https://arxiv.org/abs/2605.22139)
- **代码：** [QUEAHREN/EventGait](https://github.com/QUEAHREN/EventGait)
- **数据集：** SUSTech1K-E、CCGR-Mini-E、DVS128-Gait
- **项目主页：** 暂无

## 一句话总结

EventGait 用事件流的高时间分辨率补充普通 RGB 在低照度和运动模糊下的不足，并以 dynamic MoSE + static CroSA 双流结构同时建模短时运动与人体形状，用于更鲁棒的 gait recognition。

## 研究问题与动机

传统 gait recognition 多依赖 RGB/silhouette，在低照度、快速运动和模糊条件下容易丢失细节。事件相机能记录亮度变化，但长时间简单累积事件又会抹平细粒度运动，因此论文强调多时间尺度动态建模。

## 核心方法

动态流 MoSE 使用不同时间常数的 spiking experts 捕获短时和长时 motion；静态流 CroSA 借助 vision foundation model 提取形状结构。两支最终融合进行 gait identity recognition。

## 数据集与评价指标

SUSTech1K-E 约 25,239 sequences / 1050 identities；CCGR-Mini-E 约 47,884 sequences / 970 人 / 53 conditions / 33 views；真实事件数据 DVS128-Gait 约 4000 streams / 20 人。主要指标为 Rank-1、mAP、mINP。

## 主要结果

SUSTech1K-E 总体 Rank-1 约 92.8%；CCGR-Mini-E Rank-1/mAP/mINP 约 40.3/38.7/25.5；DVS128-Gait Rank-1 约 87.4%，高于 EV-Gait 的约 81.8%。双流模型 92.8%，static-only / dynamic-only 分别约 82.0% / 72.4%。

## 优点

- 明确利用事件相机对低照度和高速运动的优势。
- 静态形状与动态运动分开建模，结构清楚。
- 同时使用大规模合成事件数据和真实 DVS 数据验证。

## 局限

- 两个大规模 benchmark 的 event stream 主要由 RGB 合成；真实 DVS128-Gait 只有 20 人。
- 任务目标是 biometric identity recognition，不是临床疾病分类或运动学评估。
- 临床 gait 中低幅度异常能否从 event representation 保留尚未证明。

## 个人评价

这篇更适合作为传感器与时序建模灵感。其“static shape + dynamic motion”双流思想可以迁移到 RGB/keypoints/flow，而不一定必须使用事件相机。

## 与我的研究关联

可探索 `RGB/pose static branch + optical flow/event dynamic branch`，并结合 gait-cycle phase 做周期建模。对驾驶或高速体育，事件相机也可能解决夜间和 motion blur 条件下的动态感知问题。

## 后续阅读

- T-MOR。
- Action Motifs。
- 对比 optical flow 与 event-stream 在周期运动中的信息互补性。
