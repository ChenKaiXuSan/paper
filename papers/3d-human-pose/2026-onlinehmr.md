---
title: "OnlineHMR: Video-based Online World-Grounded Human Mesh Recovery"
authors: "Yiwen Zhao, Ce Zheng, Yufu Wang, Hsueh-Han Daniel Yang, Liting Wen, László A. Jeni"
venue: "CVPR 2026"
year: 2026
reading_date: 2026-08-18
status: skimmed
tags:
  - 3d-human-pose
  - world-grounded-hmr
  - moving-camera
  - slam
  - video
---

# OnlineHMR: Video-based Online World-Grounded Human Mesh Recovery

## 基本信息

- **作者：** Yiwen Zhao, Ce Zheng, Yufu Wang, Hsueh-Han Daniel Yang, Liting Wen, László A. Jeni
- **会议/期刊：** CVPR 2026
- **年份：** 2026
- **阅读日期：** 2026-08-18
- **阅读状态：** `skimmed`
- **标签：** `3d-human-pose`, `world-grounded-hmr`, `moving-camera`, `slam`, `video`
- **论文：** https://openaccess.thecvf.com/content/CVPR2026/html/Zhao_OnlineHMR_Video-based_Online_World-Grounded_Human_Mesh_Recovery_CVPR_2026_paper.html
- **代码：** https://github.com/Tsukasane/Video-OnlineHMR
- **数据集：** 暂无（未发布专属数据集；实验使用 BEDLAM、3DPW、Human3.6M 和 EMDB）
- **项目主页：** https://tsukasane.github.io/Video-OnlineHMR/

## 一句话总结

OnlineHMR 将视频人体网格恢复与人体中心的增量式 SLAM 组合起来，在只使用当前和历史帧的前提下，同时在线输出世界坐标中的 SMPL 人体运动与相机轨迹，并通过软人体掩码、尺度恢复和时序约束降低动态人体对相机估计的干扰。

## 研究问题与动机

现有 world-grounded human mesh recovery 往往依赖整段视频、未来帧或全局优化，因此难以用于实时反馈、交互式感知和长时间流式处理。即便部分视频 HMR 模型能够逐帧预测人体，其世界坐标恢复通常仍依赖离线 VO/SLAM，系统层面并不真正在线。论文因此把目标定义为同时满足 causality、faithfulness、temporal consistency 和 efficiency：每一帧只能依赖过去和当前输入，同时仍要保持局部人体姿态准确、全局轨迹稳定且延迟可控。

## 核心方法

方法采用两条相互配合的分支。第一条是 camera-coordinate Online HMR：以 HMR2.0/ViT 为初始化，将视频切为因果滑动窗口，只用当前帧作为 query、历史帧作为 cross-attention 的 key/value，并在推理时使用 FIFO key-value cache，从而避免重复计算未来帧。训练除标准 2D/3D keypoint、SMPL 和 vertex 监督外，还加入相对 pelvis 的速度与加速度正则，抑制帧间抖动。

第二条是 Human-Centric Incremental SLAM。系统通过 SAM2 得到人体区域，对掩码进行膨胀和高斯平滑形成 soft human mask，使 MASt3R-SLAM 在匹配时降低动态人体区域的权重；再使用 MoGe-V2 的 metric depth 将 SLAM 的任意尺度平移恢复为 metric scale，并通过历史相机外参的 EMA/物理连续性修正抑制轨迹抖动。最终把 camera-coordinate SMPL 通过估计的相机旋转、平移和尺度变换到 world coordinate。

论文还提出频域运动稳定性分析：对关节序列做 STFT，以时频能量描述 jitter，作为 Accel/Jitter 等时域指标之外的补充。

## 数据集与评价指标

训练使用 BEDLAM、3DPW 和 Human3.6M。论文报告模型约在 52K iterations 收敛，并使用单张 80GB NVIDIA H100 训练，但没有给出三个训练集经过采样后的统一训练序列或帧总量，因此不自行推断样本总数。

Camera-coordinate 评估使用 3DPW 和 EMDB-1，指标包括 MPJPE、PA-MPJPE、PVE、Accel。World-coordinate 评估使用 EMDB-2，将序列按 100 帧切段，报告 W-MPJPE、WA-MPJPE、RTE 和 ERVE。相机轨迹部分还使用 ATE 比较不同人体 masking 策略；效率报告 FPS 与 Avg. Delay。

## 主要结果

在 3DPW 上，OnlineHMR 的 PA-MPJPE / MPJPE / PVE / Accel 分别为 43.7 / 69.9 / 83.7 / 6.4 mm；EMDB-1 为 46.0 / 74.0 / 86.1 / 9.0 mm。相较在线 Human3R，局部人体重建和时序平滑总体更强。

在 EMDB-2 的世界坐标评估中，OnlineHMR 达到 PA-MPJPE 40.1 mm、WA-MPJPE100 93.5 mm、W-MPJPE100 310.4 mm、RTE 2.2%、ERVE 12.4 mm/frame。它的 WA-MPJPE 接近部分强离线方法，但 W-MPJPE 明显落后于 TRAM 的 222.4 mm，论文将这一差距主要归因于增量式 metric-scale 转换仍不够准确。

软人体掩码对相机恢复贡献明显：DROID-SLAM 的 ATE 从 vanilla 2.52 降到 hard mask 1.55、soft mask 1.07；MASt3R-SLAM 则从 1.22 降到 0.96 和 0.83。整体系统在 RTX 6000 Ada 上报告 3.3 FPS、平均延迟 0.30 s，Human3R 为 4.8 FPS / 0.21 s，但其 WA-MPJPE100 为 112.2 mm。

## 优点

- 把“人体局部姿态”和“移动相机世界轨迹”显式拆成专家分支，问题定义与真实 moving-camera human reconstruction 很一致。
- 不是仅在网络层声明 causal，而是从 HMR、缓存、SLAM 到尺度恢复都按 streaming protocol 设计。
- soft human mask 直接针对“动态人体占据大面积画面导致 SLAM 错配”的实际问题，消融结果清楚。
- 同时报告 local pose、world motion、camera trajectory 与 latency，使方法的收益和代价较容易分辨。

## 局限

论文明确指出，OnlineHMR 依赖 incremental SLAM 和基于历史的 EMA correction，因此假设视点连续；遇到突然切镜头、严重视角不连续或 multi-camera input 时会困难。另一个可从实验直接观察到的边界是 metric scale：WA-MPJPE 较好但 W-MPJPE 明显退化，说明世界坐标尺度仍是主要误差源之一。

**推断：**对于高速滑雪自拍视频，背景纹理不足、雪面重复纹理、快速旋转和大幅加速度都可能进一步放大 SLAM 与尺度恢复误差，因此不能直接假设其 EMDB 表现可迁移到雪场。

## 个人评价

这篇论文最值得关注的不是单独的 HMR 精度，而是它把 moving-camera human reconstruction 拆成了可诊断的 camera-space human、camera trajectory、metric scale 和 world transform 四个环节。对后续研究而言，这种拆分比简单把一个相机轨迹估计器接到 3D pose 后面更容易做强消融，也更容易回答“误差究竟来自人体、相机还是尺度”。

## 与我的研究关联

与当前移动 360° 自拍、世界坐标人体运动恢复和 camera-human joint optimization 的方向高度相关。可以直接把 OnlineHMR 作为 ViPE/SLAM 路线之外的强 baseline，并设计 `GT camera → estimated camera → soft-mask camera → jointly refined camera` 的分层实验，同时比较 ATE/RPE、W-MPJPE、PA-MPJPE 和 root trajectory error。

最值得迁移的是 soft human mask 和 metric-scale decomposition：360° 自拍里人物通常占据稳定但显著的前景区域，camera estimator 应尽量从静态背景提取运动；同时将相机旋转/相对平移与 metric scale 分开处理，可以更清楚地判断联合优化到底修正了哪个部分。**推断：**如果将多透视人体 3D keypoints 反过来作为 metric/kinematic constraint 修正相机轨迹，可能形成比 OnlineHMR 单向“camera → human”更强的双向 camera-human refinement。

## 后续阅读

- 优先精读 Sec. 3.2 的 camera-coordinate online HMR、Sec. 3.3 的 Human-Centric Incremental SLAM，以及 Tables 2、3、5 的世界坐标、效率和 masking 消融。
- 复现时先单独验证 `vanilla / hard mask / soft mask` 的 camera ATE，再接入人体世界坐标恢复，避免把 SLAM 改进与 HMR 改进混在一起解释。
- 与 TRAM、WHAM、Human3R 以及 moving-camera camera-estimation 方法做统一协议比较。
