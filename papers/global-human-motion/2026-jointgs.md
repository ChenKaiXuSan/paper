---
title: "JOintGS: Joint Optimization of Cameras, Bodies and 3D Gaussians for In-the-Wild Monocular Reconstruction"
authors: "Zihan Lou, Jinlong Fan, Sihan Ma, Yuxiang Yang, Jing Zhang"
venue: "arXiv:2602.04317"
year: 2026
reading_date: 2026-09-23
status: skimmed
tags:
  - world-human-motion
  - moving-camera
  - camera-human-refinement
  - human-scene-reconstruction
  - 3d-gaussian-splatting
  - smpl
  - differentiable-rendering
---

# JOintGS: Joint Optimization of Cameras, Bodies and 3D Gaussians for In-the-Wild Monocular Reconstruction

## 基本信息

- **作者：** Zihan Lou, Jinlong Fan, Sihan Ma, Yuxiang Yang, Jing Zhang
- **会议/期刊：** arXiv:2602.04317
- **年份：** 2026
- **阅读日期：** 2026-09-23
- **阅读状态：** `skimmed`
- **标签：** `world-human-motion`, `moving-camera`, `camera-human-refinement`, `human-scene-reconstruction`, `3d-gaussian-splatting`, `smpl`, `differentiable-rendering`
- **论文：** https://arxiv.org/abs/2602.04317
- **DOI：** https://doi.org/10.48550/arXiv.2602.04317
- **代码：** https://github.com/MiliLab/JOintGS
- **数据集：** 暂无（论文使用 NeuMan 与 EMDB 现有数据集）
- **项目主页：** https://github.com/MiliLab/JOintGS

## 一句话总结

JOintGS 将 camera extrinsics、SMPL parameters 与 foreground/background 3D Gaussians 放入同一个 differentiable-rendering 优化闭环，从 COLMAP 与 HMR2.0 的粗初始化出发，通过静态背景、人体渲染和人景分离之间的互相反馈提高单目动态人体与场景重建鲁棒性。

## 研究问题与动机

现有 monocular human-avatar / human-scene reconstruction 往往把 camera pose 与 SMPL initialization 当作固定输入，但在自然 moving-camera 视频中，COLMAP 与单目 HMR 都可能存在误差。相机误差会破坏多帧几何一致性，人体姿态误差又会污染 foreground/background separation，最终形成级联失败。

JOintGS 的核心问题是：能否不把这些上游估计当作不可修改的真值，而是利用 3D Gaussian Splatting 的 dense photometric supervision，在同一优化过程中持续修正 camera、human 与 scene representation。

论文通过显式 foreground/background disentanglement 构造三个相互作用的路径：静态背景提供 camera refinement 的稳定 photometric anchor；更准确的 camera 使人体 silhouette / appearance 对齐更可靠；更好的 SMPL 与 camera 又减少动态人体泄漏到静态场景，从而进一步稳定 background reconstruction 与 camera estimation。

## 核心方法

给定单目 RGB video，JOintGS 首先使用 COLMAP 获得粗 camera poses 与 sparse point cloud，并用 HMR2.0 初始化 SMPL。人体以 canonical-space 3D Gaussian field 表示，约采样 110k human Gaussians；背景 Gaussian 从 COLMAP sparse points 初始化，通常为 10k–50k points。

Camera refinement 将每帧初始外参写为 `T_t = ΔT_t ∘ T̂_t`，其中 `ΔT_t ∈ SE(3)`，利用 SAM human mask 之外的静态区域 photometric error 优化。人体部分使用 refined camera 后的人体 RGB 与 silhouette rendering loss 更新 SMPL parameters。Camera / SMPL 的改善又帮助 Gaussian foreground-background disentanglement，并通过 RGB、SSIM、LPIPS losses 优化 human/background Gaussian attributes。

训练采用三阶段 schedule：

1. **Warm-up**：固定 camera 与 SMPL，仅优化 Gaussian representations；
2. **Independent optimization**：开放 camera 与 SMPL 更新，但分别依赖 background 与 human-centric supervision；
3. **Joint optimization**：完整联合优化 camera trajectory、body parameters 和 Gaussian fields。

此外，Temporal Offset Module 建模衣物等 pose-dependent residual deformation，Temporal Color Module 建模光照与 appearance variation。论文还使用 RANSAC scale-shift 将 SMPL depth 与 COLMAP reconstruction 对齐到统一 world coordinate。

## 数据集与评价指标

- **NeuMan：**6 条 in-the-wild 单人序列（Seattle、Citron、Parking、Bike、Jogging、Lab），每条约 10–20 s，手持手机移动拍摄；按 80%/10%/10% frames 做 train/val/test。
- **EMDB：**完整数据包含 10 名受试者、81 条视频、约 58 min；论文选择 10 条代表性序列，每条取前 200 frames，并采用 80%/10%/10% split。
- **输入：** monocular RGB video、COLMAP coarse camera/point cloud、HMR2.0 SMPL initialization、SAM human masks。
- **输出：** refined camera extrinsics、SMPL parameters、animatable human 3D Gaussians 与 static-background 3D Gaussians。
- **指标：** novel-view rendering 的 PSNR、SSIM、LPIPS，以及 training time / rendering FPS；论文还通过向 camera/SMPL initialization 加噪声评估鲁棒性。

## 主要结果

NeuMan human-only evaluation 上，JOintGS 达到 **34.84 dB PSNR、0.984 SSIM、0.010 LPIPS**；此前较强的 Vid2Avatar-Pro 为 **32.71 / 0.983 / 0.019**。Full-image 为 **30.23 / 0.913 / 0.072**。

EMDB human-only 为 **30.99 / 0.972 / 0.027**，高于 ODHSR 的 **28.95 / 0.966 / 0.031**。Full-image 上 JOintGS 为 **23.40 dB PSNR、0.785 SSIM、0.173 LPIPS**；ODHSR 的 PSNR 略高（23.79），但 SSIM/LPIPS 较差。

单张 RTX 5090 上 15,000 iterations 约 25 min 收敛，论文报告平均训练时间约 23 min，rendering speed 为 **27.3 FPS**。当 initialization noise 为 `σ=0.01` 时，JOintGS PSNR 仅下降 **0.9 dB**，HUGS 下降 **3.7 dB**。NeuMan 消融中，去掉 synergistic refinement 后 PSNR 从 **34.84 降到 31.38 dB**；去掉 dynamics 后为 34.23 dB。

## 优点

- 不把 camera 与 body initialization 当固定真值，而是构造可微的 camera-body-scene 闭环 refinement。
- Static background、human rendering 与 foreground-background separation 各自承担不同约束，信息流比简单串联 pipeline 更清楚。
- 对初始化噪声有直接 robustness experiment，证明 joint refinement 的作用不仅体现在最终渲染分数。
- 官方代码与预训练 checkpoints 已公开，复现条件较好。
- 3DGS representation 在离线优化之后仍能达到实时 rendering。

## 局限

- 论文明确指出 SMPL 本身限制了 hands / faces 等细粒度区域，未来可用 SMPL-X 等更 expressive body model。
- 方法是**每条序列的离线优化**，并依赖 COLMAP coarse camera、HMR2.0 和 SAM mask 初始化，不是 online feed-forward world-HMR。
- 主要 quantitative metrics 是 PSNR / SSIM / LPIPS；论文没有将 camera ATE/RPE、camera scale drift、W-MPJPE/RTE 作为主要报告指标，因此 camera-human mutual refinement 对几何精度的收益仍缺少直接拆解。
- **推断：**camera refinement 主要由 static-background photometric residual 锚定，人体对 camera 的反向作用更多通过更干净的人景分离间接实现，因此与“显式 human residual 直接修正 camera state”仍应区分。
- **推断：**pinhole COLMAP + RGB reconstruction 的设定不能直接覆盖 ERP/fisheye、低纹理雪地、长距离高速 camera motion 与双 moving-camera 情形。

## 个人评价

JOintGS 对 camera-human mutual refinement 很有参考价值，特别是它把“camera、human、scene 可以在一个 differentiable rendering loop 中反复互相校正”实现得相当具体。和 PACE 的 bundle-adjustment-style human/camera optimization 相比，它更强调 dense photometric 3DGS scene representation；和 Human3R 的 shared recurrent state 相比，它又提供了明确、可拆解的 optimization pathway。

不过从 world-HMR 角度，现有证据更强地证明了 **reconstruction / rendering robustness**，而不是 camera trajectory 或 global human motion 的绝对精度。若用于新的 dual-360 工作，建议把它视为 differentiable joint-reconstruction baseline，而不是直接把其 PSNR 优势等同于 camera-human geometry 已解决。

## 与我的研究关联

**推断：**对 moving-camera / 双 360° 滑雪，可以借鉴如下递进设计：

`360 VO/VIO 或 foundation geometry initialization → spherical/background scene residual → human silhouette/reprojection residual → body scale/contact/velocity consistency → joint SE(3)+SMPL refinement`。

最值得复现的是 §3.4 Synergistic Refinement、三阶段 optimization schedule、初始化噪声实验和 Table 3 消融。自己的实验应进一步把 reconstruction quality 与 geometry quality 分开报告：camera ATE/RPE/scale drift、W-MPJPE/RTE、Jitter/Foot Sliding，以及 RGB/geometry residual 对 camera update 的贡献。

## 后续阅读

- [PACE: Human and Camera Motion Estimation from in-the-wild Videos](2024-pace.md) — human motion prior + background reprojection 的 full camera `R/t` joint optimization。
- [Reconstructing People, Places, and Cameras](2025-hsfm.md) — human-aware SfM / camera-human-scene reconstruction。
- [Human3R: Everyone Everywhere All at Once](2026-human3r.md) — online recurrent shared-state human-scene-camera reconstruction。
- [Joint Optimization for 4D Human-Scene Reconstruction in the Wild](2026-josh-joint-optimization.md) — 另一条 human-scene-camera joint optimization 路线。