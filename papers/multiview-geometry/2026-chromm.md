---
title: "Coherent Human-Scene Reconstruction from Multi-Person Multi-View Video in a Single Pass"
authors: "Sangmin Kim, Minhyuk Hwang, Geonho Cha, Dongyoon Wee, Jaesik Park"
venue: "arXiv"
year: 2026
reading_date: 2026-09-14
status: skimmed
tags:
  - multi-view
  - human-scene-reconstruction
  - camera-geometry
  - global-human-motion
  - SMPL-X
  - multi-person
  - scale-estimation
  - feed-forward
---

# Coherent Human-Scene Reconstruction from Multi-Person Multi-View Video in a Single Pass

## 基本信息

- **作者：** Sangmin Kim, Minhyuk Hwang, Geonho Cha, Dongyoon Wee, Jaesik Park
- **会议/期刊：** arXiv:2603.12789（v1: 2026-03-13；v2: 2026-03-18）
- **年份：** 2026
- **阅读日期：** 2026-09-14
- **阅读状态：** `skimmed`
- **标签：** `multi-view`, `human-scene-reconstruction`, `camera-geometry`, `global-human-motion`, `SMPL-X`, `multi-person`, `scale-estimation`, `feed-forward`
- **论文：** https://arxiv.org/abs/2603.12789
- **DOI：** https://doi.org/10.48550/arXiv.2603.12789
- **代码：** https://github.com/nstar1125/CHROMM_RELEASE （官方仓库已建立；当前仓库仍为空，代码尚未实际发布）
- **数据集：** 暂无（论文使用 EMDB-2、RICH、EgoHumans、EgoExo4D 等既有 benchmark，没有论文专属数据集页）
- **项目主页：** https://nstar1125.github.io/chromm/

## 一句话总结

CHROMM 将 camera parameters、scene point cloud 与多人物 SMPL-X 统一到一个多视角视频框架中，并通过人体尺度校正、几何式跨视角关联和按属性类型设计的多视角融合，在不依赖外部检测/ReID/测试时优化的情况下实现较强的 world-space human-scene reconstruction。

## 研究问题与动机

现有 human-scene reconstruction 已逐渐从 monocular pipeline 走向统一模型，但多视角扩展往往仍需要外部 2D keypoint detector、bounding-box detector、cross-view ReID 或迭代优化。对于多人多视角视频，这些额外模块既增加系统复杂度，也会把 detection、association 和 camera/scene estimation 的误差逐级传递到人体世界坐标结果。

CHROMM 的目标是从 `V` 个视角、`T` 个时间步的 RGB 视频直接联合估计：(1) 每帧 camera-space point map；(2) camera parameters；(3) 多人物共享世界坐标下的 SMPL-X。作者将 Pi3X 的 scene/camera prior 与 Multi-HMR 的 human prior 放入同一框架，并特别处理 scene 与 metric human body 之间的尺度不一致、跨视角人体融合以及多人身份对应问题。

## 核心方法

### 1. 双分支 human / scene representation

每帧同时经过 Pi3X scene encoder 与 Multi-HMR human encoder。Pi3X decoder 输出 camera parameters、local point maps 和 scene feature；Multi-HMR feature 中的 head tokens 用于发现和表示人体。对应位置的 scene token 与 human token 再融合以回归 SMPL-X pose、shape、root rotation 和 3D head translation。

人体 translation 不直接回归完整 3D 向量，而是利用 scene point map 的粗 depth 作为 prior，只预测 depth residual，再结合预测的 2D head position 与 camera intrinsics 反投影。这一设计把人体放置问题显式连接到 scene geometry。

### 2. Human-aware scale adjustment

Pi3X 只提供近似 metric scene scale，而 SMPL-X 本身具有人体尺度。CHROMM 使用图像中的 2D head-pelvis 距离与投影 SMPL 的 head-pelvis 距离之比，在所有有效 frame-person 对上求平均，得到全局 scale correction，并乘到 scene scale 上。该模块等价于利用人体结构作为 scene/camera metric cue。

### 3. Multi-view fusion

作者将 human representation 分成两类：

- **View-invariant：** canonical pose、shape，跨视角直接平均；
- **View-dependent：** root rotation、3D translation，先通过预测 camera extrinsics 转到共享 world frame，再对 rotation 做 quaternion averaging；global head position则通过 multi-view ray triangulation 得到。

这种设计避免直接对混合了视角因素的 token 做统一池化。

### 4. Geometry-based multi-person association

每个视角内部先用 human token 距离做时序 tracking，并用 3D joint displacement 去除异常匹配。跨视角 association 不使用 appearance ReID，而是结合 world-space 3D position 与 canonical pose 的几何距离，通过 Hungarian matching 得到 global human IDs。论文消融表明 position cue 是主要贡献，pose cue 可处理边界情况。

## 数据集与评价指标

论文在以下 benchmark 上评价：

- **EMDB-2：** dynamic-camera monocular video，用于 global human motion；
- **RICH：** fixed multi-view setup，同时评价 monocular 与 multi-view；
- **EgoHumans、EgoExo4D：** 按既有 multi-view single-frame protocol 评价世界位置与局部姿态。

原文实验段没有重新汇总四个 benchmark 的总受试者数/总序列数，因此这里不根据二手资料补猜。运行时间实验明确使用 EgoHumans 中 **3 人 × 4 视角 × 1 个时间步**；补充实验进一步使用 **25 time steps × 4 views = 100 frames**。

Global motion 使用 **WA-MPJPE、W-MPJPE（mm）和 RTE（%）**；multi-view single-frame protocol 使用 **W-MPJPE†、GA-MPJPE、PA-MPJPE（m）**。单目评估把序列切为 100-frame segments；多视角由于显存限制以 25-frame chunks 处理，再按 camera pose 用 Sim(3) 拼成 100-frame segments。

## 主要结果

- **EMDB-2 monocular：** CHROMM 为 **102.6 mm WA-MPJPE / 255.0 mm W-MPJPE / 1.7% RTE**；Human3R 为 112.2 / 267.9 mm / 2.2%，UniSH 为 118.5 / 270.1 mm / 5.8%。
- **RICH：** monocular 为 **87.5 / 138.3 mm / 3.3%**；multi-view 融合后进一步降到 **53.1 / 79.0 mm / 1.4%**。
- **EgoHumans：** W-MPJPE† / GA-MPJPE / PA-MPJPE 为 **0.51 / 0.15 / 0.05 m**；HSfM 为 1.04 / 0.21 / 0.05 m。
- **EgoExo4D：** W-MPJPE† / PA-MPJPE 为 **0.26 / 0.06 m**；HSfM 为 0.56 / 0.06 m。
- **Scale adjustment 消融（EMDB-2）：** 去掉人体尺度校正后为 169.7 / 447.9 mm / 4.2%，加入后改善为 **102.6 / 255.0 mm / 1.7%**。
- **Multi-view fusion 消融（RICH）：** 仅平均为 69.3 / 105.7 mm / 1.7%，`Avg.+Tri.` 达 **53.1 / 79.0 mm / 1.4%**。
- **Association（EgoHumans）：** pose-only accuracy 70.6%，position-only 91.1%，二者结合为 **91.3%**。
- **Runtime：** 在单张 V100、3 人、4 视角、单时间步设置下，HSfM 约 118 s、HAMSt3R 约 32 s、CHROMM 约 **4 s**，相对 HAMSt3R 超过 8× 加速。

## 优点

- 将 scene/camera/human 统一到一个多视角视频系统中，不要求外部 bbox、2D pose、cross-view ReID 或测试时迭代优化。
- 把 human body scale 显式用于 scene metric scale correction，消融结果表明其对 world-space motion 精度影响很大。
- 将 canonical pose/shape 与 root rotation/translation 分开融合，体现了 view-invariant 与 view-dependent quantity 的几何差异。
- 跨视角身份匹配使用 3D position + pose，而不是只依赖外观，对于服装相似的体育或多人场景更有潜力。

## 局限

- 论文明确展示复杂姿态、近距离人体交互、头部完全不可见以及 extreme zoom-in 是明显 failure cases；方法又以 head token 作为人体检测和定位核心，因此 head occlusion 是结构性弱点。
- 多视角长序列仍按 25-frame chunk 处理，再通过 Sim(3) 拼接 100-frame segment；这与真正连续 online reconstruction / SLAM 还有距离，也没有长期 ATE/RPE/scale-drift 评价。
- 当前官方代码仓库已经建立，但截至阅读日尚未实际发布代码，完整复现仍受限。
- 论文主要基于 pinhole-style camera / multi-view geometry，没有针对 ERP、fisheye 或同中心 360 perspective crops 做专门验证。

## 个人评价

**价值类型：** Baseline / Method Module / Related Work  
**阅读优先级：** A+

CHROMM 对我的价值不在于“又一个 multi-view HMR”，而在于它把 **scene geometry、camera、human scale、cross-view identity 与 pose fusion** 放在同一系统里，并且给出了很清楚的 scale / fusion / association 消融。它适合作为 calibration-free / feed-forward multi-view human-scene reconstruction 的强 baseline。

需要注意，它对 camera 的使用更多是统一场景几何与跨视角坐标，而不是显式 `human residual → camera trajectory correction`。因此不能把它直接当作 camera-human mutual refinement 的终点。

## 与我的研究关联

**推断：** 对双 360° 滑雪，可以把 CHROMM 的设计拆成几个可直接验证的模块：

1. `per-view HMR → view-invariant pose/shape fusion`；
2. `camera-aware world transform → view-dependent root rotation / translation fusion`；
3. `human scale cue → camera/scene metric scale refinement`；
4. `3D position + canonical pose → cross-view identity association`；
5. 在此基础上进一步加入 `human reprojection / bone length / contact / velocity residual → camera R/t/scale correction`，形成真正的 camera-human closed loop。

对于两个物理 360 相机，应把 pinhole ray triangulation 改成 fisheye/spherical-consistent geometry，并将真实双中心相机与单个 360 相机切出的 shared-center perspective views 分开评价。建议同时报告 **W-MPJPE/RTE + camera ATE/RPE/scale drift + association accuracy**，以区分“pose fusion 变好”与“camera geometry 也被人体证据修正”。

## 后续阅读

- 重点细读 Sec. 3.1 的 scale adjustment、Sec. 3.2 multi-view fusion、Sec. 3.3 geometry-based association，以及 Sec. 4.5 三组消融。
- 与 HSfM、AHAP、TROPHIES、Spatiotemporal Multi-Camera Calibration using Freely Moving People 做统一比较。
- 待官方代码实际发布后，优先复现 `scale adjustment` 与 `Avg.+Tri.`，再替换为 spherical/fisheye geometry。
