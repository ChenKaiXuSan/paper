---
title: "HAMSt3R: Human-Aware Multi-view Stereo 3D Reconstruction"
authors: "Sara Rojas, Matthieu Armando, Bernard Ghanem, Philippe Weinzaepfel, Vincent Leroy, Gregory Rogez"
venue: "ICCV 2025"
year: 2025
reading_date: 2026-10-24
status: skimmed
tags:
  - multiview-geometry
  - human-scene-reconstruction
  - uncalibrated-cameras
  - mast3r
  - densepose
  - camera-pose
---

# HAMSt3R: Human-Aware Multi-view Stereo 3D Reconstruction

## 基本信息

- **作者：** Sara Rojas, Matthieu Armando, Bernard Ghanem, Philippe Weinzaepfel, Vincent Leroy, Gregory Rogez
- **会议/期刊：** ICCV 2025, pp. 5027–5037；arXiv:2508.16433（2025-08-22 初次提交）
- **年份：** 2025
- **阅读日期：** 2026-10-24
- **阅读状态：** `skimmed`
- **标签：** `multiview-geometry`, `human-scene-reconstruction`, `uncalibrated-cameras`, `mast3r`, `densepose`, `camera-pose`
- **论文：** https://openaccess.thecvf.com/content/ICCV2025/html/Rojas_HAMSt3R_Human-Aware_Multi-view_Stereo_3D_Reconstruction_ICCV_2025_paper.html
- **代码：** 暂无
- **数据集：** 暂无（使用既有 EgoHumans、EgoExo4D 等）
- **项目主页：** https://europe.naverlabs.com/research/publications/hamst3r-human-aware-multi-view-stereo-3d-reconstruction/
- **arXiv：** https://arxiv.org/abs/2508.16433
- **主要价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** A+

## 一句话总结

将 MASt3R 系列的场景点图与人体实例、DensePose 语义结合，用未标定稀疏多视角图像联合恢复人体与场景；在 EgoExo4D 改善 world-space 人体定位，但不等于已解决动态相机轨迹的联合优化。

## 研究问题与动机

传统 dense multi-view reconstruction 更侧重静态几何；人体彩色纹理变化、姿态非刚性和遮挡，使场景点图基础模型对人体区域处理不足。作者希望不依赖先验相机标定，让场景几何表征具备人体语义，在同一世界空间下提供稠密点图、人体区域与可拟合身体模型的对应。

## 核心方法

- 将从 MASt3R、Multi-HMR 等蒸馏而来的 DUNE encoder 作为共享特征前端；使用 Siamese ViT 和 cross-attention decoder，对两张输入图像估计 pointmaps、匹配与人体信息。
- 添加人体实例分割、binary mask 与 DensePose heads，将人体语义附着在预测点图上；采用加权的点图、分割、DensePose 和 mask 损失，训练时冻结 DUNE encoder。
- 双视图预测是 feed-forward；更多视图需要计算图像对、借助 MASt3R 类全局点图对齐合并，按 2D 重叠关联人物 ID，并按置信度聚合 DensePose。人体 SMPL 通过拟合预测的 3D 人体点与 VPoser 先验获得，属于后续优化而非全流程零优化。
- 注意：作者的 human-aware 表征不等同于人体关键点显式反向修正运动相机时序轨迹的 BA。

## 数据集与评价指标

- 训练包含程序生成 HumGen3D：524,000 张图像、10,000 场景、约 1,000 种环境，典型场景约 5 人；另结合 BEDLAM、HuMMan、EgoBody 与 MASt3R 原始数据，训练时人本与原始数据各约 50%。
- 主要人体测试为 EgoHumans、EgoExo4D；官方文段没有一个可统一合计的独立测试片段/人数值，不自行填补。
- 输入：多张未标定 RGB（按图像对处理）；输出：3D pointmaps、人体 mask / instance / DensePose，再经 SMPL fitting 得到人体姿态。
- 人体评价：W-MPJPE、GA-MPJPE、PA-MPJPE（论文单位 m）；相机评价：TE、s-TE、AE、RRA、CCA；通用立体/位姿任务还使用 KITTI、ScanNet、ETH3D、DTU、Tanks and Temples、CO3Dv2、RealEstate10K。

## 主要结果

- EgoExo4D 上，HAMSt3R 的 **W-MPJPE = 0.51 m**，优于联合优化基线 HSfM 的 **0.56 m**。
- 但同一数据上的 **PA-MPJPE = 0.09 m**，逊于 HSfM 的 **0.06 m**。world-space 改善不代表局部人体姿态全优。
- 作者指出 EgoHumans 上部分 camera pose 指标反而低于 DUNE 单独估计，可能与大场景及点图误差的 log scaling 有关；通用深度性能相对 MASt3R 也有代价。
- 官方来源：ICCV 2025 论文及 CVF 正式 PDF 的 Sections 3–4 / Tables 1–6；作者机构项目页。

## 优点

- 将人体 DensePose/instance 与稠密场景点图置于同一可关联空间，而非分别独立预测再简单融合。
- 在无人工先验相机标定下兼顾人体与相机/场景几何，并明确报告 world/local pose 和 camera metrics。
- 提供与 HSfM、UnCaliPose、Multi-HMR 的比较及跨普通 3D 任务检验。

## 局限

- 多于两视图仍要 pairwise prediction 与全局 3D 对齐，人体 SMPL 要额外优化，不能视为完全端到端多帧时序系统。
- 以 pinhole/常规多视角为主要对象；没有原生 fisheye/ERP 或双高速 moving 360° 实验。
- 人本训练包含大量合成场景；室外雪地、远距离人体与低纹理长序列的 domain gap 仍需检验（推断）。
- 局部 PA-MPJPE、通用深度和部分大场景相机精度不一定同时改善。

## 个人评价

相比只提升人体 MPJPE 的方法，这篇最有启发的是将人体语义直接嵌入基础模型预测的 3D scene representation。也应警惕在 EgoExo4D 世界指标和局部 PA 指标出现反向排名：不能只报单一平均误差。论文的双视图 feed-forward 与多视图 global alignment 应分开计时和对比。

## 与我的研究关联

**推断：**适合作为 dual-360 视角下的 `scene pointmap + human semantic` baseline；首先在每台相机各自 rectified perspective crop 上预测点图，再以真实双相机 baseline 对齐，分别评价 camera ATE/RPE、scale、global W-MPJPE 与 local PA-MPJPE。与现有 axis-wise reliability fusion 比较时，增加 DensePose/mask 支持的 joint observability；之后再研究 fused human geometry 是否能以可微残差反向改进相机轨迹。不要把同一 360 相机的同中心 crops 当作真实 stereo baseline。

## 后续阅读

- 阅读 ICCV 原论文 Sec. 3（heads、loss 与多视图点图聚合）、Sec. 4 / Tables 2–6（EgoHumans/EgoExo4D 与 camera/depth trade-off）。
- 比较已有 [HSfM](../global-human-motion/2025-hsfm.md)、[CHROMM](2026-chromm.md)、[VGGT](2025-vggt.md)。
- **推断：**复现 `scene-only pointmap → +human masks/DensePose → +cross-view alignment → +human-to-camera optimization`，对 camera layout、错误内参和缺失视角作消融。
