---
title: "Driver-WM: A Driver-Centric Traffic-Conditioned Latent World Model for In-Cabin Dynamics Rollout"
authors: "Haozhuang Chi, Daosheng Qiu, Hao Su, Haochen Liu, Zirui Li, Haoruo Zhang, Chen Lv"
venue: "ECCV 2026"
year: 2026
reading_date: 2026-09-17
status: skimmed
tags:
  - driver-monitoring
  - motion-forecasting
  - world-model
  - multimodal
  - vlm
---

# Driver-WM: A Driver-Centric Traffic-Conditioned Latent World Model for In-Cabin Dynamics Rollout

## 基本信息

- **作者：** Haozhuang Chi, Daosheng Qiu, Hao Su, Haochen Liu, Zirui Li, Haoruo Zhang, Chen Lv
- **会议/期刊：** European Conference on Computer Vision (ECCV 2026), Main Conference
- **年份：** 2026
- **首次提交：** 2026-05-06（arXiv）
- **阅读日期：** 2026-09-17
- **阅读状态：** `skimmed`
- **标签：** `driver-monitoring`, `motion-forecasting`, `world-model`, `multimodal`, `vlm`
- **DOI：** https://doi.org/10.48550/arXiv.2605.05092
- **论文：** https://arxiv.org/abs/2605.05092
- **代码：** 暂无
- **数据集：** https://github.com/ydk122024/AIDE
- **项目主页：** https://fisher75.github.io/haozhuangchi.github.io/driver-wm/

## 一句话总结

Driver-WM 将驾驶员监控从当前状态识别扩展为 traffic-conditioned multi-step motion rollout：利用冻结 VLM 构造车外交通与舱内驾驶员双流 latent state，通过 gated causal injection 将外部交通上下文定向注入驾驶员动力学，并同时预测未来 2D skeleton 与行为、情绪、交通和车辆语义。

## 研究问题与动机

现有 driver monitoring 多数针对 distraction、drowsiness、emotion 或 action 的当前状态识别，而驾驶辅助系统真正需要的问题之一是：车外交通发生变化后，驾驶员接下来会怎样移动、观察或操作。传统 motion forecasting 又常只使用人体自身历史轨迹，无法显式利用同步 road context。

Driver-WM 因此把驾驶员建模为一个受外部交通条件驱动的内部动态系统。模型从同步 in-cabin / out-cabin videos 提取 frozen vision-language features，分别建立 external traffic state 与 internal driver state，再通过 directionally coupled gated causal injection 让车外状态只沿时间因果方向影响驾驶员 latent transition。预测端同时输出未来 skeleton trajectory 和多个语义任务，使几何运动与行为语义在同一个 rollout framework 中学习。

## 核心方法

### 1. Frozen VLM State Interface

每个同步时刻的 RGB frame 与固定文本 prompt 输入冻结的 Qwen-VL/Qwen3-VL 系列 encoder，提取最终层 hidden states 并池化为 frame-level feature。论文 appendix 给出的 feature dimension 为 2048。VLM 本身不参与 Driver-WM 的梯度更新，减少 end-to-end 大模型训练开销。

### 2. Dual-Stream Latent Dynamics

模型分别维护外部交通流和内部驾驶员流。AIDE 提供 front/left/right 三个 out-cabin views 与一个 in-cabin view；外部多视角先融合，再形成 traffic latent。内部 stream 则表示驾驶员当前运动与语义状态。

### 3. Gated Causal Injection

外部 traffic history 通过 learned gate 注入内部 transition，使 future driver state 的更新显式受到 road-context conditioning，同时遵守 zero-lookahead。论文还通过交换外部 clip、删除 external features、关闭或强制 injection pathway 的 test-time interventions 检查模型是否真正依赖该通路。

### 4. Unified Kinematic and Semantic Decoding

主几何任务预测 HALPE-136 2D skeleton future trajectory；辅助语义任务包括 driver behavior recognition (DBR)、driver emotion recognition (DER)、traffic context recognition (TCR) 与 vehicle condition recognition (VCR)。这种联合设计希望避免只优化平均像素误差而忽略安全相关行为语义。

## 数据集与评价指标

实验使用 AIDE assistive driving benchmark。AIDE 官方版本包含 2,898 个 3 秒样本、521.64K frames，每个样本具有 3 个 out-cabin views 和 1 个 in-cabin view，并提供 HALPE-136 keypoints 与四类语义任务标签。

Driver-WM 严格沿用官方 split，在其处理后的实验配置中得到：

- **Training：** 1,884 clips
- **Validation：** 405 clips
- **Testing：** 609 clips
- 每个 3 秒 clip 均匀采样 **10 frames（约 3.3 Hz）**，采用固定 **5→5 causal rollout**：观察前 5 帧，预测后 5 帧。
- skeleton topology 为 **136 joints**：26 body + 68 face + 42 hand joints。
- High-Motion (HM) subset 为 test set 未来窗口 joint displacement 最大的 top 10%，即 **60 / 609 clips**。

几何指标包括 MPJPE（px）、distance-normalized MPJPE（d-nMPJPE, %）和 PCK@0.05；语义指标为 DBR/DER/TCR/VCR Macro-F1。

## 主要结果

固定 5→5 rollout 下，Driver-WM main 的 **All MPJPE = 71.47 px、d-nMPJPE = 3.24%、HM MPJPE = 138.03 px、PCK@0.05 = 71.66**；对应语义 Macro-F1 为 **DBR 68.07、DER 72.61、TCR 90.15、VCR 68.34**。

MotionBERT 的 All/HM MPJPE 为 **73.51 / 141.53 px**，但没有外部交通语义；Cross-Attention-only 为 **80.14 / 142.41 px**。值得注意的是，Static Pooling 的平均 All/HM MPJPE 反而达到 **68.50 / 134.56 px**，优于 Driver-WM 的平均 L2-style geometry error。论文把这一现象解释为自然驾驶中的 inertia / mean-regression trap：大量低运动 clips 会奖励静态预测，因此额外构造 High-Motion tail 和 horizon-wise evaluation。

在 controlled intervention 中，交换外部 context 使整体 rollout 变化 ΔAll = **5.363**，移除外部 features 为 **12.953**；完全关闭 injection pathway (`lambda_CA=0`) 时 ΔAll 达 **89.641**、ΔHands 达 **95.666**。作者明确将这些实验定位为 mechanism probes，而不是严格的 causal effect estimation。

## 优点

- 将 driver monitoring 从 recognition 扩展到未来动态 rollout，任务定义与 L2/L3 shared-control 场景更接近。
- 显式区分 in-cabin 与 out-cabin dynamics，并通过 directional gated injection 建模外部交通对驾驶员未来运动的条件影响。
- 不只报告平均 skeleton error，还专门分析 High-Motion tail，揭示低动态驾驶数据中常见的 zero-velocity / mean-regression 偏差。
- 几何与语义任务共用 latent rollout，且利用 intervention 检查模型是否实际使用 traffic context，具有一定解释性。

## 局限

- 目前只在 AIDE 单一 benchmark 上验证，而且官方 AIDE split **不是 subject-held-out split**；因此跨驾驶员泛化能力仍不清楚。
- 预测对象是 2D HALPE-136 skeleton，不是 metric 3D head/body motion，也没有 vehicle-state / gaze / 3D scene geometry 的连续联合动力学。
- 平均几何指标上 Static Pooling 仍优于 Driver-WM，说明 traffic-conditioned rollout 的优势主要集中在 reactive/high-motion、语义和机制建模，而不是所有常规 clips 的纯 L2 error。
- 官方项目页目前标注 **Code Coming Soon**，完整实现尚无法直接复现。
- controlled intervention 只能说明模型对输入通路敏感，作者也明确说明不能把它解释为严格的 causal effect estimation。

## 个人评价

这篇论文最有价值的地方不是把 MPJPE 做到绝对最低，而是重新定义了 driver-motion benchmark 应该关注什么：如果数据绝大多数时间接近静止，那么只报告平均误差会让 copy-last-frame 或 static pooling 占便宜，而真正安全关键的是车外事件发生后头、手和躯干如何反应。

**推断：**对于驾驶员行为研究，可以把这套框架与显式 3D head-motion / face-mesh analysis 结合，将“未来动作预测”作为比当前 VFL classification 更接近因果行为过程的任务，同时保留 event-conditioned/high-motion subset 作为主要评价切片。

## 与我的研究关联

与三视角驾驶员 head/body motion、模拟 visual-field loss、traffic density 与 day/night 条件研究直接相关。可以把当前任务从 `condition → 当前 head-motion statistics / classification` 扩展为：

1. 输入过去窗口的 3D face mesh / head orientation / upper-body pose；
2. 加入 traffic、VFL、visibility/day-night 等 context；
3. 预测未来数秒 head rotation、scan onset、upper-body motion 或行为状态；
4. 对 high-motion / scan-onset / braking / steering event windows 单独评价。

**推断：**建议比较 `zero-velocity → motion-only Transformer → context concatenation → Driver-WM-style gated injection → 3D head/body world model`，并同时报告平均误差、event-window error、scan detection F1、calibration 和不同 VFL 条件下的泛化。

## 后续阅读

- DriveMotion：用于对比大规模 driver motion forecasting benchmark 与 event-anchored evaluation。
- AIDE 原始论文：进一步核验四任务标签和不同 camera streams。
- Risk-Aware Selective Multimodal Driver Monitoring：对比当前状态识别、风险选择与 future rollout 的不同目标。
- 后续实验重点：subject-held-out split、3D head/body trajectory、event-conditioned evaluation，以及外部 traffic context 对未来 compensatory head scanning 的真实增益。