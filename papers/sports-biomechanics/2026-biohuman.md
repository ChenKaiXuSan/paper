---
title: "BioHuman: Learning Biomechanical Human Representations from Video"
authors: "Yujun Huo, He Zhang, Chentao Song, Honglin Song, Zongyu Zuo, Tao Yu"
venue: "arXiv:2605.14772"
year: 2026
reading_date: 2026-09-21
status: skimmed
tags:
  - sports-biomechanics
  - muscle-activation
  - monocular-video
  - opensim
  - biomechanics
  - multimodal
---

# BioHuman: Learning Biomechanical Human Representations from Video

## 基本信息

- **作者：** Yujun Huo, He Zhang, Chentao Song, Honglin Song, Zongyu Zuo, Tao Yu
- **会议/期刊：** arXiv:2605.14772
- **年份：** 2026
- **阅读日期：** 2026-09-21
- **阅读状态：** `skimmed`
- **标签：** `sports-biomechanics`, `muscle-activation`, `monocular-video`, `opensim`, `biomechanics`, `multimodal`
- **论文：** https://arxiv.org/abs/2605.14772
- **DOI：** https://doi.org/10.48550/arXiv.2605.14772
- **代码：** 暂无
- **数据集：** 暂无
- **项目主页：** 暂无

## 一句话总结

BioHuman 将 monocular HMR 与 full-body muscle activation prediction 作为一个端到端 video-to-biomechanics 问题联合训练，并通过 BioSim 构建 10.1M-frame BioHuman10M，使视觉运动分析从表面 kinematics 进一步延伸到内部 musculoskeletal state。

## 研究问题与动机

现有视频人体重建主要回答“人体怎样运动”，输出 SMPL pose/mesh 等 kinematics；真正与康复、损伤风险和运动表现相关的 muscle activation 等内部 biomechanical states 通常需要第二阶段 OpenSim 或 learned simulator。这样的串联流程既缺少大规模 video-motion-biomechanics 配对数据，也会把 HMR 误差直接传给后续肌肉估计。

本文的核心观点是：motion 与 muscle activation 本来就相互约束，不应先把视频压缩成固定 pose trajectory 再做 biomechanics。作者因此先建立 BioHuman10M，再提出 BioHuman，让视觉、运动学与 muscle supervision 在同一 temporal representation 中联合优化。

## 核心方法

### BioSim / BioHuman10M

1. 将 MotionPRO、EMDB、3DPW、Human3.6M、BEDLAM 的 SMPL motion 转成 OpenSim-compatible kinematics。
2. 构建 ULBS-112 full-body musculoskeletal model，包含 **112 muscles**；用 virtual anatomical markers 与 inverse kinematics 恢复 joint-angle trajectories。
3. 因原始 SMPL 没有 GRF，使用 GaitDynamics 根据 kinematics、身高和体重预测双脚 3D GRF 与 CoP。
4. 用 inverse dynamics 计算 joint moments，再通过 OpenSim static optimization 求 muscle activations；过滤不可靠 IK/SO 结果并做 temporal smoothing。

### BioHuman

- 输入 monocular video clips 与 person boxes。
- 使用 PromptHMR 作为 visual-kinematic front-end，提取初始 SMPL pose 与 image-conditioned HMR features。
- 通过 trainable temporal transformer 联合建模 pose tokens 与 visual tokens。
- 两个共享 latent state 的预测头分别输出 **72-D SMPL pose** 与 **112-D muscle activations**；pose head 以 PromptHMR 为初始化预测 residual correction。
- 训练同时约束 pose、muscle activation、temporal difference、waveform correlation 与 amplitude calibration，使 biomechanics supervision 可以反向影响仍保留视觉证据的 HMR representation。

## 数据集与评价指标

- **BioHuman10M：10.1M frames**，来源为 MotionPRO 4.7M、EMDB 34K、3DPW 55K、Human3.6M 73K、BEDLAM 5.3M。
- 测试协议按 sequence 或 subject 划分，避免 frame leakage；导出的完整 test protocol 有 **35,837 temporal windows**，每个最多 32 frames。
- 论文没有汇总五个来源合并后的唯一受试者总数，因此不自行推断。
- **输入：** RGB frames、bounding boxes、sequence identity、timestamps；推理时不使用 GT pose、OpenSim state、GRF 或 muscle labels。
- **输出：** 72-D SMPL pose + 112-D muscle activations。
- **肌肉指标：** PCC、RMSE、nRMSE、Active MAE@0.10。
- **运动指标：** pose RMSE 与 PA-MPJPE（使用 GT shape 系数计算 joints）。

## 主要结果

- 对比 deployable 两阶段 **PromptHMR + MinT**，BioHuman 的 muscle PCC 从 **0.42 提升到 0.71**，RMSE 从 **0.071 降至 0.065**，nRMSE 从 0.77 降至 0.71，Active MAE 从 0.12 降至 0.10。
- Motion estimation 中，BioHuman 的 pose RMSE 为 **0.30 rad**、PA-MPJPE 为 **35.21 mm**；PromptHMR 分别为 0.53 rad / 40.89 mm，HMR2.0a 为 0.55 / 41.45 mm，CLIFF 为 0.53 / 45.06 mm。
- 与结构相近但去掉 image-conditioned features 和 biomechanical-to-HMR feedback 的 two-stage ablation 比较，PCC 从 **0.58 → 0.71**，RMSE 从 0.074 → 0.065，支持 end-to-end joint representation 的贡献。

## 优点

- 把 video HMR 与 muscle activation estimation 从串联 pipeline 改造成真正的联合表示学习，研究问题比单纯 pose refinement 更接近 biomechanics 应用。
- BioHuman10M 同时连接 real/synthetic image、SMPL motion、GRF 与 full-body muscle activation，为视觉到内部 biomechanical state 提供了大规模训练接口。
- 结果表明 biomechanical supervision 没有以牺牲 pose accuracy 为代价，反而能改善 motion estimation。
- 数据生成流程、过滤条件和 test protocol 描述较清楚，包括 35,837 个独立测试 windows。

## 局限

- 作者明确指出 BioHuman10M 是 **simulation-based**，与真实 human biomechanics 存在 domain gap；目前计划用真实 EMG 做进一步生理有效性验证。
- ULBS-112 当前缺少 neck degrees of freedom，颈部运动表示不完整。
- GRF pipeline 只覆盖 foot-ground contact，不包含 hand support、seated contact 或 object interaction。
- Muscle activation 由 GaitDynamics + inverse dynamics + static optimization 间接生成，并不是真实 EMG ground truth。
- **推断：**对临床病理步态或高速体育动作，较低 SMPL/activation error 不能直接等价于真实 muscle recruitment、joint loading 或 injury-risk estimation，需要独立 EMG/force/pressure 验证。

## 个人评价

这篇的重要性在于把视觉人体研究从 `pose / mesh → CoM / GRF` 再向前推进到 `muscle activation`，给体育、康复和临床视频提供了一个新的输出层级。尤其值得关注的是 biomechanical labels 反向改善 HMR，而不是只作为 pose 后处理。

不过当前最强结论仍是在“simulation labels 上预测 simulation labels”。**推断：**真正决定其临床/体育价值的不是 PCC 0.71 本身，而是与真实 EMG、force plate、pressure/insole 的跨域一致性，以及在病理人群、个体肌骨差异和高速动作下是否仍成立。

## 与我的研究关联

**推断：**对临床 gait / ASD，可把现有链条从 `RGB/flow/keypoints → gait classification` 扩展为：

`video → 3D pose/SMPL → gait phase / trunk-pelvis kinematics → GRF/CoM → muscle activation → diagnosis/severity/explanation`。

对滑雪，可进一步把 muscle activation 与 boot/insole pressure、IMU、contact phase 对齐，检验姿态更准是否真的带来更可信的发力/负荷解释。建议优先复现：
- PromptHMR+MinT vs BioHuman 的 end-to-end / two-stage 比较；
- 不同 action / source dataset 的跨域表现；
- gait phase-conditioned muscle error；
- 与真实 EMG、pressure、GRF 的独立验证，而不是仅使用模拟 activation label。

建议重点阅读 Sec. 3 BioHuman10M、Sec. 4 BioHuman、Sec. 5.2–5.4 以及 Appendix C 的 split/metrics。

## 后续阅读

- 与 OpenCap Monocular、OpenCapBench、GRIP、BadmintonGRF 和 kinetics-aware imitation learning 统一比较 kinematics、external kinetics 与 internal muscle states。
- 关注 BioHuman10M、代码和模型是否公开；当前未核验到官方发布入口。
- 在 clinical gait 上设计 `pose-only → biomechanics-aware → muscle-aware` ablation，并加入真实 sensor/EMG validation。
