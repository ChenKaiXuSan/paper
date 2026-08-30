---
title: "MASt3R-SLAM: Real-Time Dense SLAM with 3D Reconstruction Priors"
authors: "Riku Murai, Eric Dexheimer, Andrew J. Davison"
venue: "CVPR 2025"
year: 2025
reading_date: 2026-08-31
status: skimmed
tags:
  - multiview-geometry
  - slam
  - camera-trajectory
  - dense-reconstruction
  - mast3r
  - moving-camera
---

# MASt3R-SLAM: Real-Time Dense SLAM with 3D Reconstruction Priors

## 基本信息

- **作者：** Riku Murai, Eric Dexheimer, Andrew J. Davison
- **会议/期刊：** CVPR 2025（Highlight；项目页标注 Best Demo Honorable Mention）
- **年份：** 2025
- **DOI：** 10.48550/arXiv.2412.12392
- **阅读日期：** 2026-08-31
- **阅读状态：** `skimmed`
- **标签：** `multiview-geometry`, `slam`, `camera-trajectory`, `dense-reconstruction`, `mast3r`, `moving-camera`
- **价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** A+（最高）
- **论文：** https://arxiv.org/abs/2412.12392
- **代码：** https://github.com/rmurai0610/MASt3R-SLAM
- **数据集：** 暂无（评估使用 TUM RGB-D、7-Scenes、ETH3D-SLAM、EuRoC）
- **项目主页：** https://edexheim.github.io/mast3r-slam/

## 一句话总结

MASt3R-SLAM 将 MASt3R 的两视图 3D reconstruction prior 直接嵌入实时 monocular dense SLAM，在约 15 FPS 下同时恢复 camera trajectory 与 dense geometry，并能在不固定参数化相机模型的情况下处理未知或变化的 camera intrinsics，是 moving-camera world-HMR 很强的 camera-only baseline。

## 研究问题与动机

传统 monocular SLAM 往往依赖固定且已知的相机模型，或者把 optical flow、depth、normal 等多个局部 prior 分开建模；对于 casual video、zoom、未知内参和 in-the-wild camera motion，这些假设会限制泛化。MASt3R-SLAM 的出发点是：相比单独预测 depth 或 flow，两视图 foundation model 已经隐式编码了跨视图 3D geometry，因此可以把 MASt3R 的 pointmap / feature 作为统一几何先验，在一个模块化 SLAM 系统里共同解决 pose、camera model 与 dense scene geometry。

## 核心方法

系统以前端、图结构和后端优化组成。MASt3R 对图像对输出共同坐标系中的 pointmaps 与特征；作者将 pointmap 归一化成 rays，从而构造不依赖特定 pinhole 参数化的 generic central camera model。前端使用 projective pointmap matching，将原本昂贵的 3D / feature brute-force matching改写为局部角度误差优化，用于实时 tracking 与 canonical pointmap fusion。

当 tracking 信息不足时建立 keyframe，并通过增量 image retrieval 发现 loop closure。后端在 keyframe graph 上做二阶 Gauss–Newton 全局优化，用 Sim(3) gauge fixing 处理 monocular scale freedom；已知 calibration 时使用 pixel reprojection residual，未知 calibration 时使用 ray error，因此同一框架可以覆盖 calibrated / uncalibrated 情况。

## 数据集与评价指标

作者在 TUM RGB-D、7-Scenes、ETH3D-SLAM 和 EuRoC 上评估 monocular camera trajectory，并在 7-Scenes 与 EuRoC Vicon Room 上评价 dense geometry。主要 trajectory 指标为经尺度对齐后的 ATE RMSE（m）；geometry 使用 Accuracy、Completion 与 Chamfer Distance（m）。系统在 Intel i9-12900K + RTX 4090 上约 15 FPS，实验为模拟实时处理通常每两帧采样一次。

论文同时比较 calibrated 与 uncalibrated setting。对于未知 calibration，系统不要求整个序列共享固定相机模型；在 EuRoC 中由于 MASt3R 本身仅在 pinhole 图像上训练、畸变过强，作者先对图像 undistort，但不把 calibration 提供给其余 pipeline。

## 主要结果

在 TUM RGB-D 上，calibrated MASt3R-SLAM 平均 ATE 为 **0.030 m**，优于 DROID-SLAM 的 **0.038 m** 和 GO-SLAM 的 **0.035 m**；uncalibrated 情况为 **0.060 m**，明显好于先用 GeoCalib 再运行 DROID-SLAM 的 **0.158 m**。在 7-Scenes 上 calibrated / uncalibrated 平均 ATE 分别为 **0.047 / 0.066 m**，其中 calibrated 略优于 DROID-SLAM 的 **0.049 m**。

Dense reconstruction 方面，7-Scenes 上 calibrated 方法的 Accuracy / Completion / Chamfer 为 **0.074 / 0.057 / 0.066 m**，uncalibrated 为 **0.068 / 0.045 / 0.056 m**；DROID-SLAM 为 **0.115 / 0.040 / 0.077 m**。EuRoC 中 DROID-SLAM trajectory 更准（ATE 0.022 vs 0.041 m），但 MASt3R-SLAM 的 geometry Accuracy 和 Chamfer 更好（0.099 / 0.085 m vs 0.173 / 0.117 m）。

匹配消融中，完整 `Ours + feat` 约 **2 ms** matching、**14.9 FPS**；原始 MASt3R brute-force matching 约 **2000 ms、0.4 FPS**。Loop closure 对长序列帮助明显，例如 calibrated EuRoC ATE 从 **0.233 m 降到 0.029 m**。

## 优点

- 把强两视图 3D reconstruction prior 转化为可实时运行的 SLAM，而不是仅做离线 SfM / reconstruction。
- calibrated 与 uncalibrated 情况统一在同一框架内，对 changing intrinsics / zoom 更友好。
- 同时输出 camera trajectory 和 dense geometry，适合后续 human-scene alignment，而不是只提供位姿。
- 模块化程度高，pointmap、loop closure、backend residual 都可以插入 human mask / reprojection / scale / contact 等额外约束。

## 局限

- MASt3R 只在 pinhole 图像上训练，论文明确指出 geometry prediction 会随 distortion 增大而退化；因此不能把它直接视为 ERP/360° 输入的成熟方案。
- 全局优化并没有 refinement 所有 dense geometry，frontend pointmap filtering 与 global pose optimisation 之间仍可能存在几何不一致。
- Full-resolution MASt3R decoder 是 tracking 与 loop-closure latency 的主要瓶颈之一。
- 在 EuRoC 等 aggressive motion / exposure variation 条件下，camera ATE 仍可能落后于 DROID-SLAM 系方法。

## 个人评价

MASt3R-SLAM 对当前 moving-camera 研究的价值主要不是“又一个 SLAM”，而是给出了一个很强的 **camera + dense scene geometry prior baseline**。此前如果只比较 AnyCam / ViPE / 360DVO，camera estimator 和 scene reconstruction 的连接还比较松；MASt3R-SLAM 可以直接提供 trajectory 与 coherent local pointmaps，从而更自然地与 human-scene contact、reprojection 和 global scale 结合。

**推断：**对于 360° 滑雪场景，不应直接把 ERP 输入 MASt3R-SLAM。更合理的比较是：perspective crop / rectified fisheye 上运行 MASt3R-SLAM，与 panorama-level ViPE / 360DVO 并列；随后统一加入人体信息，比较哪一种 camera representation 更容易被 human constraints 修正。

## 与我的研究关联

可以构造以下递进 baseline：

1. `MASt3R-SLAM only`：完全不使用人体，恢复 camera trajectory + dense scene geometry；
2. `+ HMR`：把 camera trajectory 用于 camera-relative → world-coordinate human motion；
3. `+ human mask`：先减少动态人体对 SLAM matching 的干扰；
4. `+ human reprojection / body scale / foot contact`：将人体从被屏蔽的 dynamic object 变成 camera / scene constraint；
5. `+ multi-perspective / dual-360 consistency`：联合多个方向的人体观测修正物理 camera pose；
6. `joint / recurrent refinement`：联合更新 camera rotation、translation、scale、dense scene 与 world human motion。

评价应同时报告 ATE / RPE、W-MPJPE / RTE，以及 human-scene consistency；尤其要检查“camera ATE 降低”是否真的带来 world-human accuracy 改善。

## 后续阅读

- 与 ViPE、360DVO、AnyCam、VGGT 在相同滑雪 / casual-video 数据上比较 camera trajectory、scale drift 与 runtime。
- 补充 MASt3R-SfM、DUSt3R、VGGT-SLAM 等 foundation-model geometry 路线。
- 测试动态人体 `mask-out → human-aware matching → human-as-constraint` 对 ATE/RPE 的逐级贡献。
- 对 perspective crop、fisheye rectification 与 ERP 输入做 distortion sensitivity 分析，避免把 pinhole-trained prior 的失效误认为 camera-human coupling 的问题。
