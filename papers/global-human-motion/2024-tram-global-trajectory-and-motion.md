---
title: "TRAM: Global Trajectory and Motion of 3D Humans from in-the-wild Videos"
authors: "Yufu Wang; Ziyun Wang; Lingjie Liu; Kostas Daniilidis"
venue: "ECCV 2024; arXiv:2403.17346"
year: 2024
reading_date: 2026-11-01
status: skimmed
tags:
  - global-human-motion
  - moving-camera
  - camera-trajectory
  - metric-scale
  - slam
  - video-hmr
  - world-coordinate
---

# TRAM: Global Trajectory and Motion of 3D Humans from in-the-wild Videos

## 基本信息

- **作者：** Yufu Wang, Ziyun Wang, Lingjie Liu, Kostas Daniilidis
- **会议/期刊：** European Conference on Computer Vision (ECCV 2024)，Springer LNCS
- **年份：** 2024
- **首次提交：** 2024-03-26（arXiv v1）；2024-09-02 更新 v2
- **DOI：** https://doi.org/10.1007/978-3-031-73247-8_27
- **阅读日期：** 2026-11-01
- **阅读状态：** skimmed
- **价值类型：** Baseline / Method Module / Related Work
- **阅读优先级：** 高（经典补读：并非近期新论文）
- **标签：** moving-camera, metric-scale, SLAM, video-HMR, world-trajectory
- **论文：** https://www.ecva.net/papers/eccv_2024/papers_ECCV/html/1796_ECCV_2024_paper.php
- **代码：** https://github.com/yufu-wang/tram
- **数据集：** https://eth-ait.github.io/emdb/ （EMDB，需申请获取；另评估 3DPW）
- **项目主页：** https://yufu-wang.github.io/tram4d/
- **arXiv：** https://arxiv.org/abs/2403.17346

## 一句话总结

TRAM 用“动态人体双重 masking 的 SLAM → 鲁棒的相机 metric scale 恢复 → VIMO camera-relative HMR → world-motion 组合”构成强 camera-first 基线，并把相机漂移与人体局部姿态误差分开测量。

## 四段式推荐摘要

移动相机视频中的人体三维姿态若要进入世界坐标，必须知道相机轨迹与实际尺度。传统 HMR 擅长局部人体形态，却容易把相机运动误差传给全局轨迹。TRAM 将问题拆成抗动态前景的相机恢复，以及时序人体运动估计，再将两条轨迹组合。

输入单目 RGB 视频，输出 metric camera trajectory 与 world-space 3D human motion。核心是双重 masking 的 DROID-SLAM、ZoeDepth 支持的鲁棒尺度优化及 VIMO 时序 Transformer。EMDB-2 上 camera ATE 从原始 DROID-SLAM 的 2.42 m 降至 0.32 m；估计尺度下 ATE-S 达 0.66 m；对 WHAM，人体 RTE 从 4.6% 降至 1.4%。

**推断：**适合作为双 360° 滑雪的 perspective/rectified-view camera-only + HMR 基线。建议优先读第 3.2–3.4 节并复现双重 masking、尺度估计消融，同时报告 ATE/ATE-S、W-MPJPE 与 RTE，以分辨 camera 改进和人体改进。

主要边界是相机先估计再送入人体模型，尚无 human→camera 闭环；效果依赖分割、背景静态纹理、单目 depth prior 与透视相机假设。**推断：**低纹理雪场、强人体遮挡和原生 ERP/fisheye 会要求专门适配，不能把 pinhole 结果直接外推。

## 研究问题与动机

- Camera-relative HMR 的世界轨迹质量受相机旋转、平移与 metric scale 误差影响。
- 动态人体会污染 DROID-SLAM 的特征匹配及 dense bundle adjustment；简单输入遮罩不足以消除优化阶段的前景影响。
- 直接把单目 metric depth 当 DROID RGB-D 输入会放大 depth 噪声。本文以 metric depth 对相对轨迹做鲁棒尺度拟合，而不是硬替换深度。
- 这是 **camera-first staged baseline**，不是相机与人体参数的联合反向优化。

## 核心方法

1. **双重动态前景 masking：**以 YOLOv7 人体检测引导 SAM，分别遮蔽 SLAM 输入图像与 DROID-SLAM dense bundle adjustment（DBA）的动态区域，改善相机轨迹。
2. **Metric-scale alignment：**以 ZoeDepth 预测深度为度量尺度线索，对 DROID 的相对深度/轨迹进行鲁棒尺度优化并聚合帧级尺度，避免直接用不可靠预测深度作为 RGB-D 观测。
3. **VIMO：**HMR2.0 相关初始化基础上的视频人体模型，通过 token temporal attention 和 motion temporal attention 表征时序，预测相机坐标系中的人体运动、SMPL 形态。
4. **World composition：**以已尺度化 camera trajectory 变换 camera-relative 人体运动，得到世界坐标人体轨迹。
5. **复现入口：**作者代码提供 estimate_camera.py、estimate_humans.py、visualize_tram.py 及 EMDB evaluation 脚本；训练代码与部分预处理数据已在官方仓库发布。

## 数据集与评价指标

- **EMDB / EMDB-2：**EMDB 官方说明总集为 10 人、81 条室内外序列、58 分钟，具真实全局相机与人体运动；TRAM 的 EMDB-2 camera ATE 表分为 5 条短、10 条中、10 条长的共 25 条轨迹，数据集总量不可误当测试子集样本量。
- **3DPW：**用于 camera-relative HMR 对比；论文表格中 3DPW(14)、EMDB(24) 的括号数字指用于计算误差的 **关节数**，不是视频或受试者数量。
- **Camera：**ATE（尺度对齐，m）、ATE-S（使用方法自行估计的尺度，m）。
- **Human：**PA-MPJPE、MPJPE、PVE（mm）、Accel（m/s²）、WA-MPJPE100、W-MPJPE100（mm）、RTE（%）及 ERVE（mm/frame）。
- **Baseline：**DROID-SLAM、ORB-SLAM2、WHAM、TRACE、GLAMR、SLAHMR；局部人体另对比 HMR2.0 等。

## 主要结果

- **Camera / EMDB-2：**DROID-SLAM 原版平均 ATE 2.42 m，Masked DROID 为 **0.32 m**；仅 mask 输入为 1.42 m，仅 mask DBA 为 0.91 m，体现双重 masking 的必要性。ORB-SLAM2 25 条中 9 条追踪失败，因此其平均值不能直接当作全部 25 条的同协议结果。
- **Scale / EMDB-2：**Masked DROID + 直接 ZoeDepth 作为深度输入，ATE-S **3.09 m**；论文提出的尺度估计降低至 **0.66 m**。
- **World human / EMDB-2：**相对 WHAM（w/ DROID），TRAM 的 WA-MPJPE100 **133.3→76.4 mm**，W-MPJPE100 **343.9→222.4 mm**，RTE **4.6%→1.4%**，ERVE **14.7→10.3 mm/frame**。论文项目页将全局误差下降概括为约 60%，应以指标与对比对象逐项报告。
- **VIMO local HMR：**EMDB 的 MPJPE 74.4 mm、PA-MPJPE 45.7 mm；相同训练设置 HMR2.0(ft) 为 82.7 / 49.7 mm。不能只因为局部 MPJPE 优良就认定相机估计也准确。
- **核验来源：**ECVA 正式论文 PDF 第 9–12 页的 Tables 1–4；官方项目页及代码 README。

## 优点

- 明确分离 camera pose、metric scale、local human pose 和 global human trajectory 的误差来源。
- 在公开 EMDB-2 轨迹上报告 masking、尺度拟合和世界人体轨迹的多指标量化消融。
- 官方提供端到端代码、推理、训练和 EMDB 评估脚本，适合直接构造 baseline。

## 局限

- Camera 轨迹先估计且固定输入人体阶段；**不能证明 human-to-camera mutual refinement**。
- 依赖足量静态背景、可靠 human segmentation 和 monocular depth scale；强动态、低纹理与尺度先验偏差可能使结果退化（后半句为**推断**）。
- 该方法主要是 pinhole/perspective 系统，尚不能直接当作 native spherical ERP/fisheye pipeline。
- ATE 允许尺度对齐，ATE-S 不允许使用 GT scale；混淆二者会高估实际 metric 恢复能力。

## 个人评价

这是应补齐的 **2024 年重要 camera-first 对照基线**，不应与 PACE、SynCHMR 或 BodySLAM++ 的 human→camera 路线混为一谈。该文最值得复现的是两级 masking 与 robust scale alignment，而不是只取 VIMO 的 MPJPE。**推断：**在现代 foundation camera estimator 上加入同类前景屏蔽/尺度恢复，是检验新 human-to-camera correction 是否真的有独立贡献的必要前置对照。

## 与我的研究关联

- 对双 360° 移动相机，应先区分真实物理相机的 metric trajectory、从 ERP 切出的共享中心 virtual perspective cameras，以及人体根关节轨迹。
- **推断：**可构造 pipeline：masked camera-only estimation → metric scale alignment → per-view HMR → confidence-aware 3D fusion → optional human-based camera refinement。
- 评价必须同时报告 camera ATE、RPE、ATE-S、scale drift，以及人体 W-MPJPE/RTE/MPJPE、velocity/jerk；不能只看一个 smoothed 3D skeleton 误差。
- 与 WHAM、PACE、SynCHMR、Human3R、PanoVGGT / 360DVO 做相同 camera perturbation 与 background-occlusion ablation，避免混合不同数据协议。

## 后续阅读

- [WHAM](2024-wham.md) — world-grounded motion 对照。
- [PACE](2024-pace.md) — 显式 human-camera full-trajectory joint optimization。
- [SynCHMR](2024-synchmr.md) — 利用人体 metric cue 反向校准 camera/scene。
- [EMDB 官方数据集](https://eth-ait.github.io/emdb/) — 确认 EMDB-2 的申请、划分与协议。
- 复现第 3.2–3.4 与第 4.1–4.3 节（mask image / mask DBA / scale / VIMO）。
