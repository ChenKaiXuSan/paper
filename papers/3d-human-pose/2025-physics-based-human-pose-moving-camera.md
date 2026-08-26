---
title: "Physics-based Human Pose Estimation from a Single Moving RGB Camera"
authors: "Ayce Idil Aytekin, Chuqiao Li, Diogo Luvizon, Rishabh Dabral, Martin Oswald, Marc Habermann, Christian Theobalt"
venue: "CVPR Workshops 2025 (RHOBIN)"
year: 2025
reading_date: 2026-08-26
status: skimmed
tags:
  - 3d-human-pose
  - moving-camera
  - world-coordinate
  - physics
  - slam
---

# Physics-based Human Pose Estimation from a Single Moving RGB Camera

## 基本信息

- **作者：** Ayce Idil Aytekin, Chuqiao Li, Diogo Luvizon, Rishabh Dabral, Martin Oswald, Marc Habermann, Christian Theobalt
- **会议/期刊：** CVPR Workshops 2025 (RHOBIN)
- **年份：** 2025
- **阅读日期：** 2026-08-26
- **阅读状态：** `skimmed`
- **标签：** `3d-human-pose`, `moving-camera`, `world-coordinate`, `physics`, `slam`
- **论文：** [arXiv:2507.17406](https://arxiv.org/abs/2507.17406)
- **代码：** [aidilayce/PhysDynPose](https://github.com/aidilayce/PhysDynPose)
- **数据集：** MoviCam（官方仓库提供 raw / processed 数据入口）
- **项目主页：** 暂无

## 一句话总结

PhysDynPose 将 4DHumans、DROID-SLAM 与物理仿真结合，把 moving-camera 视频中的 camera-relative 人体恢复到 world coordinates，并利用地面接触、摩擦与时序物理约束减少脚滑和全局漂移。

## 研究问题与动机

移动相机下，人体运动与 camera motion 耦合。普通 HMR 即使局部关节较准，也可能在世界坐标中出现漂移、脚底穿地和不合理 root motion。论文希望在现有 HMR + SLAM 初始化之后，用场景和动力学约束提升 global motion 的物理合理性。

## 核心方法

4DHumans 提供 SMPL 人体初始化，DROID-SLAM 提供 camera trajectory。人体先被变换到 world frame，再在 PyBullet 中构建 humanoid 与 scene height map，利用 foot contact、friction cone、no-sliding、root no-drifting 等约束进行物理优化。

## 数据集与评价指标

论文构建 MoviCam：7 名参与者、7 段序列、约 22,000 帧，其中 5 段为非平坦地形、2 段为平地。GT 包括 SMPL、world human trajectory、moving-camera extrinsics、scene mesh 和 foot contact。评价包括 MPJPE、W-MPJPE、RTE、foot sliding 等。

## 主要结果

在非平坦场景上，PhysDynPose 的 W-MPJPE 约 779.60 mm、RTE 1.16%、foot sliding 3.22 mm；4DHumans 对应约 833.57 mm、1.18%、13.99 mm，WHAM 的 W-MPJPE 约 1352.06 mm。局部 MPJPE 并非全面更优，说明物理一致性与逐帧局部精度存在 trade-off。

## 优点

- 把 camera trajectory、scene geometry 与 human dynamics 放进一个清晰的 world-coordinate pipeline。
- MoviCam 提供 moving-camera + human + scene 的定量 GT，适合作为 benchmark。
- 明确评价 foot sliding 和 root trajectory，而不只报告 PA-MPJPE。

## 局限

- DROID-SLAM 的 monocular scale 需要额外对齐；论文使用前两帧 GT camera trajectory 校正尺度，因此并非完全 GT-free metric recovery。
- 依赖 4DHumans、SLAM 和 scene height information，初始化失败会传入物理优化。
- 物理接触模型主要针对 foot-ground，复杂滑动表面需要重定义。

## 个人评价

这是 moving-camera world HMR 的很实用 baseline。它最值得借鉴的是实验评价体系：camera/global human/physical plausibility 三方面一起看，而不是仅靠 canonical pose error。

## 与我的研究关联

可直接设计 `GT camera → ViPE/AnyCam camera → physics refinement → camera-human joint refinement` 的递进对比，并增加 ATE/RPE、W-MPJPE、RTE、root trajectory、foot sliding。对 360 自拍场景，可进一步让多 perspective 3D keypoints 反向约束 camera trajectory。

## 后续阅读

- JOSH: Joint Optimization for 4D Human-Scene Reconstruction in the Wild。
- HAC: Humans as Checkerboards。
- OnlineHMR / AnyCam / ViPE 的 camera estimation 与 world HMR 方案。
