---
title: "Human Mesh Reconstruction of Sports Players with Multiple Dynamic Cameras"
authors: "Yamato Hokari, Ryosuke Hori, Hideo Saito"
venue: "CVPR Workshops (CVSPORTS), pp. 6095–6105 (2025)"
year: 2025
reading_date: 2026-11-10
status: skimmed
tags:
  - global-human-motion
  - multi-moving-camera
  - multi-view-pose-fusion
  - sports-biomechanics
  - smpl
  - confidence-weighting
  - similarity-alignment
---

# Human Mesh Reconstruction of Sports Players with Multiple Dynamic Cameras

## 基本信息

- **作者：** Yamato Hokari, Ryosuke Hori, Hideo Saito
- **会议/期刊：** IEEE/CVF CVPR Workshops (CVSPORTS 2025), 6095–6105；会议 2025-06；IEEE Xplore 2025-09-15 上线。正式会议 Workshop 论文；2025 年重要论文补读。
- **年份：** 2025
- **阅读日期：** 2026-11-10
- **阅读状态：** skimmed
- **标签：** global-human-motion, multi-moving-camera, multiview-fusion, sports, sim3-alignment, confidence
- **价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** 高（与双移动 360° 滑雪 pose fusion 直接相关）
- **论文：** https://openaccess.thecvf.com/content/CVPR2025W/CVSPORTS/papers/Hokari_Human_Mesh_Reconstruction_of_Sports_Players_with_Multiple_Dynamic_Cameras_CVPRW_2025_paper.pdf
- **DOI：** https://doi.org/10.1109/CVPRW67362.2025.00602
- **代码：** 暂无（未核验到本文作者的独立官方实现，不把 TRAM 仓库误记为本文代码）
- **数据集：** https://github.com/rawalkhirodkar/egohumans
- **项目主页：** 暂无（CVF 会议论文页面可用：https://openaccess.thecvf.com/content/CVPR2025W/CVSPORTS/html/Hokari_Human_Mesh_Reconstruction_of_Sports_Players_with_Multiple_Dynamic_Cameras_CVPRW_2025_paper.html）

## 一句话总结

从每台移动 RGB 相机独立获得 TRAM world-motion/SMPL 估计，利用跨视角人体身份聚类、人体 root 轨迹的 Sim(3) 对齐和置信度加权融合，在没有预先相机外参标定的情况下改善多人物全局运动重建；但没有联合优化物理相机轨迹。

## 研究问题与动机

体育动作可能同时存在高速运动、严重遮挡和多台手持/动态相机。单视角 camera-relative HMR 的深度和遮挡歧义较大，固定外参多机位方法难直接用于自由运动的拍摄系统。本文试图把每台相机的独立人体重建对齐到统一的全球参考系，再按可见性和置信度进行多视角融合。其 calibration-free 主要指不要求事先标定相机之间的外参，并非摆脱相机与人体估计误差或得到独立的绝对地理坐标。

## 核心方法

1. **Per-camera 前端（Section 4.1）：** 对同步 RGB 视频独立运行 TRAM，输出每人每帧 SMPL shape β、body pose θ、global orientation τ、root translation π；TRAM 内部依赖 SLAM + 背景深度与 VIMO。
2. **跨时间/跨相机身份关联（Section 4.2）：** 用轨迹相交帧的 PA-MPJPE 构造候选人体之间的图距离。通过 spectral clustering + k-means 汇总同一人物在不同观察者下的估计；需提前知道人数 N，类似姿态的人易被误分。
3. **坐标系对齐（Section 4.3）：** 对跨相机共同可见的人体 root 位置求解带比例尺度的 Procrustes Sim(3) 转换 (s,R,t)；按平均配准残差选择参考相机，再将其余视角的 root 和全局朝向变换至参考系。这是对“各视角人体轨迹”的整体 similarity alignment，并不是直接反向求解每帧 camera R/t。
4. **人体融合（Section 4.4）：** shape、body pose、global orientation 采用每视角置信度的幂次归一化加权，旋转先转换为 SO(3) 矩阵，经加权和与 SVD 投影恢复合法旋转；global root translation 则使用简单算术平均，并**没有**按坐标轴或关节、按可见性分别加权。

## 数据集与评价指标

- **数据源：** EgoHumans，官方全集为 133 场景、2–4 人、8–20 外部相机，并含佩戴式 egocentric 视角；本文从 5 类活动（tagging、Lego、volleyball、badminton、tennis）选择 **10 个场景**，按 2 / 4 / 6 相机设置测试，每场景先挑选 8 个可用相机作为候选。不要把 133 场景误记为本文全部评估样本。
- **输入/输出：** 同步多路 RGB 视频 → 每人 SMPL mesh、body pose 与参考坐标下的 global human trajectory。
- **指标：** PA-MPJPE (mm)、ACCEL (m/s²)、W-MPJPE100 (mm)、WA-MPJPE100 (mm)。后两项采用不同全局对齐方式，不能与 camera ATE 混淆。
- **baseline：** WHAM 和 TRAM 单视角；另有 6 相机不使用 confidence 的参数平均消融。

## 主要结果

- **局部姿态（Table 2）：** WHAM PA-MPJPE **116.1 mm**、TRAM **49.6 mm**；本文 2/4/6 相机分别 **45.6 / 40.2 / 38.1 mm**。ACCEL：TRAM 17.5、本文六机 14.2 m/s²。
- **全局轨迹（Table 3）：** W-MPJPE100 从 TRAM **301.9 mm** 降至六机 **200.7 mm**；WA-MPJPE100 从 **820.2 mm** 降至 **427.9 mm**。需要严格保留表中的 alignment 定义。
- **置信度消融（Table 4）：** 六机简单平均的 PA-MPJPE **38.7 mm**、ACCEL **14.8 m/s²**；confidence fusion 为 **38.1 mm / 14.2 m/s²**。相对幅度较小，主要收益还来自跨视角信息与融合。
- 论文未提供逐帧物理相机的 ATE/RPE 改善报告，不能将人体 W-MPJPE 的改善等价表述为 SLAM/camera refinement 成功。

## 优点

- 和双移动相机直接相关：有清晰的 per-view world-HMR → 轨迹级 similarity alignment → 融合的可实现 baseline。
- 报告 2/4/6 视角消融、global trajectory 与 local pose，且和 TRAM/WHAM 比较。
- 使用 SO(3) 矩阵加权和 SVD 正交化，避免直接平均 axis-angle 引起的无效旋转。

## 局限

- 作者明确指出：**预先需要知道人数**；多人姿态相似时 spectral clustering 可能交换身份，导致错误 mesh。
- 底层相机世界轨迹作为 TRAM 上游结果；方法没有回传人体约束修正 camera pose，也没有 camera ATE / intrinsics / scale 的独立精度评估。
- 基于 Sim(3) 人体 root 的配准与参考相机锚定：相对一致性改善不自动证明真实 world metric scale 正确（**推断**）。
- 只抽样 EgoHumans 10 个场景，缺双移动 360°/ERP、高速滑雪、低纹理雪地或未知同步偏差的直接验证。
- 公开的逐视角置信度加权对 PA-MPJPE 的额外提升只有 0.6 mm；仍使用 root 位置均值，不体现 joint-axis confidence（**推断**）。
- 未找到可核验的本论文官方代码，应将复现细节以论文公式为准。

## 个人评价

**推断：**论文适合当作你现有双视角 CanonFuse3D 中比“平均 3D pose”更强的 multi-moving-camera baseline，而不适合直接当作 camera-human mutual refinement SOTA。对论文宣称 calibration-free，应严谨拆开“无需预先跨物理相机外参”、“用人体 world trajectory 做 Sim(3) 配准”、“真正优化每帧 camera pose”三个不同层级。

## 与我的研究关联

**推断：**在双 360° 滑雪实验中，先将两台物理相机得到的 3D body/root 序列以本文 Sim(3) 对齐，再复现跨观察者身份关联和 SMPL-rotation SVD 融合；追加你目前 per-joint/per-axis reliability 与 cross-view attention 以比较相对于标量 confidence 的增益。对于共享光心的同一台 360 相机裁切 view，不应错误地当作有 baseline 的多个独立相机做三角化。若后续研究 camera-human mutual refinement，须另加显式球面 reprojection、SLAM residual 与相机 ATE/RPE；本方法本身没有这个环节。

## 后续阅读

- 优先阅读 Section 4.2–4.4（spectral clustering、式 (4)–(12) 的 Sim(3)/SO(3) fusion），Tables 1–4，Section 7 的失败案例。
- 与 [TRAM](2024-tram-global-trajectory-and-motion.md)、[SmartMocap](2023-smartmocap-uncalibrated-moving-cameras.md)、[PACE](2024-pace.md)、[Multi-camera calibration using people](../multiview-geometry/2025-spatiotemporal-multicamera-calibration-people.md) 比较，分别统计 camera error、root trajectory、PA-MPJPE、W-MPJPE、ACCEL、各轴误差。
- 在 2/4/6 视角、人体近似相同动作、不同 overlap、遮挡、同步误差和相机 scale drift 条件下压力测试。

## 核验来源

- https://openaccess.thecvf.com/content/CVPR2025W/CVSPORTS/papers/Hokari_Human_Mesh_Reconstruction_of_Sports_Players_with_Multiple_Dynamic_Cameras_CVPRW_2025_paper.pdf
- https://openaccess.thecvf.com/content/CVPR2025W/CVSPORTS/html/Hokari_Human_Mesh_Reconstruction_of_Sports_Players_with_Multiple_Dynamic_Cameras_CVPRW_2025_paper.html
- https://doi.org/10.1109/CVPRW67362.2025.00602
- https://github.com/rawalkhirodkar/egohumans
