---
title: "Wid3R: Wide Field-of-View 3D Reconstruction via Camera Model Conditioning"
authors: "Dongki Jung, Jaehoon Choi, Adil Qureshi, Somi Jeong, Dinesh Manocha, Suyong Yeon"
venue: "arXiv:2602.05321（作者官方仓库标注 ECCV 2026，未独立核验会议录）"
year: 2026
reading_date: 2026-10-27
status: skimmed
tags:
  - 360-vision
  - fisheye
  - omnidirectional
  - camera-geometry
  - multiview-reconstruction
  - foundation-model
---

# Wid3R: Wide Field-of-View 3D Reconstruction via Camera Model Conditioning

## 基本信息

- **作者：** Dongki Jung, Jaehoon Choi, Adil Qureshi, Somi Jeong, Dinesh Manocha, Suyong Yeon
- **会议/期刊：** arXiv:2602.05321；作者官方 GitHub 标注 ECCV 2026，独立正式会议录暂未核验
- **年份：** 2026；首次提交 2026-02-05
- **阅读日期：** 2026-10-27
- **阅读状态：** `skimmed`
- **标签：** `360-vision`、`fisheye`、`multiview-reconstruction`、`camera-geometry`
- **论文：** https://arxiv.org/abs/2602.05321
- **DOI：** https://doi.org/10.48550/arXiv.2602.05321
- **代码：** https://github.com/jdk9405/Wid3R （含 demo、训练/评价脚本，官方 checkpoint 下载说明；数据预处理代码仍在 TODO）
- **数据集：** 暂无（无独立发布的 Wid3R 数据集；使用现有公开数据）
- **项目主页：** https://jdk9405.github.io/Wid3R/
- **主要价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** A+

## 一句话总结

通过 camera-model token、球谐函数（spherical harmonics）射线表示及 angular/radial 分解，直接在 fisheye 与 360° 全景多视图上估计相机位姿、稠密点图和不确定性，减少 pinhole 先验带来的畸变失配。

## 研究问题与动机

VGGT、π³ 等 feed-forward 3D 基础模型主要针对 rectified pinhole 训练。fisheye、ERP/球面投影拥有不同的射线几何，先纠畸或切分为透视图既增加预处理，又可能削弱视场覆盖。本文研究在同一多视图网络中支持 pinhole、fisheye 与 spherical camera 的几何恢复。

## 核心方法

- **输入/输出：** N 张 RGB 图像与对应 camera-model condition；网络输出每视角相机 SE(3) 位姿、pixel-aligned ray direction、radial distance 和 uncertainty，组合为 `P_i = R_i ⊙ D_i` 点图。官方代码中的 demo 需用户选择相机模型。
- **Ray-based geometry：** Angular Module 预测 spherical harmonics 系数，得到每像素射线方向；Radial Module 估计射线长度及其误差代理。与直接预测 pinhole point map 的 head 相区分。
- **Camera-model token：** 显式区分投影族；在联合训练时共享几何特征，同时学习特定相机的畸变适配。
- **Local coordinates / pose：** 借鉴 π³ 的 permutation-equivariant 表征，各视图使用局部点图并预测跨视角 pose。训练时联合采用点图、法线、旋转/平移、ray、radial 与 uncertainty losses，尺度在序列层面进行对齐。
- **训练构成：** TartanAirV2、ASE、Hypersim、KITTI-360、360Loc、Matterport3D、ScanNet++、EDEN、Virtual KITTI 2，共九个来源；五个合成、四个真实，360° 图像在训练中不足 1%。含透视转鱼眼与 ERP 旋转增强。

## 数据集与评价指标

- **zero-shot camera pose：** FIORD（fisheye）、Zip-NeRF（fisheye）、Stanford2D3D（360°），采用 RRA@30、RTA@30、AUC@30（报告数值以百分制呈现）及 ATE / translation/rotation RPE。
- **点图与深度：** ScanNet++、Matterport3D、Stanford2D3D；比较 Acc、Completeness、Normal Consistency 以及 depth AbsRel / RMSE。
- **定位速度：** 在 Matterport3D 三个场景对比 SfM/COLMAP/IM360 管线；作者未给出训练图像总数，不能把九个数据集误报成样本量。
- **Stanford2D3D 测试协议：** area_5a、area_5b 每场景随机 10–30 张图，重复 10 次，共 20 个 evaluation cases（论文 Sec. 4.2）。

## 主要结果

- **Stanford2D3D 的 AUC@30：** Wid3R **79.93**、VGGT **2.60**、π³ **2.06**；对应 ATE 分别为 **1.11 / 3.07 / 2.83**（论文 Table 2–3；ATE 原文未在表头声明具体米制单位，不自行推断）。
- **Zip-NeRF AUC@30：** **74.61**，VGGT **20.68**、π³ **40.94**；FIORD：**68.20** vs **44.63** / **48.35**。
- **Stanford2D3D 点图：** mean Acc **0.197**、mean Completeness **0.172**、mean Normal Consistency **0.729**，对应 π³ **0.380 / 0.745 / 0.557**（Table 5）。
- **速度/适用边界：** Matterport3D 三场景，Wid3R 注册帧数与 EDM+IM360 一致，约 **3 秒** vs 约 **30 分钟**；但在部分场景的 AUC@10° 低于该几何管线，不能认为所有精度指标都领先（Table 4）。

## 优点

- 在统一网络中处理多种投影，与双 fisheye / ERP 相机的真实输入条件贴近。
- 用 angular / radial / uncertainty 分解暴露了可针对性消融的几何因素。
- 官方已提供代码、checkpoint 使用说明和 evaluation adapters，适合构建 camera-only 基线。

## 局限

- 主要评价 camera / static-scene geometry，没有 human pose、contact、human-to-camera feedback 或长时间 world-HMR 的直接实证。
- 训练数据 360° 图像低于 1%；长距离高速雪地、人体大遮挡和低纹理背景没有直接验证。
- 最优对齐尺度的训练/评估协议不等于已证明真实双 360 系统绝对 metric scale 完全可观测。
- 官方数据预处理代码仍列为 TODO；论文表中个别 ATE 的单位需阅读实验协议后再与米制 RTK 真值对比。

## 个人评价

**推断：**应将 Wid3R 视为直接处理原始广角/全景的 *camera-only geometry front-end*，而非完整滑雪 world-HMR 方案。它比“把 ERP 切成透视图后运行 VGGT”更有针对性；但必须在相同输入帧、校准条件和计算预算下实测，不宜直接横比来自不同论文的 AUC。

## 与我的研究关联

**推断：**用于双 360° 滑雪 `fisheye/ERP → camera R/t/scene depth → multi-view HMR → human-assisted camera correction`。建议建立 raw wide-FoV Wid3R、ERP→perspective crop VGGT、ViPE 与 360DVO 的公平比较，使用 RTK/IMU 或校准的物理基线检查 ATE/RPE、scale drift、roll/yaw 和 tracking coverage，再报告 fused W-MPJPE。可将预测的 depth/ray uncertainty 与现有 joint-wise/axis-wise pose reliability 结合，按轴抑制不稳定视图，但这属于待验证的研究假设。

## 后续阅读

- 重点阅读 **Sec. 3.2–3.4、Sec. 4.1–4.2、Tables 2–5、7**，复现 camera token、ray/radial 结构和异质投影训练消融。
- 对比已收录 VGGT、ViPE、360DVO、PanoAir、Holo360D 与 AnyCam。
- 官方数据：暂无线性统一发布包；原始训练集应分别遵守各数据源许可。
