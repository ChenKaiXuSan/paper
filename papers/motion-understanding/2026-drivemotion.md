---
title: "DriveMotion: A Large-Scale Multi-Source Benchmark for Driver Motion Sequence Modeling and Forecasting"
authors: "Yuhang Wang, Chuheng Wei, Jingxin Yang, Xishun Liao, Hao Zhou"
venue: "arXiv:2609.08117"
year: 2026
reading_date: 2026-09-10
status: skimmed
tags:
  - motion-understanding
  - driver-monitoring
  - motion-forecasting
  - skeleton
  - autonomous-driving
  - multimodal
---

# DriveMotion: A Large-Scale Multi-Source Benchmark for Driver Motion Sequence Modeling and Forecasting

## 基本信息

- **作者：** Yuhang Wang, Chuheng Wei, Jingxin Yang, Xishun Liao, Hao Zhou
- **会议/期刊：** arXiv:2609.08117
- **年份：** 2026
- **提交日期：** 2026-09-08
- **阅读日期：** 2026-09-10
- **阅读状态：** `skimmed`
- **标签：** `motion-understanding`, `driver-monitoring`, `motion-forecasting`, `skeleton`, `autonomous-driving`, `multimodal`
- **DOI：** 10.48550/arXiv.2609.08117
- **论文：** https://arxiv.org/abs/2609.08117
- **代码：** https://huggingface.co/datasets/HenryYHW/DriveMotion （官方 dataset repository 内含 `code/` reference implementation；数据仍在分阶段上传）
- **数据集：** https://huggingface.co/datasets/HenryYHW/DriveMotion
- **项目主页：** 暂无

## 一句话总结

DriveMotion 将自然驾驶、公开车内视频与 AIDE 统一为大规模 133-keypoint 驾驶员运动序列，并提出以车辆操纵事件为锚点的 8 秒观测→4 秒人体运动预测 benchmark，避免普通随机采样被“驾驶员大部分时间不动”的 persistence baseline 主导。

## 研究问题与动机

现有 driver-monitoring 数据集多关注短片段动作分类，而通用 human-motion forecasting benchmark 通常来自实验室、行走或室外全身动作，并不反映驾驶舱内“长时间稳定 + 短时关键动作”的分布。

如果均匀采样自然驾驶，预测模型只输出“保持不动”也可能获得较好的平均误差，从而掩盖方向盘操作、头部扫描、伸手等真正与驾驶状态和近期意图相关的运动。DriveMotion 因此同时解决数据规模、跨来源表示统一和 evaluation sampling 三个问题。

## 核心方法

### 1. Multi-source skeleton corpus

数据整合三类来源，并统一经过 RTMW whole-body pose extraction、10 Hz 重采样、坐标归一化、joint validity 与 quality scoring：

- BATON fleet：长时间自然驾驶、CAN、device-grounded head pose、road-view context；
- Web corpus：不同视角、遮挡和驾驶舱外观；
- AIDE：带行为/情绪语义标注的车内片段。

每帧核心表示为 133 个 COCO-WholeBody keypoints `(x, y, score)`，另提供 per-joint mask、head yaw/pitch/roll、part-validity 与 provenance。BATON 还含 speed / steering / throttle / brake CAN。

### 2. Forecasting protocol

任务是在 canonical torso frame 中观察 **8 s** 驾驶员运动，预测后续 **4 s** 的 keypoint trajectories 与 head pose，频率为 10 Hz。road-view 的 frozen ResNet-50 exterior features（2 Hz）可作为可选 context。

CAN **不作为 inference input**，仅离线挖掘 vehicle-dynamics transitions，用来构建 pre-maneuver、post-maneuver 与 matched stable-control windows，从而把测试重点放在真正发生运动变化的时段。

### 3. Two-level evaluation

- **MPJPE@4s：**评价 23 个跨视角相对稳定关键点的几何预测误差。
- **Part-State F1@2s：**判断 head / torso / arms 在未来是 still、small motion 还是 large motion，用于补充“坐标误差小但行为状态判断错”的情况。

## 数据集与评价指标

- **总受试者/驾驶员：**360 drivers。
- **论文摘要规模：**393 h、133-keypoint motion、10 Hz；官方 dataset card 将规模概括为约 **400 h**。
- **官方 card：**9,010 sequences、680,082 forecasting windows。
- **来源构成：**BATON 1,347 routes / 320 h；Web 4,765 spans / 78 h；AIDE 2,898 clips / 2.4 h。
- **事件规模：**76,026 maneuver initiations 由 CAN 离线挖掘。
- **dynamics-anchored test：**pre-maneuver 6,138、post-maneuver 6,238、stable-control 4,262，共 **16,638 windows**。
- **输入：**8 s skeleton + head history；可选 exterior context。
- **输出：**未来 4 s 2D normalized keypoint trajectories + head pose。
- **主要指标：**MPJPE@4s、Part-State F1@2s。官方 card 没有为 MPJPE 表中数值声明物理单位，因此这里保留原始数值，不自行解释为 mm/cm。
- **baseline：**zero-motion persistence、GRU seq2seq、siMLPe、Transformer encoder-decoder、Transformer-L/XL、CVAE、DDPM、autoregressive motion-token LM、Llama-3B backbone。

## 主要结果

- 预操纵窗口中的 arm motion 是 route-matched stable-driving control 的 **3.4×**，说明事件锚定确实筛出了更有预测价值的时段。
- Zero-motion persistence：MPJPE@4s **7.75**、Part-State F1 **0.215**。
- Transformer-XL：MPJPE **6.62**、F1 **0.284**；相对 persistence 的几何误差下降约 14.6%，与论文“最高约 15%”的总结一致。
- Transformer ED (+context, maneuver-enriched)：MPJPE **6.95**、F1 **0.309**，其 F1 相对 zero-motion 约提升 **44%**。
- AR motion-token LM (enriched)：MPJPE **8.16**，但 F1 达到 **0.458**；Llama-3B 为 8.24 / 0.457，显示最低 coordinate error 与最佳 future part-state anticipation 并非同一模型家族。
- 论文摘要报告：用完整 multi-source corpus 训练，相比 BATON-only，在 held-out web drivers 上 forecast error 降低 **38%**。

## 优点

- 规模大、来源异质，并使用 identity-disjoint split，适合研究跨驾驶员、跨 camera view 与跨可见性泛化。
- 显式 per-joint validity mask，不把 occlusion 简单插值成“可信关节”。
- dynamics-anchored evaluation 针对自然驾驶的强静止偏置，评价问题定义比均匀随机采样更贴近实际驾驶事件。
- 同时报告几何误差与运动状态 F1，揭示“坐标最接近”与“行为趋势判断最准”的差异。
- 官方 dataset repository 同时提供 benchmark definitions 与 reference implementation，便于复现。

## 局限

- 133-keypoint motion 来自 RTMW 自动提取，并非 optical-mocap / 3D ground truth；会继承 2D pose estimator 的遮挡、视角与 domain bias。
- 发布的主要视觉表示是 privacy-reduced skeleton motion，而不是统一可用的原始 RGB appearance；因此不适合直接验证 RGB/face/hand appearance cues 的贡献。
- MPJPE 在 normalized 2D representation 上评价，不能替代 3D head/body kinematics、biomechanics 或真实空间运动误差。
- CAN 用于离线选择 evaluation windows 虽不会泄漏到 inference，但 benchmark 关注的是 vehicle-dynamics transition 周围的动作，不等同于开放世界中所有 driver intent / hazard event。
- 官方仓库当前注明 full data payload 仍在分阶段上传，立即完整复现可能受数据可用性影响。
- **推断：**多源统一表示可提升泛化，但不同来源的 camera viewpoint、pose-extraction quality 与语义标签分布仍可能形成 source shortcut，需要 source-held-out / view-held-out 分析。

## 个人评价

这篇的最大价值不是提出某个特别强的 forecaster，而是把驾驶员运动建模从“看当前是什么动作”推进到“未来几秒身体会如何变化”，同时设计了能抵抗静止分布偏置的 evaluation protocol。

对驾驶员行为研究，尤其值得借鉴 dynamics-anchored evaluation、joint validity mask 和 geometry-vs-state 双指标。它也说明仅报告平均 MPJPE 可能奖励保守预测，而遗漏真正与驾驶操纵相关的短暂运动。

## 与我的研究关联

现有三视角驾驶员数据可建立 `current-state classification → future motion forecasting → future risk/state prediction` 的扩展路线。**推断：**可把 face-mesh head motion、3D head orientation、upper-body 3D pose 与 VFL / traffic / day-night condition 作为结构化 context，比较 `motion-only → + visual-field condition → + traffic context → + multi-view 3D representation`。

特别值得迁移的是 event-anchored protocol：例如围绕方向盘操作、lane-change / braking event 或明显 head-scan onset 建立 pre/post/stable windows，并分别报告 head amplitude/frequency、future pose error 与 state F1。

**推断：**同样的评价思想也可迁移到临床 gait：围绕 heel-strike、turn initiation 或异常步态 episode 做 event-anchored temporal evaluation，避免长时间重复周期把短暂病理变化稀释在平均指标中。

## 后续阅读

- Risk-Aware Selective Multimodal Driver Monitoring：从 current driver state 分类进一步考虑 uncertainty / abstention。
- NextMotionQA：细粒度 body-part / direction / action semantics 的 motion understanding 评价。
- Universal Skeleton：跨 skeleton layout 的统一动作表征，对 DriveMotion 133-keypoint 与其他驾驶员 skeleton 的迁移有参考价值。
- **计划实验：**先在现有驾驶员数据构造 8 s→4 s forecasting baseline，并比较均匀窗口与 maneuver/head-scan anchored 窗口下 persistence、GRU、Transformer 的排名是否发生变化。
