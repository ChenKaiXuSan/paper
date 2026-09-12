---
title: "Towards Context-Aware Clinical Motion Understanding in Daily Living at Home: Freezing of Gait Detection with Egocentric Vision"
authors: "Vayalet Stefanova, Diwas Lamsal, Margot Genbrugge, Maxim Yudayev, Christian Schlenstedt, Moran Gilat, Bart Vanrumste, Benjamin Filtjens"
venue: "ECCV 2026 MoCha Workshop; arXiv:2608.13283"
year: 2026
reading_date: 2026-09-13
status: skimmed
tags:
  - medical-ai
  - clinical-gait
  - parkinsons-disease
  - freezing-of-gait
  - egocentric-video
  - imu
  - multimodal
---

# Towards Context-Aware Clinical Motion Understanding in Daily Living at Home: Freezing of Gait Detection with Egocentric Vision

## 基本信息

- **作者：** Vayalet Stefanova, Diwas Lamsal, Margot Genbrugge, Maxim Yudayev, Christian Schlenstedt, Moran Gilat, Bart Vanrumste, Benjamin Filtjens
- **会议/期刊：** ECCV 2026 Workshop on Human Motion Challenges in Real-World and Clinical Settings (MoCha), Full Paper；arXiv:2608.13283
- **年份：** 2026
- **阅读日期：** 2026-09-13
- **阅读状态：** `skimmed`
- **标签：** `medical-ai`, `clinical-gait`, `parkinsons-disease`, `freezing-of-gait`, `egocentric-video`, `imu`, `multimodal`
- **论文：** https://arxiv.org/abs/2608.13283
- **DOI：** https://doi.org/10.48550/arXiv.2608.13283
- **代码：** 暂无
- **数据集：** 暂无
- **项目主页：** 暂无
- **会议页面：** https://mocha.care-pd.ca/
- **价值类型：** Method Module / Related Work
- **阅读优先级：** A（高）

## 一句话总结

这项工作用真实居家 Parkinson's disease 场景检验 egocentric video 是否能为 Freezing of Gait（FOG）提供超出纯运动学信号的上下文信息，结果显示视觉表征具有独立判别能力，但现阶段仍明显落后于专门训练的 IMU TCN。

## 研究问题与动机

Freezing of Gait 是 Parkinson's disease 中高度依赖环境与任务上下文的运动障碍。在真实日常生活里，IMU 中相似的减速、停顿或身体运动可能分别来自主动停下、与物体交互，或真正的病理性 FOG。只依赖惯性运动模式容易把这些不同原因混在一起，因此作者探索第一视角视频是否能够补充“人在做什么、身处什么环境”的视觉上下文。

与常见受控步态实验不同，本文数据直接来自患者家中，并采用 synchronized egocentric video、wearable IMUs 和专家 FOG annotation。研究重点不是重新训练一个大型视觉模型，而是比较 pretrained ego-video / time-series foundation representations 与一个从头训练的 IMU-based TCN 在跨受试者检测上的能力。

## 核心方法

### Egocentric visual context

作者使用预训练 ego-video foundation model 的 frozen representations，将视频片段编码为上下文特征，再训练下游 FOG detector。论文明确报告了 V-JEPA2 视觉特征的结果，用它衡量通用视频表征在没有专门临床预训练的情况下能否捕获 FOG 相关信息。

### Wearable time-series 与 supervised baseline

与视觉路线并行，作者评估预训练 time-series foundation representations，并设置一个从头训练的 IMU-based Temporal Convolutional Network（TCN）作为任务特定 baseline。所有模型使用 expert-annotated FOG labels 进行事件检测评价。

### Leave-one-subject-out protocol

由于临床样本量较小，论文采用 leave-one-subject-out（LOSO）评估，避免同一患者的数据同时进入训练和测试，从而更接近“对未见患者泛化”的实际使用条件。

## 数据集与评价指标

- **数据来源：** 真实家庭环境中的 synchronized egocentric video + wearable IMUs + expert-annotated FOG labels。
- **参与者：** 13 名 Parkinson's disease 患者。
- **输入：** 第一视角视频或 wearable IMU time series；论文比较 frozen foundation representations 与专门训练的 IMU TCN。
- **输出：** FOG event detection。
- **划分：** leave-one-subject-out evaluation。
- **主要指标：** F1、AUROC。
- 本次从允许的一手来源未核验到公开数据下载页、总视频时长或事件总数，因此不自行补全。

## 主要结果

任务特定 **IMU-based TCN** 表现最好，达到 **42.3 F1、83.0 AUROC**。使用 **V-JEPA2 egocentric-video features** 的视觉方法达到 **32.6 F1、77.2 AUROC**。因此，单独视觉上下文目前没有超过 wearable motion sensing，但其性能明显高于随机水平，说明第一视角场景与任务内容中确实包含 FOG 相关线索。

作者的定性分析进一步认为，egocentric vision 捕获的信息可能与 IMU 不完全重复。论文据此把视觉定位为未来 multimodal clinical motion understanding 的“上下文补充”，而不是替代 IMU 的单一传感器方案。

## 优点

- 使用真实患者居家 ADL，而不是实验室规定路线，更接近真实 clinical deployment。
- LOSO protocol 避免明显的 subject leakage，对只有 13 名患者的小样本研究尤其重要。
- 明确比较通用 pretrained representation 与任务特定 IMU TCN，能够区分 foundation feature 的迁移能力和 supervised task model 的优势。
- 研究问题具有临床解释性：视觉不是为了重复估计运动幅度，而是提供“为什么停下”的环境与任务上下文。

## 局限

- 只有 **13 名 PD 参与者**，外部中心、不同家庭环境和不同病情严重度的泛化仍未得到充分验证。
- 视觉单模态结果仍低于 IMU TCN，当前证据不能支持用 camera 取代 wearable sensing。
- 本次未核验到官方代码和公开数据，因此复现条件有限。
- 论文主要报告事件检测 F1 / AUROC，尚不足以回答长期家庭部署中的 false alarms per hour、连续监测稳定性和患者级 calibration。
- **推断：**从摘要和已核验结果看，论文证明了视觉具有互补潜力，但并不能仅凭现有单模态比较断言 video+IMU fusion 一定优于最佳 IMU baseline；这一点需要专门融合实验验证。

## 个人评价

这篇论文的价值在于把 clinical motion understanding 从“只看人体怎么动”推进到“结合人在什么情境下运动”。对于病理步态，很多表面相似的运动片段可能具有完全不同的临床含义，因此 context-aware representation 很可能比单纯增加 pose backbone 容量更有价值。

与此同时，32.6 F1 与 42.3 F1 的差距也说明通用视频 foundation feature 还没有直接解决临床事件检测。更现实的方向是让 RGB / ego-video 负责语境、pose/biomechanics 负责人体现象、IMU 负责高频运动，再用 uncertainty-aware fusion 判断何时相信哪种模态。

## 与我的研究关联

对临床步态和脊柱疾病视频分析，**推断：**可以把当前 `RGB + optical flow + keypoints / 3D pose` 的多模态路线进一步拆成：

`kinematics-only → RGB context-only → kinematics + scene/task context → + clinical knowledge / VLM explanation`。

例如 ASD gait 中，步速降低、转身、停顿或躯干姿态变化可能同时受场景任务和病理影响。可以在 pose-based temporal representation 外加入 video foundation feature，测试其是否减少因非病理行为导致的 false positives；同时应采用 subject-disjoint、site-disjoint 和不同场景的外部验证。

对于 driver behavior 研究，这一思路也可迁移为“head/body motion + road/task context”：相似的 head turn 在不同 traffic state 下可能代表不同驾驶意图或风险。

## 后续阅读

- CARE-PD：用于理解多中心 Parkinson's gait benchmark 与 clinical domain shift。
- GAITGen：病理步态 severity-conditioned generation 与小样本增强。
- GaitEncoder / Gait Foundation Model：与 pretrained motion representation 路线比较。
- BioGait-VLM：检查视觉、运动学、语言解释如何形成 clinically grounded multimodal representation。
- 后续实验重点应比较 `IMU-only / video-only / pose-only / video+IMU / video+pose+IMU`，并加入 calibration、false alarms/hour 与患者级泛化指标。
