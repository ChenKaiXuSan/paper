---
title: "Decoupling Human and Camera Motion from Videos in the Wild"
authors: "Vickie Ye, Georgios Pavlakos, Jitendra Malik, Angjoo Kanazawa"
venue: "CVPR 2023"
year: 2023
reading_date: 2026-09-20
status: skimmed
tags:
  - world-human-motion
  - moving-camera
  - camera-scale
  - slam
  - human-motion-prior
  - optimization
  - multi-person
---

# Decoupling Human and Camera Motion from Videos in the Wild

## 基本信息

- **作者：** Vickie Ye, Georgios Pavlakos, Jitendra Malik, Angjoo Kanazawa
- **会议/期刊：** Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR), 2023, pp. 21222–21232
- **年份：** 2023
- **阅读日期：** 2026-09-20
- **阅读状态：** `skimmed`
- **标签：** `world-human-motion`, `moving-camera`, `camera-scale`, `slam`, `human-motion-prior`, `optimization`, `multi-person`
- **论文：** https://openaccess.thecvf.com/content/CVPR2023/html/Ye_Decoupling_Human_and_Camera_Motion_From_Videos_in_the_Wild_CVPR_2023_paper.html
- **DOI：** https://doi.org/10.48550/arXiv.2302.12827
- **代码：** https://github.com/vye16/slahmr
- **数据集：** 暂无（论文使用 EgoBody、PoseTrack 与 3DPW，但未发布独立新数据集）
- **项目主页：** https://vye16.github.io/slahmr/

## 一句话总结

SLAHMR 将 DROID-SLAM 提供的相对相机运动与 HuMoR 人体运动先验放入同一世界坐标优化中，通过人体运动约束估计单目相机平移的 metric scale，并显著改善 moving-camera 下的全局人体轨迹，是后续 WHAC、PACE 等 world-HMR / camera-human coupling 工作的重要经典基线。

## 研究问题与动机

单目动态相机视频中，图像只观察到 camera motion 与 human motion 的合成结果；仅依赖人物局部姿态无法恢复真实 world trajectory，而仅依赖单目 SLAM 又存在绝对尺度歧义。SLAHMR 的核心问题是：在不要求完整静态场景三维重建的情况下，是否可以利用背景像素提供的相对 camera motion，以及人体可行运动范围，联合恢复人在世界坐标中的运动、相机尺度与地面。

论文首先用 DROID-SLAM 从背景估计逐帧 world-to-camera `R_t, T_t`，但该轨迹只有未知尺度；同时用 PHALP/PHALP+ 获得每个人的身份轨迹与 camera-frame SMPL-H 初始化。随后引入全局尺度 `alpha`，将人体与相机放进同一坐标系，并通过 2D joint reprojection、人体平滑、形体/姿态先验以及 learned human motion prior 逐级优化。

## 核心方法

SLAHMR 的优化分为三个主要阶段：

1. **World-frame initialization 与 reprojection fitting**：使用 DROID-SLAM 的相对相机 `R/T` 将 PHALP 的 camera-frame global orientation/root translation 转入世界坐标；初始 `alpha=1`，先只优化人体 global orientation 与 root translation，使 3D joints 与 2D keypoints 重投影一致。
2. **Scale + smoothness refinement**：开始联合优化 camera translation scale `alpha`、人体 shape、body pose 与 global trajectory，并使用 joint smoothness、VPoser pose prior、shape prior 约束欠定问题。
3. **HuMoR motion-prior optimization**：最终用 HuMoR 的 transition-based cVAE 约束连续状态，以 learned transition latent rollout 整段人体运动；同时优化 ground plane，并加入 contact-conditioned foot-skate 与 ground-contact losses。

一个必须明确的边界是：这里人体先验主要用于求解 **camera translation 的全局尺度 `alpha`**。DROID-SLAM 给出的逐帧相机 rotation 与相对 translation 方向保持固定，并没有像后来的 PACE 那样让人体 residual 直接更新完整 camera `R_t/T_t`。

## 数据集与评价指标

- **EgoBody**：主要 world-coordinate 定量评估。EgoBody 原始数据集包含 125 个交互序列；SLAHMR 使用其 validation split。论文没有在 SLAHMR 实验段汇总实际参与评估的 validation 序列总数，因此不进一步猜测。
- **PoseTrack**：用于复杂 in-the-wild 多人定性结果及 downstream tracking；通过 identity switches 验证 metric camera scale 是否改善跟踪。
- **3DPW**：仅用于 local pose ablation，因为其 GT 不能直接支持完整 world human trajectory 评价。
- EgoBody 评估使用 DROID-SLAM + **ground-truth intrinsics**，为加速优化将原始视频切成 100-frame segments 独立处理。
- 核心指标：W-MPJPE、WA-MPJPE、PA-MPJPE、Acceleration Error，以及 PoseTrack identity switches。

## 主要结果

EgoBody 上，完整 SLAHMR 达到 **141.1 mm W-MPJPE、101.2 mm WA-MPJPE、25.78 mm/s² Acceleration Error、79.13 mm PA-MPJPE**。相比只依赖局部 pose 预测 world trajectory 的 GLAMR（416.1 / 239.0 / 173.5 / 114.3），全局轨迹和运动平滑度均大幅改善。

消融中，去掉最后的 motion prior + scale stage 后，W/WA-MPJPE 退化到 **234.0 / 152.9 mm**，Acceleration Error 从 25.78 增至 **275.9 mm/s²**；PHALP+ 加未尺度化 SfM camera 时为 253.6 / 150.3 mm，说明 human motion prior 与 metric scale 是主要贡献。

PoseTrack tracking 中，PHALP+ 为 **450 ID switches**；直接加入未尺度化 DROID-SLAM camera 为 446，而加入 SLAHMR 恢复的 `alpha` 后降至 **420**，说明人体约束得到的相机尺度不仅改善重建，也能支持下游世界坐标 tracking。

## 优点

- 把“相机相对几何”与“人体可行运动”明确分离，再通过人体运动解决 monocular SLAM 的 metric-scale ambiguity，问题定义非常清晰。
- 不要求可靠的 dense scene reconstruction，因此比依赖完整 SfM/MVS 场景的路线更适合普通 in-the-wild video。
- 多人共享同一个 camera scale，多个独立人体运动可以共同约束尺度。
- W-MPJPE、WA-MPJPE、Acceleration Error 与 tracking ID switches 从不同层面验证了 global motion，而不是只报告每帧 PA-MPJPE。
- 官方代码完整提供 EgoBody、PoseTrack、3DPW 与 custom video 的 preprocessing、optimization 和 evaluation 配置，适合作为经典可复现 baseline。

## 局限

- 相机 `R_t/T_t` 的相对轨迹由 DROID-SLAM 固定，人体只优化 translation scale；论文在 Discussion 中也明确把“用 human motion prior 进一步更新 camera 与 scene reconstruction”列为未来方向。
- 主要优化为 batch / iterative L-BFGS，并把 EgoBody 切成 100-frame chunks，无法直接满足 online 长序列应用。
- EgoBody 定量实验使用 ground-truth intrinsics；未知/变化 intrinsics 未被处理。
- 假设单一 ground plane，并在纯旋转相机或 camera-human 共线运动等几何退化场景可能恢复不一致轨迹。
- EgoBody 严重人体截断时，进一步优化甚至可能使 local PA-MPJPE 比初始 PHALP+ 更差，说明 motion/global consistency 与局部 pose accuracy 并不总是同步。

## 个人评价

SLAHMR 是理解当前 moving-camera world HMR 文献脉络不可缺少的基线：它把 GLAMR 式“从人体局部运动猜全局轨迹”推进到“camera relative geometry + human motion prior → metric world trajectory”。但它并不是完整意义上的 camera-human mutual refinement，因为人体并未反向修改相机 rotation/translation trajectory，只修正了全局 scale。

**推断：**从研究新颖性角度，SLAHMR 很适合作为 `camera trajectory fixed, scale optimized by human` 的基准层；PACE 则代表 `full camera R/t jointly optimized`，BodySLAM++ 代表 factor-graph tightly-coupled tracking。将这三者同时纳入 baseline，比把所有 human→camera 方法归为一类更能说明新方法究竟新增了什么。

## 与我的研究关联

**推断：**双 360° / moving-camera 滑雪实验可以把 SLAHMR 改造成一个明确的 **scale-only human feedback baseline**：

`360 VO/VIO relative camera → per-view HMR → human motion/contact/anthropometry 优化单一 camera scale → world human trajectory`。

随后再逐级增加 `full camera R/t correction`、dual-camera cross-view consistency、unknown intrinsics、spherical/fisheye reprojection 与 uncertainty weighting，并同时报告 camera ATE/RPE/scale drift 与 W-MPJPE/RTE。这样可以直接回答性能提升来自“尺度校准”还是“真正的 trajectory correction”。建议重点复现 Sec. 3.1–3.3、EgoBody Table 1/3 与 PoseTrack camera-scale tracking ablation。

## 后续阅读

- [PACE: Human and Camera Motion Estimation from in-the-wild Videos](2024-pace.md) — 从 SLAHMR 的 scale-only camera coupling 进一步推进到完整 camera `R_t/T_t` 与人体 motion latent 联合优化。
- [WHAC: World-grounded Humans and Cameras](2024-whac.md) — 利用人体运动恢复 camera metric scale，再更新 world human trajectory。
- [Synergistic Global-space Camera and Human Reconstruction from Videos](2024-synchmr.md) — human-aware metric depth / SLAM 路线。
- [BodySLAM++: Fast and Tightly-Coupled Visual-Inertial Camera and Human Motion Tracking](2023-bodyslam-plus-plus.md) — 实时 factor-graph camera-human tightly coupled baseline。
