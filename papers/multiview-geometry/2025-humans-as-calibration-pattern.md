---
title: "Humans as a Calibration Pattern: Dynamic 3D Scene Reconstruction from Unsynchronized and Uncalibrated Videos"
authors: "Changwoon Choi, Jeongjun Kim, Geonho Cha, Minkwan Kim, Dongyoon Wee, Young Min Kim"
venue: "IEEE/CVF International Conference on Computer Vision (ICCV) 2025, pp. 6598–6608"
year: 2025
reading_date: 2026-11-04
status: skimmed
tags:
  - multi-view-geometry
  - camera-calibration
  - temporal-synchronization
  - human-motion
  - dynamic-nerf
  - moving-camera
  - joint-optimization
---

# Humans as a Calibration Pattern: Dynamic 3D Scene Reconstruction from Unsynchronized and Uncalibrated Videos

## 基本信息

- **作者：** Changwoon Choi, Jeongjun Kim, Geonho Cha, Minkwan Kim, Dongyoon Wee, Young Min Kim
- **会议/期刊：** IEEE/CVF International Conference on Computer Vision (ICCV), October 2025, pp. 6598–6608（正式会议论文）
- **年份：** 2025；arXiv 首次提交：2024-12-26
- **阅读日期：** 2026-11-04
- **阅读状态：** skimmed
- **标签：** multi-view-geometry, camera-calibration, temporal-synchronization, human-motion, dynamic-nerf, moving-camera, joint-optimization
- **论文：** https://openaccess.thecvf.com/content/ICCV2025/html/Choi_Humans_as_a_Calibration_Pattern_Dynamic_3D_Scene_Reconstruction_from_ICCV_2025_paper.html
- **arXiv：** https://arxiv.org/abs/2412.19089
- **代码：** 暂无（作者项目页标记 Code (soon)，未确认可用官方实现）
- **数据集：** 暂无（正文评估 CMU Panoptic Studio、Mobile-Stage 与 EgoBody；不将未经核验的下载地址当作官方链接）
- **项目主页：** https://changwoonchoi.github.io/HCP/
- **价值类型：** Method Module / Baseline / Related Work
- **阅读优先级：** 高（2025 正式会议论文，补读）

## 一句话总结

把不同视频里可恢复的**人体 3D 关节动作**当成动态标定板，用 canonical-pose 时间对齐和 Procrustes 空间对齐初始化相机与时间偏移，再把二者带入动态 NeRF 做渐进式联合优化，从而在无预先同步、无外参输入的多视角视频中完成场景重建。

## 研究问题与动机

动态 NeRF 常假设视频已同步且 camera extrinsics 已知；传统 COLMAP/SfM 在纹理不足、动态主体占比高时可能失败。如果多视角采集本身不具有硬件同步和准确标定，直接优化动态 NeRF 易落入局部极小。作者认为人体动作的语义一致性比像素匹配更适合作为跨视角中间表示，尤其适用于背景变化大且人物动作可重复匹配的序列。

## 核心方法

1. **单视频人体初始化：** 每路视频用 SLAHMR 估计人体 SMPL-H 形状、局部姿态、全局根朝向/平移与关联的相机运动；单视频输出暂不需要跨相机几何配准。
2. **时间同步：** 将 22 个关节变换至去除全局朝向的 canonical body frame，以关节 3D 位置距离做 pairwise Dynamic Time Warping；合并 pairwise 相对 offset 与 matching costs，求一组全局一致的时间偏移；混合 FPS 时对低 FPS 关节序列插值。
3. **空间标定：** 使用同步后具有 root rotation/translation 的人体 3D joints，通过 Procrustes 求 Sim(3) 变换，统一各视频坐标系，再更新相机轨迹。
4. **细化：** 使用 K-Planes 4D NeRF 的颜色/密度场及 photometric rendering loss，先拟合场景、再解冻相机姿态、最后解冻时间偏移；多分辨率空间-时间网格逐渐启用高频特征以保证稳定性。论文中的细化阶段针对静态多相机视频为每路视频优化一个 camera pose；移动相机 EgoBody 仅报告初始化评价，不应把静态相机的 refinement 数值外推到移动相机。

## 数据集与评价指标

- **CMU Panoptic Studio：** 五个场景（Baseball, Office1, Office2, Office3, Tennis），31 台相机形成多视图，约 29–30 路用于动态 NeRF 训练、1 路留作新视角评估；按帧人为引入不同起始时间，以 GT 偏移和姿态作评价而非方法输入。
- **Mobile-Stage (4K4D)：** 三人舞蹈，使用 20 个视角训练、1 个视角测试新视角合成。
- **EgoBody：** 两个人体、4/5 台静态 Kinect 与一台移动的 HoloLens2；因视图稀疏只评估初始化。
- **指标：** camera rotation error（度）、translation error（Panoptic 使用 cm；Mobile-Stage/EgoBody 以 scene bbox=1 归一化）、time offset error（帧）、novel-view PSNR（dB）/SSIM/LPIPS。未报告统一的下游人体 W-MPJPE 或长序列移动相机 ATE/RPE。

## 主要结果

- Panoptic 五个场景平均旋转误差：**5.46° → 0.412°**（初始化 → 完整细化）；平移误差 **25.1 cm → 0.22 cm**；时间偏移误差 **1.343 帧 → 0.028 帧**，起始随机错位平均约 30.64 帧。注意这些误差在与 GT 对齐后评估。
- Panoptic 新视角合成平均 PSNR：**26.55 dB**，高于给了 GT 相机姿态的 Sync-NeRF **21.19 dB**，接近 GT pose+time oracle **27.16 dB**；完整方法 SSIM/LPIPS 为 **0.874/0.112**。
- Mobile-Stage PSNR：完整方法 **23.27 dB**，Sync-NeRF（GT pose）**17.26 dB**。EgoBody 初始化的旋转误差平均 **9.07°**、time offset error **3.067 帧**；无细化结果。
- 论文对混合 FPS、光照退化、SMPL 参数噪声做鲁棒性测试；渐进优化对避免不稳定联合拟合很重要。
- 一手来源：ICCV 2025 论文 Sec. 3、Sec. 4、Tables 1–3；作者项目页 https://changwoonchoi.github.io/HCP/ 。

## 优点

- 同时处理未知跨视频时差与 camera pose，而非先假设完成硬件同步和准确 SfM。
- canonical joint sequence 作为跨视角对应的中间表示，与 RGB 光度变化相对解耦。
- 明确从 initialization 到 joint refinement 的数值增益，对误差传播的分析很直观。
- 有跨多视角、跨不同设备、部分移动相机数据的研究设计。

## 局限

- **核心边界：** Panoptic/Mobile-Stage 的高精度 camera/time refinement 主要针对静态多相机；EgoBody 的移动相机仅测试初始化，不能证明所述 0.412° / 0.22 cm / 0.028 帧结果适用于双自由移动相机。
- 假设已知 camera intrinsics，且多人物情况下已知跨视角 person correspondence；模型依赖 SLAHMR 的姿态/形状和运动可观察性。
- 大规模、离线 300k 次优化与密集视角，不是现成的实时双 360° pipeline；没有 ERP/fisheye 与低纹理雪地验证。
- 完整官方代码尚未核验公开；从官方项目页可见 Code (soon)。

## 个人评价

这篇工作的可迁移价值主要在于 **human-based temporal registration → camera Sim(3) alignment → multi-stage refinement** 三段式设计；它不是“移动相机下已经验证完备的 human-to-camera 在线闭环”。作者给 GT camera poses 的 Sync-NeRF 作为放宽条件的对比，比较结果说明初始化很关键，但不能替代真实双移动相机 SLAM baseline。

## 与我的研究关联

**推断：**双 360° 运动相机采集可先用 CanonFuse3D 的 pelvis-centered canonical joints 建立两路 wrist/hip/shoulder 时序，估计时间 offset（也可与 TC 硬件同步作残差校准）；再以真实双相机的有基线几何做 Sim(3)/SE(3) 估计，加入 joint-wise confidence。与现有静态 RX0 II 12 相机系统比较时，应分别测量 camera R/t error、time offset error、W-MPJPE、velocity/jerk。若加入人体对 camera state 的反向反馈，需对比仅 pose initialization 与 full online camera correction，避免仅改善人体融合却误称 camera accuracy 改善。

**建议阅读/复现：** Sec. 3.2 的 DTW 与全局同步、Eq. (6)–(7) 的 human joint Procrustes；Sec. 3.3 的分阶段解冻；Table 1/2 的同步与姿态误差及噪声消融。先在 12 相机定点实验上验证，再在双移动 360 上评估退化。

## 后续阅读

- [Spatiotemporal Multi-Camera Calibration using Freely Moving People](../multiview-geometry/2025-spatiotemporal-multicamera-calibration-people.md) —— 相邻 human-as-calibrator 方案；比较时间同步假设和优化变量。
- [PACE](../global-human-motion/2024-pace.md) —— 人体先验参与 full camera R/t 优化。
- [SynCHMR](../global-human-motion/2024-synchmr.md) —— 从人体信息辅助 camera/scene metric calibration。
- **待核验：** 作者官方 GitHub 代码是否真正开放；未来版本能否提供移动相机 NeRF refinement。
