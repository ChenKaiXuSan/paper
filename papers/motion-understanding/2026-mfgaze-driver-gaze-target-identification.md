---
title: "Driver Gaze Target Identification via Gaze Estimation and Structured Driving Scene Understanding"
authors: "Xinze Cao, Ke Wang, Deyong Guan, Yuanquan Li, Ke Zhang"
venue: "arXiv"
year: 2026
reading_date: 2026-11-24
status: skimmed
tags:
  - driver-monitoring
  - gaze
  - head-pose
  - multimodal
  - uncertainty
  - calibration
---

# Driver Gaze Target Identification via Gaze Estimation and Structured Driving Scene Understanding

## 基本信息

- **作者：** Xinze Cao, Ke Wang, Deyong Guan, Yuanquan Li, Ke Zhang
- **会议/期刊：** arXiv 预印本；正式发表暂无核验
- **年份：** 2026
- **首次提交：** 2026-11-19
- **阅读日期：** 2026-11-24
- **阅读状态：** `skimmed`
- **标签：** driver-monitoring, gaze, head-pose, multimodal, uncertainty, calibration
- **论文：** https://arxiv.org/abs/2611.25771 （[HTML](https://arxiv.org/html/2611.25771v1)）
- **代码：** 暂无（第一作者账号的相关实现 https://github.com/XinzeCao/gazeflow- 可访问，README 使用 UniGaze，尚未确认与本文 MFGaze 完整对应）
- **数据集：** [Look Both Ways](https://github.com/Kasai2020/look_both_ways)（既有数据）；本文自采数据公开入口暂无
- **项目主页：** 暂无
- **价值类型：** Method Module / Baseline / Related Work
- **阅读优先级：** 中高

## 一句话总结

将头姿条件化 gaze estimation、道路 panoptic segmentation、个体标定与熵自适应语义融合组合，识别驾驶员实际注视的道路对象。

## 研究问题与动机

传统 gaze zone 分类无法区分同一区域内不同对象，连续注视点也容易受标定误差和目标尺度影响。作者提出模块化双相机系统，将视线从车内坐标映射到车外图像并识别候选对象。

## 核心方法

- MFGaze：MAE 预训练 ViT 与 head pose FiLM 调制，回归 gaze direction。
- Panoptic segmentation 将道路图像转换为带边界和语义类别的候选集合。
- 个体化二阶 polynomial ridge regression 估计 gaze-to-scene 映射。
- 按注视点到候选边界距离计算几何分数，并用归一化熵控制语义先验权重；几何不确定时增大语义贡献。
- 输入同步车内面部和车外道路图像；输出逐帧注视对象或区域。

## 数据集与评价指标

MPIIFaceGaze、ETH-XGaze、Gaze360 用于 gaze angular error 评价；模拟器标定 9 人、3,375 样本，按标定目标组内五折；Cityscapes 道路图像配合实验者注视指令获得 804 个有效测试样本。真实道路 LBW 为 28 人、75 条序列、123,297 对图像；每序列 30% 帧用于个体/序列标定、70% 留作评价，六类任务相关分析最终含 55,550 帧。指标为角度 MAE 和 Top-1 target accuracy。

## 主要结果

- Table II：ETH-XGaze baseline / Concat / FiLM 的 MAE 为 4.62° / 4.31° / 4.23°；跨域 ETH-XGaze→MPIIFaceGaze 为 9.10° / 7.31° / 6.93°。
- Table III：MPIIFaceGaze、ETH-XGaze、Gaze360 的 MAE 分别为 4.06°、4.23°、10.52°。跨域 Table IV 并非所有方向最优。
- Table V：模拟器 804 个样本，M1 中心距 75.75%、M2 边界距 74.25%、M3 固定融合 92.04%、M4 自适应融合 96.27%。
- Table IX：LBW 任务相关对象 Top-1 79.92%（M3 76.76%），road 85.58%；总体 84.82%。交通灯 47.06%、交通标志 47.37%，小目标困难。

## 优点

模块可独立评价；对比固定与自适应权重，结合真实道路数据测试。用头姿作为几何条件具有直接消融证据。

## 局限

LBW 每段使用标注 gaze point 标定，不等于 unseen-driver calibration-free generalization。对象标签由 eye tracker 注视点及 panoptic segmentation 生成，并非独立人工物体标注。模拟器注视指令不能等同于自然驾驶，真实道路低分辨率及类别不平衡影响小目标识别。**推断：**语义先验可能把异常但真实的注视错判为典型驾驶目标，尤其在 VFL compensation 中。完整本文代码尚未核验。

## 个人评价

**推断：**FiLM conditioning 和 ambiguity-aware fusion 可拆成独立模块，值得借鉴；注意 gaze direction、head direction 与 object-level attention 不能混为一谈，也不能将准确率直接解释为驾驶安全提升。

## 与我的研究关联

**推断：**可用现有模拟视野受限驾驶数据的三视角头姿 yaw/pitch/roll 作为条件，比较 RGB-only、头姿 concat、FiLM 与多视角置信度融合。若具备同步场景或模拟器对象位置及独立注视标注，可进一步构建对象级 attention 分支；否则仅评价头部扫描及方向分类，不声称真实眼球视线。按 day/night、交通密度及视野受限条件分别报告效果；区分按人标定与完全未见人员泛化。

## 后续阅读

- Sections III-A–D、IV-D、V-A/C/D，Tables II–IV、V、IX。
- 对比 EyeCue、DriverGaze360、Drive&Act；优先复现 FiLM 和基于熵的动态权重。
- 事实来源：arXiv 原文、LBW 作者官方仓库；相关 Gazeflow 仓库不当作完整论文复现代码。
