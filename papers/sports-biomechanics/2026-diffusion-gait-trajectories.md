---
title: "Diffusion-Based Generation of Gait Trajectories"
authors: "Damián Benasco, Juan Carballeira-Lopez, Jaime Ramos-Rojas, Julio S. Lora-Millan, Antonio J. Del-Ama, David Rodriguez-Cianca, Pablo Lanillos"
venue: "arXiv:2609.14642 (comments: ICNR 2026)"
year: 2026
reading_date: 2026-09-18
status: skimmed
tags:
  - gait
  - biomechanics
  - diffusion
  - rehabilitation
  - wearable-robotics
  - motion-generation
---

# Diffusion-Based Generation of Gait Trajectories

## 基本信息

- **作者：** Damián Benasco, Juan Carballeira-Lopez, Jaime Ramos-Rojas, Julio S. Lora-Millan, Antonio J. Del-Ama, David Rodriguez-Cianca, Pablo Lanillos
- **会议/期刊：** arXiv:2609.14642；arXiv comments 标注 ICNR 2026（2026-09-29 至 2026-10-02，Seoul）
- **年份：** 2026
- **阅读日期：** 2026-09-18
- **阅读状态：** `skimmed`
- **标签：** `gait`, `biomechanics`, `diffusion`, `rehabilitation`, `wearable-robotics`, `motion-generation`
- **论文：** https://arxiv.org/abs/2609.14642
- **DOI：** https://doi.org/10.48550/arXiv.2609.14642
- **代码：** 暂无
- **数据集：** 暂无
- **项目主页：** 暂无

## 一句话总结

该工作用带周期性、速度平滑和 forward-kinematics 约束的 conditional diffusion / DiT 生成下肢关节角步态轨迹，并用 step length 做可控条件，证明 diffusion 能生成具有一定生物力学合理性的周期 gait，但也暴露出仅靠 step length 无法覆盖临床步态异质性的问题。

## 研究问题与动机

下肢外骨骼和康复机器人需要能够适应个体形态、病理状态和治疗目标的 reference gait trajectories。传统 hand-crafted template 或逐目标优化很难在不同受试者与步态条件之间连续扩展。论文因此探索 diffusion-based gait synthesis，希望直接从真实 VICON gait cycles 学习关节角分布，并通过物理/步态参数控制生成结果。

当前版本只使用健康年轻成人，并把 step length 分为 short / medium 两类，因此更准确地说，这是面向 rehabilitation / wearable robotics 的**可控步态轨迹生成 proof-of-concept**，而不是病理 gait generator。

## 核心方法

1. **Base Transformer diffusion**：把每个时间帧作为 token，用 gait-phase pulse / event marker 等 biomechanical keypoints 作为通道条件，采用 500-step 线性 noise schedule，预测 clean trajectory。
2. **Controllable DiT**：通过 AdaLN-Zero 将 timestep 与 step-length condition 注入每一层；训练时以 `p=0.25` 使用 learnable null embedding，从而支持 classifier-free guidance。
3. **Biomechanical training losses**：总损失包含 MSE、cyclicity、velocity smoothness 和 forward-kinematics foot-trajectory term，权重分别为 2.0、0.5、0.1、0.1；FK 项使用 curriculum ramp-up。
4. **评价不仅看 waveform**：除 Pearson R / NRMSE 外，还报告关节 ROM、左右 Symmetry Index 以及 per-subject correlation，以检查可控生成是否保持基本生物力学性质。

## 数据集与评价指标

- **4,590 个 gait cycles，22 名健康受试者**：12 male / 10 female，年龄 `26.5 ± 7.7` 岁，身高 `172.1 ± 7.9 cm`，体重 `68.5 ± 9.1 kg`。
- 每个 gait cycle 包含 **21 个 lower-limb joint angles**（degree）以及从 VICON motion capture 得到的 biomechanical keypoints。
- 通过 forward kinematics 计算 step length，并设置 4 cm exclusion gap：**1,756 个 short steps <48 cm**，**2,834 个 medium steps >52 cm**。
- 指标：Pearson correlation `R`、NRMSE、per-joint ROM、bilateral Symmetry Index、per-subject `R`。

## 主要结果

- Base model 六个 sagittal hip/knee/ankle joints 的平均 **R=0.777，NRMSE=0.783**，但生成 step length 集中在 `57.4 ± 3.7 cm`，体现向总体均值收缩。
- Short-step DiT 的平均 **R=0.813，NRMSE=1.225**，生成 step length 为 **36.38 ± 6.43 cm**；虽然成功把步长推向 short class，但误差明显增大。
- Medium-step DiT 达到平均 **R=0.838，NRMSE=0.920**，step length 为 **68.58 ± 6.95 cm**；其 sagittal ROM error 控制在约 `±6°` 内。
- Short class 对 knee ROM 系统性低估约 **10°**、hip 约 **6°**，hip/knee Symmetry Index 约 **16–17%**，作者将其归因于 short-stride 更高的 inter-subject variability 与 MSE-driven regression-to-mean。

## 优点

- 将 diffusion motion generation 与 gait-specific cyclicity、smoothness、forward kinematics 显式结合，而不是把人体 motion 当成普通时间序列。
- 用 ROM 与 bilateral symmetry 补充 R/NRMSE，评价更贴近 gait biomechanics。
- Short-step 失败结果具有研究价值：它说明“条件值被控制住”并不等于生成运动在个体层面生物力学可信。
- 论文明确指出 step length 单一条件无法区分 slow shuffle 与 fast short stride，避免过度声称临床适用性。

## 局限

- 只有 **22 名健康受试者**，没有 Parkinson's、ASD、OA、stroke 等病理 gait，因此不能直接推断临床病态生成能力。
- 条件变量只有离散 short/medium step length；cadence、walking speed、body dimensions、disease severity、asymmetry 等关键因素未联合建模。
- Short class 的 NRMSE 反而高于 base model，且 ROM / symmetry 明显退化，说明当前 controllability 与 biomechanical fidelity 存在 trade-off。
- 主要结果集中在 sagittal hip/knee/ankle，尚不足以覆盖 3D pelvis/trunk 与多平面病理步态。
- 官方代码、数据集下载和项目主页当前均未核验到。

## 个人评价

这篇的核心价值是给“周期运动生成”提供了一个非常清楚的 loss / metric 组合：不仅要求轨迹接近真实，还要求 cyclicity、速度平滑、足部 FK、ROM 和 bilateral symmetry 合理。它与临床分类工作不同，更接近“如何生成一个受控、可解释的 gait reference”。

**推断：**如果把 step length 扩展为 `speed + cadence + phase + trunk/pelvis kinematics + asymmetry + clinical severity`，并引入病理 cohort，就可以用于临床数据增强、counterfactual gait generation 或康复目标轨迹生成；但必须用真实 subject-disjoint test 验证生成数据是否提升 downstream diagnosis，而不能只看 trajectory similarity。

## 与我的研究关联

- 临床步态：可借鉴 cyclicity / velocity / FK / symmetry objective，为 ASD 或其他脊柱疾病的周期动作 representation 增加显式 biomechanical regularization。
- PhaseMix / periodic motion：可以比较 frequency/phase-aware representation 与 diffusion periodicity loss 在病理步态建模中的互补性。
- 数据增强：**推断**可构造 `real only → generic temporal augmentation → GAITGen-style pathology disentanglement → multi-parameter diffusion gait generation`，观察 severe / rare class recall 是否真正改善。
- 体育/康复：可把 reference trajectory 生成扩展到滑雪 turn phase、joint ROM 或左右负荷目标，但需要 3D、多平面与接触/力学条件。

## 后续阅读

- 与 GAITGen 对照：一个强调 pathology-factor disentanglement，一个强调 physical gait-parameter control。
- 与 GaitEncoder / gait foundation model 对照，研究 representation learning 与 controllable generation 是否可以共享 latent space。
- 后续优先关注多参数连续 conditioning、病理 gait cohort、3D pelvis/trunk、GRF/contact 以及生成数据对 downstream clinical task 的增益。