---
title: "Everybody Tracking Every Body"
authors: "Daeyun Shin, Yunhan Zhao, Shu Kong, Alexander C. Berg, Charless Fowlkes"
venue: "arXiv:2608.29927"
year: 2026
reading_date: 2026-09-05
status: skimmed
tags:
  - global-human-motion
  - egocentric
  - multi-camera
  - visual-inertial-odometry
  - diffusion
  - multi-person
---

# Everybody Tracking Every Body

## 基本信息

- **作者：** Daeyun Shin, Yunhan Zhao, Shu Kong, Alexander C. Berg, Charless Fowlkes
- **会议/期刊：** arXiv preprint, arXiv:2608.29927
- **年份：** 2026
- **提交日期：** 2026-08-30
- **阅读日期：** 2026-09-05
- **阅读状态：** `skimmed`
- **标签：** `global-human-motion`, `egocentric`, `multi-camera`, `visual-inertial-odometry`, `diffusion`, `multi-person`
- **DOI：** 10.48550/arXiv.2608.29927
- **论文：** https://arxiv.org/abs/2608.29927
- **代码：** 暂无
- **数据集：** 暂无（论文使用 AMASS、EgoHumans、Harmony4D；未发布新的专用数据集）
- **项目主页：** https://vision.ics.uci.edu/papers/everybody-tracking-every-body-2026/

## 一句话总结

该工作利用多人各自佩戴的移动相机，把 VIO 提供的连续头部/相机世界轨迹与其他佩戴者提供的稀疏、噪声较大的 exocentric 人体观测，通过可靠性感知的 conditional diffusion 融合为统一世界坐标下的全身 SMPL-H 运动轨迹。

## 研究问题与动机

多人的 egocentric 相机网络具有一个天然的 ego–exo 互补结构：佩戴者自己的身体几乎不在本机视野中，但 VIO 可以持续给出头部所在相机的世界轨迹；与此同时，其他人的相机会间歇性地看到该佩戴者，从而获得更准确的局部人体姿态，但存在出视野、遮挡、单目尺度和观测噪声问题。论文统计中约 24% 的 person-frame 没有任何 exocentric 观测，因此仅依靠视觉追踪无法保持全时段覆盖；而仅依靠头部运动先验又很难约束四肢细节。作者的目标是学习如何根据观测存在性和可靠性自动融合这两类互补信号，而不是把视觉估计当作硬约束或简单插值。

## 核心方法

论文假设场景中有 2–4 名参与者，每人佩戴 Meta Aria，相机 RGB、IMU/VIO 轨迹和人体估计均同步到共享世界坐标。人体采用 SMPL-H 表示，并以 Central Pupil Frame（CPF）作为从头部相机轨迹连接到身体 root motion 的运动学锚点。对于目标人物，其他佩戴者视角中的人体由 CoMotion 等第三人称模型估计，再利用两台相机的相对变换映射到目标人物的参考坐标。

核心 fusion model 是约 54M 参数的 DiT conditional denoiser：noisy body trajectory 与每个 observer 的 exocentric body observation 经过共享 body encoder 和 temporal self-attention；不同 observer 通过 attentional pooling 合成为可变数量的观测表示，并以 additive residual 注入 DiT；目标人物自身的 VIO ego trajectory 单独编码，并通过 cross-attention 在各层提供全局位置约束。训练时随机丢弃观测，使模型能同时处理从完全缺失到高覆盖的情况。作者还与 RePaint、CondMDI 和 diffusion posterior sampling 等条件生成/补全方案进行对比。

## 数据集与评价指标

- **AMASS：** 用于学习大规模单人 SMPL-H motion prior，并从人体 mesh 合成 CPF/egocentric device trajectory。
- **EgoHumans：** fully-egocentric multi-camera setting，每个场景 3–4 人；使用 RGB、VIO camera trajectories，以及通过 multi-view triangulation/optimization 获得的 pseudo-GT SMPL。论文的 headline test 为 `n=1869`。
- **Harmony4D：** 208 个双人近距离互动序列、6 个活动类别、约 115k frames；其中 118 个序列为双方均佩戴 Aria 的 egocentric 数据。Harmony4D-ego test 为 `n=496`。
- 训练 motion prior 60k steps，global batch size 2048，使用 8×NVIDIA RTX PRO 6000；conditional fine-tuning 20k steps。
- **指标：** world-coordinate MPJPE、WA-MPJPE、PA-MPJPE、MPJAE，以及 temporal MPJVE 和 acceleration error。

## 主要结果

在 EgoHumans test 上，ego-only motion prior 的 MPJPE / PA-MPJPE 为 111.9 / 82.6 mm，Exocentric Video（CoMotion+SLAM）为 130.3 / 50.9 mm，体现二者分别擅长绝对位置和相对姿态。完整 fusion 达到 **90.7 mm MPJPE、78.6 mm WA-MPJPE、45.9 mm PA-MPJPE、19.7° MPJAE**，优于 RePaint、CondMDI 和 DPS 的主要位置/姿态指标。在 Harmony4D-ego 的 single-observer setting 中，完整模型达到 **73.4 mm MPJPE、67.8 mm WA-MPJPE、40.9 mm PA-MPJPE、15.9° MPJAE**。

论文进一步按 128-frame（约 4.3 s）窗口分析 observation fraction。18,339 个 subject-window pairs 中，59% 的窗口 exocentric coverage 高于 0.9，median 为 0.97，而低于 0.1 的窗口仅 4.4%。作者因此强调 aggregate metric 受高覆盖场景影响较大，但 learned fusion 在不同 coverage 区间仍表现出比单一 ego/exo source 更稳定的协同增益，而不是简单在两个 baseline 之间插值。

## 优点

- 把多名佩戴者的**多个动态相机**统一到一个 world-coordinate human motion fusion 问题中，比固定多相机或单 moving camera 更接近 distributed wearable sensing。
- 明确区分 ego trajectory 的高全局覆盖与 exocentric pose observation 的高局部姿态质量，并让网络学习观测噪声和可靠性。
- 对 missing observations、观测覆盖率和多个 fusion baseline 做了较系统的分析，而不只报告一个总 MPJPE。
- 输出同时覆盖 world position、relative pose 和 temporal motion quality，评价维度较完整。

## 局限

- EgoHumans 的 SMPL ground truth 本身来自 multi-view triangulation/optimization，因此不是完全独立的高精度 mocap GT；真实数据规模和 device configuration 也仍有限。
- 论文明确指出当前每个人的 body state 独立估计，人与人之间只通过 mutual visibility 间接耦合，没有显式 collision、contact 或 social interaction constraint。
- 当前验证集中高 observation coverage 的窗口占比很高，因此极低可见率和长时间完全不可见情况下的可靠性证据仍较弱。
- **推断：** VIO camera trajectories 在该框架中作为固定 conditioning 输入，模型并不反向优化相机轨迹。因此它解决的是“多个 moving cameras 如何帮助 human reconstruction”，而尚未完成 `human → camera` 的闭环 mutual refinement；VIO drift 若存在仍可能传递到 world human motion。

## 个人评价

这篇非常适合作为当前 moving-camera world HMR 的新强 baseline，因为它展示了一个比单相机 HMR 更接近“distributed moving-camera fusion”的设置：不同移动相机之间的观测既不连续，也不能假设同等可靠。尤其值得借鉴的是 observer-wise temporal encoding + attentional pooling + observation dropout，这比简单平均/concatenate 多视角 pose 更符合实际户外采集。

另一方面，它目前仍把 VIO 当作可信世界锚点，因此不能直接证明人体约束会改善 camera ATE/RPE。对 camera-human mutual refinement 来说，真正有价值的下一步是把 fusion residual、人体尺度、contact 或跨视角一致性继续反馈到 camera trajectory，而不只是从 camera 到 human 的单向条件化。

## 与我的研究关联

对于双 360° / 多视角滑雪，可以把每台物理移动相机或从 360° 生成的有效观测视为 reliability-varying observer。建议建立 `camera-only VIO/SLAM → independent HMR → confidence-aware multi-observer fusion → human-assisted camera correction → joint refinement` 的递进实验，并同时报告 camera ATE/RPE、W-MPJPE/RTE、跨视角一致性以及 visibility-conditioned performance。

**推断：**如果在雪地低纹理、强旋转时 VIO 置信度降低，可以进一步把 camera confidence 与 body observation confidence 放到同一个 fusion module 中，让人体的可观测尺度、骨长和接触约束反向修正相机；这样可以从本文的 ego–exo fusion 自然推进到真正的 camera–human mutual refinement。

建议重点阅读 **Sec. 3.1–3.2** 的 conditional fusion、**Sec. 4.4–4.6** 的 source reliability / coverage analysis，以及 limitations 中关于 interaction 与 data diversity 的讨论。

## 后续阅读

- EgoAllo：基于头部/设备轨迹恢复 ego wearer 全身运动。
- EgoHumans：多人全 egocentric multi-camera benchmark。
- CoMotion：exocentric multi-person 3D motion estimation。
- JOSH / DuoMo / WHAC：分别比较 joint optimization、camera/world motion prior 和 human-assisted camera scale 的耦合方式。
- 进一步实验：按 observer 数量、visibility、VIO drift 和 camera confidence 分层评估，并测试 human constraints 是否能降低 camera ATE/RPE。
