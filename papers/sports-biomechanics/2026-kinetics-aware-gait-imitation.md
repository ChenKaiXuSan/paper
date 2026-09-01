---
title: "Imitation Learning from Human Motion Alone Does Not Guarantee Biomechanically Plausible Gait Kinetics"
authors: "Xinyi Liu, Jangwhan Ahn, Edgar Lobaton, Jennie Si, He Huang"
venue: "arXiv"
year: 2026
reading_date: 2026-09-02
status: skimmed
tags:
  - biomechanics
  - gait
  - imitation-learning
  - kinetics
  - ground-reaction-force
  - center-of-pressure
---

# Imitation Learning from Human Motion Alone Does Not Guarantee Biomechanically Plausible Gait Kinetics

## 基本信息

- **作者：** Xinyi Liu, Jangwhan Ahn, Edgar Lobaton, Jennie Si, He Huang
- **会议/期刊：** arXiv:2603.12408
- **年份：** 2026
- **阅读日期：** 2026-09-02
- **阅读状态：** `skimmed`
- **标签：** `biomechanics`, `gait`, `imitation-learning`, `kinetics`, `ground-reaction-force`, `center-of-pressure`
- **论文：** https://arxiv.org/abs/2603.12408
- **DOI：** https://doi.org/10.48550/arXiv.2603.12408
- **代码：** 暂无
- **数据集：** 暂无；实验使用作者采集的单受试者 treadmill gait motion/force 数据
- **项目主页：** 暂无

## 一句话总结

该工作证明“动作看起来像人”并不等于“动力学也像人”：仅模仿人体运动的策略可以得到不错的关节角轨迹，却产生不合理的 GRF、CoP 和关节力矩；显式加入 GRF 与 CoP 奖励后，动力学真实性显著提高。

## 研究问题与动机

Physics-based imitation learning 常用人体 motion capture 作为主要监督，让仿真人体追踪关节角、身体姿态和质心运动。一个隐含假设是：如果运动学轨迹足够接近真人，那么 ground reaction force、center of pressure 和 joint moments 等 kinetics 也会自然接近真实人体。

论文直接检验这个假设，并给出否定结果。作者比较 motion-only imitation learning（MOIL）与加入显式 kinetics supervision 的策略，提出 Kinetics-Aware Imitation Learning（KAIL），在保持 gait kinematics 的同时引入 GRF 与 CoP matching rewards，使 learned control policy 不只“动作像”，还要在力学上更接近人体。

## 核心方法

### 人体动力学模型

作者在 MuJoCo 中建立 floating-base lower-limb model：

- trunk 为 floating base；
- 每侧 hip 3 DoF、knee 1 DoF、ankle 3 DoF；
- 共 17 个 joint coordinates；
- 足部用 box geometry 表示，每只脚包含 4 个 contact points；
- 使用 residual wrench 辅助处理模型误差。

### Imitation learning

策略使用 PPO 训练，actor 为两层 MLP（512 / 256 hidden units，ReLU），输出 Gaussian action distribution。仿真频率 450 Hz，policy control interval 为 1/30 s。

所有条件先进行 700 iterations 的 kinematic-only pretraining，然后再训练 300 iterations：

1. **MOIL**：只保留 motion / kinematic rewards；
2. **GRF Match**：增加 ground reaction force reward；
3. **CoP Match**：增加 center-of-pressure reward；
4. **KAIL**：同时加入 GRF 与 CoP rewards。

四种条件共享相同初始化，用于尽量隔离 kinetics reward 的贡献。

## 数据集与评价指标

实验数据来自 **1 名健康年轻男性**：

- 29 岁；
- 身高 176 cm；
- 体重 70 kg；
- treadmill walking speeds：0.9、1.2、1.5 m/s；
- 每个速度约 2 min；
- 42 reflective markers；
- Vicon motion capture：120 Hz；
- Bertec force-instrumented treadmill：1000 Hz。

每个速度的前 10 s 被丢弃；随后 20 个 strides 用于训练，再取后续 15 个 strides 测试。

评价包括：joint-angle RMSE、CoM velocity RMSE、GRF RMSE、CoP RMSE、joint-moment RMSE，以及 predicted 与 inverse-dynamics joint moments 的 Pearson correlation。

## 主要结果

### Kinematics

三种速度平均 joint-angle RMSE：

- MOIL：3.66°；
- GRF Match：4.40°；
- CoP Match：4.33°；
- KAIL：4.02°。

因此加入 kinetics constraints 并没有让关节角误差进一步下降，甚至略高于 MOIL；CoM velocity RMSE 对 MOIL 与 KAIL 都约为 0.17 m/s。

### Ground reaction force 与 CoP

相较 MOIL 的 total GRF RMSE：

- GRF Match 降低约 38%；
- CoP Match 降低约 41%；
- KAIL 降低约 54%。

CoP RMSE：

- CoP Match 降低约 52%；
- KAIL 降低约 45%；
- 仅增加 GRF reward 并没有同步降低 CoP error。

### Joint moments

KAIL 相比 MOIL 的 joint-moment RMSE：

- hip：降低约 40%；
- knee：降低约 34%；
- ankle：降低约 6%。

对应 Pearson correlation 分别提高约 0.36、0.38 和 0.08。

这些结果直接说明：相似的 kinematics 并不能自动保证相似的 kinetics；需要显式动力学信息才能可靠约束地面反力、压力中心与关节力矩。

## 优点

- 研究问题非常清楚，直接挑战“motion imitation 足以隐式恢复 biomechanics”这一常见假设。
- 把 kinematic fidelity 与 kinetic fidelity 分开报告，避免只凭 joint-angle / pose error 得出过度结论。
- 使用 GRF-only、CoP-only 与 GRF+CoP 三种配置，能看出不同动力学监督并非互相可替代。
- 对体育/临床视频研究具有方法论价值：视觉姿态准确不等于载荷、接触和关节力矩准确。

## 局限

- 核心验证只有 **1 名健康受试者**，且仅包含三种水平 treadmill walking speed，外部泛化证据非常有限。
- 仿真模型只覆盖 lower limb，足部采用简化 box geometry，没有 MTP 等更细致 foot mechanics，contact model 与真实人体存在偏差。
- 控制器与 residual wrench 是工程化建模，并不等价于真实 muscle-actuation mechanism。
- KAIL 虽显著改善 kinetics，但 joint-angle RMSE 从 MOIL 的 3.66° 增至 4.02°，说明运动学和动力学目标之间存在真实 trade-off。
- GRF / CoP supervision 依赖 force-instrumented treadmill。**推断：**因此本文不能证明“纯视频就能可靠估计真实 kinetics”，反而提示需要额外 contact / pressure / force evidence 或非常谨慎的物理先验。
- 本次未从作者的一手来源核验到官方代码、公开数据或项目主页。

## 个人评价

这篇的主要价值不是提出一个可直接部署的 gait model，而是提供一个很强的“评价原则”：如果研究声称从视频或 pose 推断 biomechanics，仅报告 MPJPE、joint-angle error 或 trajectory similarity 是不够的，必须独立验证 GRF、CoP、joint moments 或至少可观测的 contact/load proxy。

同样重要的是，KAIL 并没有在所有指标上同时变好。相比“统一追求最低误差”，更合理的做法是明确哪些指标描述几何/运动学，哪些指标描述动力学，并接受不同目标之间存在 Pareto trade-off。

## 与我的研究关联

对滑雪与临床 gait 都很直接。

在滑雪中，**推断：**可以把现有 world 3D pose / trajectory pipeline 扩展为 `motion-only → contact-aware → pressure/IMU-aware → kinetics-aware` 的递进实验。若能获得 boot pressure、insole pressure、IMU 或 ski-ground contact proxy，就可以检验“更准确的 3D motion 是否真的意味着更合理的外部载荷”。

在临床 gait 中，可把 pose-based diagnosis 与 constrained kinematics / kinetics 分开评估，尤其避免从 monocular pose 直接过度解释 joint moment、GRF 或 loading asymmetry。与 OpenCap Monocular、BadmintonGRF、GRIP 等工作一起，可以形成从 motion capture 到 kinetics validation 的完整 Related Work 路线。

## 后续阅读

- 与 OpenCap Monocular 比较：一个从视频推导 kinematics/kinetics，一个从控制策略角度证明 motion-only 不能保证 kinetics。
- 与 GRIP、BadmintonGRF 比较 pressure/contact/force supervision 的形式和评价指标。
- 在 sports motion 中增加 `pose-only vs contact-aware vs pressure/force-aware` ablation，同时报告 kinematic 与 kinetic metrics。
- 研究没有 force plate 时，boot pressure、IMU、contact timing、ski deformation 等 proxy 能否提供足够的 kinetics supervision。
